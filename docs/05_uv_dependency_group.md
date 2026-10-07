# uv — Dependency Groups and `uv sync --no-dev`

## 1. Why Dependency Groups?

A project usually has different types of dependencies.

For example:

### Runtime dependencies

Packages required for the application to run:

```toml
[project]
dependencies = [
    "requests==2.32.0",
]
```

### Development dependencies

Packages required during development/testing:

```toml
[dependency-groups]
dev = [
    "pytest>=9.1.1",
]
```

The project therefore has:

```text
uv-learning
│
├── Runtime dependencies
│   └── requests
│
└── Development dependencies
    └── pytest
```

---

# 2. Adding a Development Dependency

Instead of:

```bash
uv add pytest
```

we used:

```bash
uv add --dev pytest
```

This created:

```toml
[dependency-groups]
dev = [
    "pytest>=9.1.1",
]
```

So `pytest` is explicitly classified as a development dependency.

---

# 3. Runtime vs Development Dependencies

Our dependency graph is:

```text
uv-learning
│
├── requests 2.32.0
│   ├── certifi
│   ├── charset-normalizer
│   ├── idna
│   └── urllib3
│
└── pytest 9.1.1 (group: dev)
    ├── iniconfig
    ├── packaging
    ├── pluggy
    └── pygments
```

Runtime dependency:

```text
requests
```

Development dependency:

```text
pytest
```

`pytest` is needed for testing the project but is not required by the application itself at runtime.

---

# 4. `uv tree` Shows Dependency Groups

Running:

```bash
uv tree
```

shows:

```text
uv-learning v0.1.0
├── requests v2.32.0
│   ├── certifi v2026.7.22
│   ├── charset-normalizer v3.5.2
│   ├── idna v3.20
│   └── urllib3 v2.8.0
└── pytest v9.1.1 (group: dev)
    ├── iniconfig v2.3.1
    ├── packaging v26.3
    ├── pluggy v1.6.0
    └── pygments v2.21.0
```

The important part is:

```text
pytest v9.1.1 (group: dev)
```

This tells us that `pytest` belongs to the `dev` dependency group.

---

# 5. `uv sync`

Running:

```bash
uv sync
```

synchronizes the environment with the project's dependency configuration.

In our project, this resulted in both:

```text
Runtime dependencies
+
Development dependencies
```

being installed.

The environment contained:

```text
certifi
charset-normalizer
idna
iniconfig
packaging
pluggy
pygments
pytest
requests
urllib3
uv-learning
```

---

# 6. `uv sync --no-dev`

Running:

```bash
uv sync --no-dev
```

synchronizes the environment without the development dependency group.

Before:

```text
pytest
iniconfig
packaging
pluggy
pygments
```

were installed.

After:

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
certifi
charset-normalizer
idna
requests
urllib3
uv-learning
```

---

# 7. Restoring Development Dependencies

Running:

```bash
uv sync
```

again restored the development dependencies.

`uv` installed:

```text
+ iniconfig
+ packaging
+ pluggy
+ pygments
+ pytest
```

The environment again contained both runtime and development dependencies.

---

# 8. Important Observation

When we ran:

```bash
uv sync --no-dev
```

`uv` did not remove the development dependencies from the project configuration.

This remained in `pyproject.toml`:

```toml
[dependency-groups]
dev = [
    "pytest>=9.1.1",
]
```

Only the **environment installation state** changed.

Therefore:

```text
pyproject.toml
    │
    └── dev group still exists
              │
              ▼
       uv sync --no-dev
              │
              ▼
       dev packages not installed
```

Running:

```bash
uv sync
```

can install them again.

---

# 9. Important Observation About `uv.lock`

Even when we ran:

```bash
uv sync --no-dev
```

the resolver still reported:

```text
Resolved 12 packages
```

The development dependencies were still part of the project's resolved dependency state.

`--no-dev` controls what gets installed into the environment; it does not mean:

> Remove the development dependencies from the project or lock file.

This gives us an important distinction:

```text
uv.lock
    ↓
Project's resolved dependency state

uv sync --no-dev
    ↓
Controls which dependency groups are installed
```

---

# 10. Development vs Production Mental Model

A typical project can look like:

```text
                 Project
                    │
          ┌─────────┴─────────┐
          │                   │
       Runtime              Dev
          │                   │
      requests              pytest
          │                   │
          ▼                   ▼
     Application           Testing
```

Development environment:

```bash
uv sync
```

Conceptually:

```text
Runtime dependencies
+
Development dependencies
```

Production-oriented environment:

```bash
uv sync --no-dev
```

Conceptually:

```text
Runtime dependencies only
```

---

# 11. Why This Is Useful

Development tools are often unnecessary in production.

For example:

```text
pytest
linters
formatters
development utilities
```

may only be needed while developing or testing the application.

The production environment can therefore avoid installing them.

This results in a cleaner runtime environment with fewer unnecessary packages.

---

# 12. Commands Learned

### Add runtime dependency

```bash
uv add requests
```

### Add development dependency

```bash
uv add --dev pytest
```

### Show dependency tree

```bash
uv tree
```

### Install/synchronize normal project environment

```bash
uv sync
```

### Synchronize without development dependencies

```bash
uv sync --no-dev
```

### List installed packages

```bash
uv pip list
```

---

# 13. Complete Mental Model

```text
                     pyproject.toml
                           │
             ┌─────────────┴─────────────┐
             │                           │
       [project]                 [dependency-groups]
             │                           │
             ▼                           ▼
        dependencies                    dev
             │                           │
             ▼                           ▼
          requests                    pytest
             │                           │
             └─────────────┬─────────────┘
                           ▼
                       uv resolver
                           │
                           ▼
                        uv.lock
                           │
                           ▼
                       uv sync
                           │
                           ▼
                         .venv
```

With:

```bash
uv sync
```

the environment includes the normal development group.

With:

```bash
uv sync --no-dev
```

the environment excludes the development group.

---

# 14. Key Takeaways

1. `uv add --dev` adds a dependency to the `dev` dependency group.

2. Runtime dependencies are declared under:

```toml
[project]
dependencies = [...]
```

3. Development dependencies are declared under:

```toml
[dependency-groups]
dev = [...]
```

4. `uv tree` shows dependency groups.

5. `uv sync` installs the project's normal dependency groups.

6. `uv sync --no-dev` excludes the development dependency group from the environment.

7. `uv sync --no-dev` does not remove the `dev` group from `pyproject.toml`.

8. The lock file can still contain the resolved development dependencies even when they are not installed into the current environment.

9. Dependency groups allow us to separate:

```text
Runtime
   ↓
Application requirements

Development
   ↓
Testing/development requirements
```

10. The core concept is:

```text
Project configuration
        ↓
Dependency groups
        ↓
Dependency resolution
        ↓
uv.lock
        ↓
Choose what to install
        ↓
.venv
```