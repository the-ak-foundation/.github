## Description

<!-- What does this PR do? One paragraph is enough. -->

## Type of change

- [ ] Bug fix
- [ ] New feature / HAL driver
- [ ] Documentation update
- [ ] Refactor (no behaviour change)
- [ ] Build / toolchain change

## How to test

<!-- Describe how a reviewer can verify this works. Hardware steps, expected output, etc. -->

```
# Example:
make
ak_flash /dev/ttyUSB0 application.bin 0x08003000
# Expected: LED blinks at 1Hz, serial output shows "TIMER OK"
```

## Checklist

- [ ] Code follows the project style (4-space indent, snake_case)
- [ ] Tested on AK Embedded Base Kit hardware (or noted if untestable)
- [ ] Relevant documentation updated
- [ ] Commit messages follow the format: `type(scope): description`

<!-- Closes #ISSUE_NUMBER -->
