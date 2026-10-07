# uv — Python Version Changes and Multiple Environments

## Question

### What happens if I change the Python version in an existing `uv` project?

And:

### What if I want more than one Python environment for the same project?

---

# 1. Changing the Python Version

Suppose the project currently uses:

```text
.python-version
       ↓
      3.12
```

and the environment was created using:

```text
.venv
   ↓
Python 3.12
```

The project looks like:

```text
uv_learning/
│
├── .python-version
├── pyproject.toml
├── uv.lock
└── .venv/
```

---

## Change the Project's Python Version

Use:

```bash
uv python pin 3.13
```

This changes:

```text
.python-version
```

to:

```text
3.13
```

The project now declares:

```text
Project Python
       ↓
     3.13
```

---

# 2. What About the Existing `.venv`?

Before the change:

```text
.python-version
       ↓
      3.12

.venv
       ↓
Python 3.12
```

After:

```bash
uv python pin 3.13
```

we have:

```text
.python-version
       ↓
      3.13

.venv
       ↓
still previously created with 3.12
```

Changing `.python-version` does not mean that an already-created virtual environment magically changes its Python interpreter.

---

# 3. Synchronize the Environment

Run:

```bash
uv sync
```

`uv` will synchronize the project environment with the project's current configuration.

Conceptually:

```text
.python-version
       ↓
      3.13
       │
       ▼
     uv sync
       │
       ▼
     .venv
       ↓
Python 3.13
```

Therefore, the important workflow is:

```bash
uv python pin 3.13
uv sync
```

---

# 4. `uv python pin` vs `uv venv`

These commands have different purposes.

## `uv python pin`

```bash
uv python pin 3.13
```

This sets the Python version associated with the project.

It modifies:

```text
.python-version
```

Think:

```text
uv python pin
       ↓
"Which Python should this project use?"
```

---

## `uv venv`

```bash
uv venv .venv --python 3.13
```

This explicitly creates a virtual environment using a specified Python version.

Think:

```text
uv venv
       ↓
"Create an isolated Python environment."
```

---

# 5. What if I want two environments?

Suppose we want to test the same project with:

```text
Python 3.12
Python 3.13
```

We could create:

```bash
uv venv .venv-py312 --python 3.12
```

and:

```bash
uv venv .venv-py313 --python 3.13
```

The project could then contain:

```text
uv_learning/
│
├── .venv-py312/
│   └── Python 3.12
│
├── .venv-py313/
│   └── Python 3.13
│
├── .python-version
├── pyproject.toml
└── uv.lock
```

Now there are two independent virtual environments.

---

# 6. Why Don't We Normally Maintain Multiple `.venv`s?

If the goal is:

> Test whether the project works on multiple Python versions.

we generally don't manually maintain:

```text
.venv-py312
.venv-py313
.venv-py314
```

Instead, we can test the project against multiple Python versions using automation/CI.

Conceptually:

```text
             Project
                │
       ┌────────┼────────┐
       ▼        ▼        ▼
   Python 3.12 Python 3.13 Python 3.14
       │        │        │
       ▼        ▼        ▼
     Tests    Tests    Tests
```

This is especially useful in CI/CD.

---

# 7. Important Mental Model

There are three different concepts:

```text
1. Python version selection
          │
          ▼
    .python-version


2. Virtual environment
          │
          ▼
        .venv


3. Project dependencies
          │
          ▼
    pyproject.toml
          │
          ▼
       uv.lock
```

---

# 8. Example

Initial project:

```text
.python-version → 3.12
.venv            → Python 3.12
```

Run:

```bash
uv python pin 3.13
```

Now:

```text
.python-version → 3.13
.venv            → previously created with 3.12
```

Then:

```bash
uv sync
```

The environment is synchronized with the project's current configuration.

Conceptually:

```text
.python-version → 3.13
       │
       ▼
    uv sync
       │
       ▼
.venv → Python 3.13
```

---

# 9. Key Takeaways

### `uv python pin`

Sets the Python version for the project.

```bash
uv python pin 3.13
```

Updates:

```text
.python-version
```

---

### `uv sync`

Synchronizes the project's environment with its configuration.

```bash
uv sync
```

---

### `uv venv`

Creates a virtual environment.

```bash
uv venv .venv --python 3.13
```

---

### Multiple environments

Possible:

```bash
uv venv .venv-py312 --python 3.12
uv venv .venv-py313 --python 3.13
```

Result:

```text
.venv-py312 → Python 3.12
.venv-py313 → Python 3.13
```

But multiple environments are usually useful for **testing different Python versions**, rather than something we maintain manually for everyday development.

---

# 10. Summary

```text
                 uv project
                     │
          ┌──────────┼──────────┐
          │          │          │
          ▼          ▼          ▼
 .python-version  pyproject.toml  uv.lock
          │          │          │
          │          │          │
          ▼          ▼          ▼
       Python     Requirements  Resolved
       version                  dependencies
          │
          ▼
        .venv
          │
          ▼
    Installed environment
```

The key distinction is:

```text
uv python pin
    → selects the project's Python version

uv venv
    → creates a virtual environment

uv sync
    → synchronizes the environment with the project
```