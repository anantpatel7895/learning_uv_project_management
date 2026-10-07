# Lesson 1 — uv Fundamentals

## 1. What is `uv`?

`uv` is a fast Python package and project manager.

It can manage:

- Python versions
- Virtual environments
- Python dependencies
- Dependency resolution
- Lock files
- Running Python commands
- Project configuration

Instead of using multiple tools independently:

```text
Python
    +
venv
    +
pip
    +
pip-tools
```

`uv` provides a unified workflow for Python projects.

---

# 2. Why do we need project management?

Consider two Python projects:

```text
Project A
└── FastAPI 0.115

Project B
└── FastAPI 0.120
```

If both projects use the same system Python environment, their dependencies can conflict.

A virtual environment isolates dependencies:

```text
System Python
│
├── Project A
│   └── .venv
│       └── FastAPI 0.115
│
└── Project B
    └── .venv
        └── FastAPI 0.120
```

Each project gets its own environment.

---

# 3. Traditional Python workflow

Without `uv`, a common workflow is:

```bash
python -m venv .venv
```

Activate the environment:

```bash
source .venv/bin/activate
```

Install dependencies:

```bash
pip install fastapi
```

For multiple dependencies, we may use:

```text
requirements.txt
```

and then:

```bash
pip install -r requirements.txt
```

This works, but dependency management can become complicated as projects grow.

---

# 4. Direct and transitive dependencies

Suppose our application directly depends on:

```text
Application
    │
    ▼
  FastAPI
```

FastAPI itself depends on other packages:

```text
Application
    │
    ▼
  FastAPI
    │
    ├── Starlette
    │
    └── Pydantic
            │
            └── Other dependencies
```

### Direct dependency

A package that our project explicitly requires.

Example:

```text
fastapi
```

### Transitive dependency

A dependency required by one of our dependencies.

Example:

```text
FastAPI
   ↓
Starlette
```

We install FastAPI, and the package manager resolves Starlette automatically.

---

# 5. The dependency-resolution problem

Suppose:

```text
Our application
      │
      ├── FastAPI
      │
      └── Some other package
```

Both packages may depend on different versions of another package.

For example:

```text
FastAPI
   └── Package X >= 1.0

Library B
   └── Package X < 2.0
```

The package manager must find a version of Package X satisfying all constraints.

This is called:

```text
Dependency Resolution
```

`uv` performs this resolution for us.

---

# 6. What problem does `uv` solve?

At a high level:

```text
                    uv
                     │
       ┌─────────────┼─────────────┐
       │             │             │
       ▼             ▼             ▼
 Python versions  Dependencies   Environments
       │             │             │
       └─────────────┼─────────────┘
                     │
                     ▼
              Reproducible
                 project
```

---

# 7. Important files in a uv project

A typical project contains:

```text
my-project/
│
├── .python-version
├── pyproject.toml
├── uv.lock
└── ...
```

Each file has a different responsibility.

---

# 8. `.python-version`

`.python-version` specifies the Python version associated with the project.

Example:

```text
3.12
```

Conceptually:

```text
.python-version
       │
       ▼
Which Python version should this project use?
```

This is particularly useful when different projects require different Python versions.

Example:

```text
Project A → Python 3.11
Project B → Python 3.12
Project C → Python 3.13
```

---

# 9. `pyproject.toml`

`pyproject.toml` is the main project configuration file.

Example:

```toml
[project]
name = "my-project"
version = "0.1.0"
requires-python = ">=3.12"

dependencies = [
    "fastapi>=0.115",
    "numpy>=2.0",
]
```

It describes what our project requires.

For example:

```text
pyproject.toml
       │
       ├── Project name
       ├── Project version
       ├── Python requirement
       └── Direct dependencies
```

Important:

`pyproject.toml` describes the **requirements/constraints of our project**.

It does not necessarily represent every exact package version installed in the environment.

---

# 10. `uv.lock`

`uv.lock` contains the resolved dependency information.

Conceptually:

```text
pyproject.toml
       │
       │ "I need FastAPI >= 0.115"
       ▼
      uv
       │
       │ dependency resolution
       ▼
   uv.lock
       │
       │ exact resolved versions
       ▼
   Environment
```

For example, our project may request:

```text
fastapi >= 0.115
```

The resolver may determine a specific compatible version.

The lock file records the resolved dependency graph so the environment can be reproduced consistently.

### Important

Normally, we should **not manually edit `uv.lock`**.

`uv` manages it.

---

# 11. `.venv`

The `.venv` directory is the project's virtual environment.

Conceptually:

```text
my-project/
│
├── pyproject.toml
├── uv.lock
├── .python-version
└── .venv/
```

The relationship is:

```text
pyproject.toml
       │
       │ project requirements
       ▼
      uv
       │
       │ resolve dependencies
       ▼
    uv.lock
       │
       │ exact dependency set
       ▼
    .venv
       │
       ▼
 installed packages
```

---

# 12. The four important components

Remember these four:

```text
.python-version
    │
    └── Which Python?

pyproject.toml
    │
    └── What does my project require?

uv.lock
    │
    └── What exact dependency versions were resolved?

.venv
    │
    └── Where are the project's packages installed?
```

---

# 13. `uv init`

To create a new project:

```bash
mkdir uv-learning
cd uv-learning

uv init
```

`uv` creates the Python project structure and configuration files.

You can inspect the result with:

```bash
ls -la
```

and:

```bash
cat pyproject.toml
```

---

# 14. Existing Git repository

`uv` project management and Git version control are separate concepts.

Git:

```text
.git/
    │
    └── Version control
```

uv:

```text
pyproject.toml
uv.lock
.python-version
.venv
    │
    └── Python project management
```

If a project is already a Git repository:

```text
my-repo/
├── .git/
├── src/
└── ...
```

we can initialize the Python project inside it:

```bash
cd my-repo
uv init
```

We do **not** need to create another Git repository.

The resulting structure might be:

```text
my-repo/
│
├── .git/
├── .python-version
├── pyproject.toml
├── README.md
├── src/
└── ...
```

Git and uv serve different purposes:

```text
Git
 └── Version control

uv
 └── Python project/dependency management
```

---

# 15. Mental Model

The most important mental model from Lesson 1:

```text
                 Python Project
                       │
                       ▼
              ┌─────────────────┐
              │ pyproject.toml  │
              │                 │
              │ What do I need? │
              └────────┬────────┘
                       │
                       ▼
                      uv
                       │
              Dependency Resolution
                       │
                       ▼
                ┌──────────────┐
                │   uv.lock    │
                │              │
                │ What exactly │
                │ was resolved?│
                └──────┬───────┘
                       │
                       ▼
                    .venv
                       │
                       ▼
              Installed packages
```

And separately:

```text
.git/
  │
  └── Git version control
```

---

# 16. Key Takeaways

1. `uv` is a Python project and package manager.

2. Virtual environments isolate project dependencies.

3. `pyproject.toml` describes the project's requirements and configuration.

4. `uv.lock` records the resolved dependency information.

5. `.python-version` identifies the Python version associated with the project.

6. `.venv` is the project's isolated Python environment.

7. Direct dependencies are packages explicitly required by our project.

8. Transitive dependencies are dependencies required by our dependencies.

9. Dependency resolution determines compatible versions across the dependency graph.

10. Git and `uv` solve different problems.

```text
Git → Version control

uv  → Python project/dependency management
```

---

# 17. Commands Learned

Create a project:

```bash
uv init
```

Create a project directory and initialize it:

```bash
mkdir uv-learning
cd uv-learning
uv init
```

Inspect project configuration:

```bash
cat pyproject.toml
```

Inspect project files:

```bash
ls -la
```

---

# 18. Next Lesson

## Lesson 2 — `uv init` Deep Dive

We will create a real project and inspect exactly what `uv init` does.

We will investigate:

```text
uv init
   │
   ├── pyproject.toml
   ├── .python-version
   ├── README.md
   ├── main.py
   └── .gitignore
```

Then we will answer:

- Why is each file created?
- What is inside `pyproject.toml`?
- What does `[project]` mean?
- What is `requires-python`?
- What does `dependencies = []` mean?
- Does `uv init` create `.venv`?
- Does `uv init` create `uv.lock`?
- What happens if the directory is already a Git repository?