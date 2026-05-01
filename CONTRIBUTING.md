# Contributing to AK Foundation

Thank you for your interest in contributing! This document covers everything you need to get started.

## Before you start

- Check the [open issues](https://github.com/the-ak-foundation/ak-base-kit-stm32l151/issues) — your idea might already be tracked.
- For **large changes** (new features, API changes, new hardware ports), open an issue first and discuss before writing code.
- For **small fixes** (typos, obvious bugs, documentation), go ahead and open a pull request directly.

## Development environment

**Required tools:**

```bash
# ARM cross-compiler
sudo apt-get install gcc-arm-none-eabi make

# Flash tool
git clone https://github.com/the-ak-foundation/ak-flash
```

**Build the firmware:**

```bash
git clone https://github.com/the-ak-foundation/ak-base-kit-stm32l151
cd ak-base-kit-stm32l151/application
make
```

**Flash to hardware:**

```bash
ak_flash /dev/ttyUSB0 ak-base-kit-stm32l151-application.bin 0x08003000
```

## Contribution workflow

1. **Fork** this repository and create a branch:
   ```bash
   git checkout -b fix/uart-baud-calculation
   # or
   git checkout -b feat/add-spi-dma
   ```

2. **Make your changes.** Keep commits small and focused. Write a clear commit message:
   ```
   fix(hal/uart): correct baud rate calculation for 115200

   The divisor formula was off by one for HSI clock at 16MHz.
   Fixes #42.
   ```

3. **Test your changes** on hardware (AK Embedded Base Kit) or document clearly if hardware is unavailable.

4. **Open a pull request** against `main`. Fill in the PR template.

## Code style

- **C/C++ style:** Follow the existing code style in the repo. 4-space indentation, no tabs.
- **Naming:** `snake_case` for functions and variables. `UPPER_CASE` for macros and constants.
- **Comments:** Write comments in English. Document public API functions with a short description.
- **File headers:** Keep the existing copyright header format in modified files.

## What we accept

| Type | Welcome? |
|------|----------|
| Bug fixes | ✅ Always |
| Documentation improvements | ✅ Always |
| New hardware examples | ✅ Welcome |
| New HAL drivers (GPIO, UART, SPI…) | ✅ Welcome |
| New board support packages | ✅ Discuss first |
| Kernel architecture changes | ⚠️ Open issue first |
| Dependency additions | ⚠️ Discuss first |

## Commit message format

We follow a simplified [Conventional Commits](https://www.conventionalcommits.org/) style:

```
<type>(<scope>): <short description>

[optional body]

[optional: Fixes #<issue>]
```

Types: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`  
Scopes: `kernel`, `hal`, `bsp`, `boot`, `examples`, `tools`

## Getting help

- Open a [GitHub Discussion](https://github.com/the-ak-foundation/ak-base-kit-stm32l151/discussions) for questions
- Join the [Facebook community group](https://www.facebook.com/groups/laptrinhvidieukhiennangcao)
- Email the maintainers: [akembeddedsoftware@gmail.com](mailto:akembeddedsoftware@gmail.com)
