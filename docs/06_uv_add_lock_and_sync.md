# uv — `uv add` vs `uv lock` vs `uv sync`

## 1. Overview

These three commands are related to dependency management, but they perform different jobs:

```text
uv add
uv lock
uv sync
```

The simplest mental model is:

```text
uv add
    ↓
Change project dependencies
    ↓
Resolve + update lock
    ↓
Synchronize environment
```

```text
uv lock
    ↓
Resolve/update dependency lock
```

```text
uv sync
    ↓
Make .venv match the project's dependency state
```

---

# 2. `uv add`

## Purpose

`uv add` is used when you want to **add or change a dependency of the project**.

Example:

```bash
uv add requests
```

For a development dependency:

```bash
uv add --dev pytest
```

---

## What `uv add` does

Conceptually:

```text
uv add pytest
      │
      ├── Update pyproject.toml
      │
      ├── Resolve dependency
      │
      ├── Update uv.lock
      │
      └── Synchronize .venv
```

Therefore:

| File / Environment | `uv add` |
|---|---|
| `pyproject.toml` | ✅ Updates |
| `uv.lock` | ✅ Updates |
| `.venv` | ✅ Synchronizes / installs |

### Important

`uv add` is **not only** a `pyproject.toml` operation.

It also synchronizes the environment.

Our experiment:

```bash
uv add --dev pytest
```

resulted in:

```text
Installed 6 packages
+ pytest
+ iniconfig
+ packaging
+ pluggy
+ pygments
```

Therefore:

> `uv add` adds the dependency to the project and normally synchronizes the environment.

---

# 3. `uv lock`

## Purpose

`uv lock` is used to **resolve/update the project's lock file**.

Example:

```bash
uv lock
```

Conceptually:

```text
pyproject.toml
      │
      ▼
   Resolver
      │
      ▼
   uv.lock
```

It is primarily about dependency resolution and the lock file.

### What it does

| File / Environment | `uv lock` |
|---|---|
| `pyproject.toml` | ❌ Does not normally add/change dependencies |
| `uv.lock` | ✅ Updates/resolves |
| `.venv` | ❌ Not primarily an installation operation |

Example output from our experiment:

```text
Resolved 6 packages in 18ms
```

---

# 4. `uv sync`

## Purpose

`uv sync` is used to **synchronize the virtual environment with the project's dependency state**.

Example:

```bash
uv sync
```

Conceptually:

```text
pyproject.toml
      +
   uv.lock
      │
      ▼
   uv sync
      │
      ▼
    .venv
```

---

# 5. `uv sync` Does More Than Install

A common misconception is:

```text
uv sync = install missing packages
```

A better definition is:

> `uv sync` makes the environment match the project's desired dependency state.

Therefore it can:

```text
install missing packages
update packages when required
remove unnecessary packages
```

---

# 6. Our `--no-dev` Experiment

Initially:

```text
.venv
├── requests
├── pytest
├── iniconfig
├── packaging
├── pluggy
└── pygments
```

We then ran:

```bash
uv sync --no-dev
```

`uv` removed:

```text
- iniconfig
- packaging
- pluggy
- pygments
- pytest
```

The runtime dependencies remained:

```text
requests
certifi
charset-normalizer
idna
urllib3
```

This demonstrates that:

> `uv sync` can remove packages that are no longer part of the selected dependency set.

---

# 7. `uv sync --no-dev`

Our project contains:

```toml
[dependency-groups]
dev = [
    "pytest>=9.1.1",
]
```

Normally:

```bash
uv sync
```

includes the development group.

But:

```bash
uv sync --no-dev
```

excludes it from the environment.

Conceptually:

```text
uv sync
    ↓
Runtime + dev
```

while:

```text
uv sync --no-dev
    ↓
Runtime only
```

---

# 8. Important Difference: Project State vs Environment State

Consider:

```toml
[dependency-groups]
dev = [
    "pytest>=9.1.1",
]
```

Running:

```bash
uv sync --no-dev
```

does **not** delete this configuration.

The `dev` dependency still exists in:

```text
pyproject.toml
```

It simply isn't installed in the current environment.

Running:

```bash
uv sync
```

can install it again.

Therefore:

```text
pyproject.toml
    │
    └── describes what the project has
             │
             ▼
        uv sync --no-dev
             │
             └── choose not to install dev group
```

---

# 9. Command Comparison

| Command | Main Purpose | `pyproject.toml` | `uv.lock` | `.venv` |
|---|---|---:|---:|---:|
| `uv add requests` | Add/change dependency | ✅ | ✅ | ✅ |
| `uv add --dev pytest` | Add dev dependency | ✅ | ✅ | ✅ |
| `uv lock` | Resolve/update lock | ❌* | ✅ | ❌ |
| `uv sync` | Synchronize environment | ❌ | Uses lock/state | ✅ |
| `uv sync --no-dev` | Sync without dev group | ❌ | Uses lock/state | ✅ |

\* `uv lock` can make project-related lock/config changes in some workflows, but its primary purpose is lock resolution rather than declaring a new dependency.

---

# 10. Practical Mental Model

Think of the commands as different levels of the workflow.

### `uv add`

> **I want to change what my project depends on.**

```text
Developer
    │
    ▼
uv add
    │
    ├── pyproject.toml
    ├── uv.lock
    └── .venv
```

---

### `uv lock`

> **I want to resolve/update the dependency lock.**

```text
pyproject.toml
      │
      ▼
uv lock
      │
      ▼
uv.lock
```

---

### `uv sync`

> **I want my environment to match the project.**

```text
pyproject.toml + uv.lock
            │
            ▼
         uv sync
            │
            ▼
          .venv
```

---

# 11. Real-World Examples

## Adding a new runtime dependency

```bash
uv add fastapi
```

This changes the project and synchronizes the environment.

---

## Adding a testing dependency

```bash
uv add --dev pytest
```

This adds pytest to the `dev` dependency group and synchronizes the environment.

---

## Recreate an environment

If `.venv` is deleted:

```bash
rm -rf .venv
uv sync
```

`uv` recreates the environment.

---

## Remove development packages from the environment

```bash
uv sync --no-dev
```

The project still knows about the `dev` group, but those packages aren't installed in the current environment.

---

## Refresh the lock

```bash
uv lock
```

Use this when the goal is dependency resolution/lock-file maintenance rather than explicitly adding a new dependency.

---

# 12. Most Important Distinction

Do not memorize these as three unrelated commands.

Think:

```text
                 PROJECT
                    │
                    ▼
             pyproject.toml
                    │
                    │
          "What does my project need?"
                    │
                    ▼
                uv add
                    │
                    ▼
              Dependency
              resolution
                    │
                    ▼
                 uv.lock
                    │
                    │
                    ▼
                 uv sync
                    │
                    ▼
                  .venv
```

And remember:

```text
uv add
    = change dependencies

uv lock
    = resolve/update lock

uv sync
    = synchronize environment
```

---

# 13. Key Takeaways

1. **`uv add` changes the project's dependencies.**

2. `uv add` normally updates:
   - `pyproject.toml`
   - `uv.lock`
   - `.venv`

3. **`uv lock` focuses on dependency resolution and the lock file.**

4. **`uv sync` synchronizes the environment with the project's dependency state.**

5. `uv sync` can install missing packages.

6. `uv sync` can remove packages that are no longer required by the selected dependency set.

7. `uv sync --no-dev` excludes development dependencies from the environment.

8. `uv sync --no-dev` does not remove the `dev` dependency declarations from `pyproject.toml`.

9. `.venv` is an environment that can be recreated.

10. `uv.lock` represents the resolved dependency state.

The simplest rule to remember:

```text
uv add   → Change dependencies
uv lock  → Resolve the lock
uv sync  → Make .venv match the project
```