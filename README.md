<div align="center">

![Repo Traffic](https://komarev.com/ghpvc/?username=the-ak-foundation-dotgithub&label=Repo+Traffic&color=blue&style=flat-square)

</div>

# AK Foundation — Organization defaults

This repository contains files that apply to **every repository** in the
[AK Foundation](https://github.com/the-ak-foundation) organization.

---

## Community health files

| File | Purpose |
|------|---------|
| [CONTRIBUTING.md](CONTRIBUTING.md) | How to contribute to any AK Foundation repo |
| [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) | Community standards |
| [PULL_REQUEST_TEMPLATE.md](PULL_REQUEST_TEMPLATE.md) | Default PR checklist |
| [ISSUE_TEMPLATE/](ISSUE_TEMPLATE/) | Bug report and feature request forms |

GitHub automatically applies these to any repo in the org that does not have
its own version.

---

## Coding style (org-wide)

| File | Purpose |
|------|---------|
| [CODING_STYLE.md](CODING_STYLE.md) | Full style guide — naming, braces, comments, commits |
| [.clang-format](.clang-format) | Machine-readable format rules (copy to each repo root) |
| [tools/pre-commit](tools/pre-commit) | Git hook — rejects commits with style violations |

### Applying the style to a new repo

```bash
# 1. Copy .clang-format to the repo root (required by the clang-format tool)
curl -o .clang-format \
  https://raw.githubusercontent.com/the-ak-foundation/.github/main/.clang-format

# 2. Install the pre-commit hook (each developer, once per clone)
curl -o .git/hooks/pre-commit \
  https://raw.githubusercontent.com/the-ak-foundation/.github/main/tools/pre-commit
chmod +x .git/hooks/pre-commit

# 3. Format all existing files
find . -name '*.c' -o -name '*.h' | xargs clang-format -i
```

### CI style check

Add this workflow file to `.github/workflows/style.yml` in each repo:

```yaml
name: Style check
on:
  pull_request:
    branches: [main]
    paths: ['**.c', '**.cpp', '**.h', '**.hpp']
jobs:
  clang-format:
    runs-on: ubuntu-22.04
    steps:
      - uses: actions/checkout@v4
      - run: sudo apt-get install -y clang-format
      - name: Check
        run: |
          find . -not -path './build/*' \( -name '*.c' -o -name '*.h' \) \
            | xargs clang-format --dry-run --Werror
```

---

## Organization profile

[profile/README.md](profile/README.md) is displayed on the
[AK Foundation organization page](https://github.com/the-ak-foundation).
