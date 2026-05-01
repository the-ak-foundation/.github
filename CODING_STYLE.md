# AKOS Coding Style Guide

This document defines the coding style for all C and C++ source files in the
AKOS project. Following a consistent style makes the codebase easier to read,
review, and maintain — especially for contributors working across different
modules.

Style is enforced automatically by `.clang-format`. When in doubt, run the
formatter rather than debating whitespace.

---

## 1. Language standard

| Layer | Standard |
|-------|----------|
| Kernel core, HAL, BSP | C11 (`-std=c11`) |
| C++ wrappers, samples | C++17 (`-std=c++17`) |
| Assembly port files | ARM UAL syntax |

Avoid compiler extensions. Code must compile cleanly with
`-Wall -Wextra -Werror` on `gcc-arm-none-eabi`.

---

## 2. Indentation and line length

- **4 spaces** per indent level. No tabs.
- Maximum line length: **80 characters**.
- Long expressions break after operators, aligned to the opening parenthesis:

```c
/* Good */
ret = akos_task_post(TASK_SENSOR_ID,
                     SIG_DATA_READY);

/* Bad */
ret = akos_task_post(TASK_SENSOR_ID, SIG_DATA_READY); /* 85 chars — too long */
```

---

## 3. Naming conventions

### 3.1 Functions

Pattern: `module_verb()` or `module_noun_verb()`

All lowercase, words separated by underscores. The module prefix groups
functions by the subsystem they belong to.

```c
/* Kernel public API — prefix: akos_ */
void     akos_init(void);
void     akos_start(void);
int      akos_task_post(ak_task_id_t id, ak_signal_t sig);
void     akos_timer_set(ak_task_id_t id, ak_signal_t sig,
                        uint32_t ms, ak_timer_type_t type);
void     akos_timer_cancel(ak_task_id_t id, ak_signal_t sig);

/* HAL layer — prefix: hal_<peripheral>_ */
void     hal_gpio_set_output(uint32_t pin);
void     hal_gpio_toggle(uint32_t pin);
int      hal_uart_write(uint8_t *buf, size_t len);
uint8_t  hal_uart_read_byte(void);

/* BSP layer — prefix: bsp_ */
void     bsp_board_init(void);
uint32_t bsp_get_tick_ms(void);

/* Static (file-scope) helpers — no module prefix */
static void reset_task_queue(void);
static int  find_free_timer_slot(void);
```

### 3.2 Types and structs

Pattern: `module_noun_t` — always lowercase, always with `_t` suffix.

```c
/* Typedefs */
typedef uint8_t   ak_task_id_t;
typedef uint16_t  ak_signal_t;

/* Structs — define tag and typedef separately */
struct ak_task {
    ak_task_id_t      id;
    ak_task_handler_t handler;
    uint8_t           priority;
    const char       *name;
};
typedef struct ak_task ak_task_t;

/* Enums */
typedef enum {
    TIMER_ONE_SHOT = 0,
    TIMER_PERIODIC,
} ak_timer_type_t;
```

### 3.3 Macros and constants

All uppercase, words separated by underscores. Module prefix in uppercase.

```c
/* Configuration constants */
#define AKOS_CONFIG_MAX_TASKS     16
#define AKOS_CONFIG_MAX_TIMERS    32
#define AKOS_CONFIG_TICK_RATE_HZ  1000

/* Status codes */
#define AKOS_OK        0
#define AKOS_ERR      -1
#define AKOS_ERR_FULL -2

/* Bit masks */
#define AKOS_TASK_FLAG_ACTIVE    (1u << 0)
#define AKOS_TASK_FLAG_BLOCKED   (1u << 1)
```

### 3.4 Variables

Local variables: lowercase `snake_case`, descriptive names. Single-letter
variables are acceptable only as loop counters (`i`, `j`, `k`).

```c
/* Good */
uint32_t elapsed_ms;
ak_task_t *current_task;
int retry_count;

/* Bad */
uint32_t t;      /* not descriptive */
int rC;          /* mixed case */
uint8_t ucData;  /* Hungarian notation — not used in AKOS */
```

Global variables: prefix with `g_`. Use sparingly — most state belongs inside
a module's `.c` file as `static`.

```c
static uint32_t g_tick_counter;   /* file-global */
```

### 3.5 File names

Lowercase, words separated by underscores. The file name mirrors the primary
module prefix of the functions it contains.

```
kernel/core/ak_task.c       → contains akos_task_*() functions
kernel/core/ak_timer.c      → contains akos_timer_*() functions
hal/stm32l151/hal_gpio.c    → contains hal_gpio_*() functions
hal/stm32l151/hal_uart.c    → contains hal_uart_*() functions
bsp/ak-base-kit-v3/bsp_board.c
include/akos/akos.h
include/akos/task.h
```

---

## 4. Braces and control flow

Opening brace on the **same line** as the statement (K&R style).
Exception: function definitions place the opening brace on a new line.

```c
/* Control flow — brace on same line */
if (task_id >= AKOS_CONFIG_MAX_TASKS) {
    return AKOS_ERR;
}

for (int i = 0; i < count; i++) {
    process(i);
}

switch (msg->sig) {
case SIG_INIT:
    init_module();
    break;
case SIG_UPDATE:
    update_module();
    break;
default:
    break;
}

/* Function definition — brace on new line */
int akos_task_post(ak_task_id_t dest_id, ak_signal_t sig)
{
    if (dest_id >= AKOS_CONFIG_MAX_TASKS) {
        return AKOS_ERR;
    }
    /* ... */
    return AKOS_OK;
}
```

Always use braces, even for single-statement bodies:

```c
/* Good */
if (err) {
    return err;
}

/* Bad */
if (err)
    return err;
```

---

## 5. Comments

### 5.1 File header

Every `.c` and `.h` file starts with:

```c
/**
 * @file    ak_task.c
 * @brief   AKOS task scheduler — core implementation.
 *
 * @copyright  AK Foundation — Apache License 2.0
 */
```

### 5.2 Function documentation (public API only)

Document every function in the public header with a Doxygen block:

```c
/**
 * @brief  Post a signal to a task.
 *
 * Safe to call from both task context and ISR context.
 *
 * @param  dest_id  Target task ID.
 * @param  sig      Signal to post.
 * @return AKOS_OK on success, AKOS_ERR_FULL if the queue is full.
 */
int akos_task_post(ak_task_id_t dest_id, ak_signal_t sig);
```

Internal (static) functions use a shorter single-line comment:

```c
/* Find the highest-priority ready task. Returns NULL if none. */
static ak_task_t *find_next_task(void);
```

### 5.3 Inline comments

Use `/* */` for C files. Use `//` only in C++ files.

```c
/* C file — use block comments */
uint32_t mask = (1u << bit_pos);   /* compute the bitmask */

// C++ file — line comments are fine
auto task = find_task(id);  // may return nullptr
```

Avoid obvious comments:

```c
/* Bad — states what the code already says */
i++;   /* increment i */

/* Good — explains WHY */
i++;   /* skip the null terminator byte */
```

---

## 6. Header file structure

Every header must have an include guard using the file path:

```c
#ifndef AKOS_TASK_H
#define AKOS_TASK_H

#ifdef __cplusplus
extern "C" {
#endif

/* --- includes ------------------------------------------------------------ */
#include <stdint.h>
#include <stdbool.h>
#include "akos/signal.h"

/* --- macros -------------------------------------------------------------- */
#define AKOS_TASK_NAME_MAX  16

/* --- types --------------------------------------------------------------- */
typedef uint8_t ak_task_id_t;

struct ak_task { ... };
typedef struct ak_task ak_task_t;

/* --- public API ---------------------------------------------------------- */
void akos_task_register(const ak_task_t *task);
int  akos_task_post(ak_task_id_t dest_id, ak_signal_t sig);

#ifdef __cplusplus
}
#endif

#endif /* AKOS_TASK_H */
```

---

## 7. Includes order

Group includes in this order, with a blank line between each group:

```c
/* 1. Corresponding header (in .c files) */
#include "akos/task.h"

/* 2. Other AKOS headers */
#include "akos/signal.h"
#include "akos/timer.h"

/* 3. HAL / BSP headers */
#include "hal/hal_uart.h"

/* 4. System / CMSIS headers */
#include "stm32l1xx.h"
#include <stdint.h>
#include <string.h>
```

---

## 8. Error handling

Functions that can fail return `int`. Return `AKOS_OK` (0) on success,
negative values on error. Use the defined error codes:

```c
#define AKOS_OK        0
#define AKOS_ERR      -1    /* generic error */
#define AKOS_ERR_FULL -2    /* queue or buffer full */
#define AKOS_ERR_INVAL -3   /* invalid argument */
#define AKOS_ERR_TIMEOUT -4
```

Always check return values at the call site:

```c
int ret = akos_task_post(TASK_LED_ID, SIG_LED_ON);
if (ret != AKOS_OK) {
    /* handle error */
}
```

Do not use `assert()` in production kernel code. Use early-return guards:

```c
int akos_task_post(ak_task_id_t dest_id, ak_signal_t sig)
{
    if (dest_id >= AKOS_CONFIG_MAX_TASKS) {
        return AKOS_ERR_INVAL;
    }
    /* ... */
}
```

---

## 9. Integer types

Always use fixed-width types from `<stdint.h>` for hardware registers,
protocol fields, and anything size-sensitive:

```c
uint8_t   byte_val;
uint16_t  word_val;
uint32_t  reg_val;
int32_t   signed_val;
```

Use `int` / `unsigned int` for loop counters and general-purpose integers
where the exact size does not matter.

Use `size_t` for sizes and counts of objects in memory.

Use `bool` (from `<stdbool.h>`) for boolean values. Never use `int` as bool.

---

## 10. Portability

- No use of `long` or `short` — their sizes are implementation-defined.
- No pointer-to-integer casts unless explicitly porting assembly glue.
- No uninitialized variables.
- No variable-length arrays (VLA) — fixed-size or heap only.
- Interrupt-safe sections must use the port's critical section macros:

```c
AKOS_CRITICAL_ENTER();
/* access shared data */
AKOS_CRITICAL_EXIT();
```

---

## 11. Commit messages

Follow Conventional Commits format:

```
<type>(<scope>): <short description in imperative mood>

[optional body — explain WHY, not what]

[optional: Fixes #<issue>]
```

**Types:** `feat`, `fix`, `docs`, `refactor`, `test`, `chore`, `perf`

**Scopes:** `kernel`, `hal`, `bsp`, `samples`, `docs`, `ci`, `build`

```
feat(hal): add hal_uart_write_dma() for non-blocking TX
fix(kernel): prevent timer list corruption on concurrent cancel
docs(concepts): add scheduler preemption diagram
chore(ci): upgrade arm-none-eabi to 13.x
```

Rules:
- Subject line ≤ 72 characters
- Use imperative mood: "add", "fix", "remove" — not "added", "fixing"
- Do not end subject line with a period

---

## 12. Automated enforcement

### clang-format

Run before every commit:

```bash
clang-format -i <file.c>
# or format all files:
find . -name '*.c' -o -name '*.h' | xargs clang-format -i
```

The `.clang-format` file at the repo root encodes all rules in this document.

### Pre-commit hook (optional)

```bash
cp tools/pre-commit .git/hooks/pre-commit
chmod +x .git/hooks/pre-commit
```

The hook runs `clang-format --dry-run --Werror` and rejects commits with
style violations.

---

## Quick reference card

| Element | Pattern | Example |
|---------|---------|---------|
| Public function | `module_verb()` | `akos_task_post()` |
| HAL function | `hal_periph_verb()` | `hal_uart_write()` |
| BSP function | `bsp_noun_verb()` | `bsp_board_init()` |
| Static helper | `verb_noun()` | `find_free_slot()` |
| Type / struct | `module_noun_t` | `ak_task_t` |
| Macro / constant | `MODULE_UPPER` | `AKOS_CONFIG_MAX_TASKS` |
| Error code | `AKOS_ERR_NAME` | `AKOS_ERR_FULL` |
| Config macro | `AKOS_CONFIG_NAME` | `AKOS_CONFIG_TICK_RATE_HZ` |
| Global variable | `g_noun` (avoid) | `g_tick_counter` |
| File name | `module_noun.c` | `ak_task.c`, `hal_gpio.c` |
| Include guard | `MODULE_NOUN_H` | `AKOS_TASK_H` |
