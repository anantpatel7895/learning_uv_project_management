# uv — Dependencies, Resolution, uv.lock, and uv tree

## 1. `pyproject.toml` — Project Requirements

`pyproject.toml` defines what the project directly depends on.

Example:

```toml
[project]
name = "uv-learning"
version = "0.1.0"
requires-python = ">=3.12"

dependencies = [
    "requests==2.32.0",
]
```

The `dependencies` section describes the **direct dependencies** of our project.

In this example:

```text
uv-learning
    └── requests
```

`requests` is a dependency that our project explicitly requested.

---

# 2. Direct vs Transitive Dependencies

When we add:

```bash
uv add requests
```

we explicitly request `requests`.

Therefore:

```text
requests = Direct dependency
```

But `requests` itself depends on other packages.

For example:

```text
uv-learning
└── requests
    ├── certifi
    ├── charset-normalizer
    ├── idna
    └── urllib3
```

These packages are **transitive dependencies**.

We did not directly add:

```bash
uv add certifi
```

They were required by `requests`.

### Mental Model

```text
Your Project
     │
     └── Direct Dependency
              │
              ├── Transitive Dependency
              ├── Transitive Dependency
              └── Transitive Dependency
```

---

# 3. `uv tree`

`uv tree` displays the resolved dependency graph.

Example:

```bash
uv tree
```

Output:

```text
uv-learning v0.1.0
└── requests v2.32.0
    ├── certifi v2026.7.22
    ├── charset-normalizer v3.5.2
    ├── idna v3.20
    └── urllib3 v2.8.0
```

This answers:

> What dependency graph has `uv` resolved for my project?

It is useful for understanding:

- direct dependencies
- transitive dependencies
- dependency versions
- dependency relationships

---

# 4. `pyproject.toml` vs `uv.lock`

These two files have different responsibilities.

## `pyproject.toml`

Describes the project's declared requirements.

Example:

```toml
dependencies = [
    "requests>=2.34.2",
]
```

This means:

> The project accepts a compatible `requests` version that satisfies this requirement.

---

## `uv.lock`

Contains the resolved dependency information produced by the resolver.

For example, the project might declare:

```text
requests>=2.34.2
```

while the resolver selects:

```text
requests==2.34.2
```

The important distinction is:

```text
pyproject.toml
    ↓
Project requirement / constraint
    ↓
uv resolver
    ↓
uv.lock
    ↓
Resolved dependency graph
```

---

# 5. Why Doesn't `pyproject.toml` Need to Contain Every Package?

Suppose we have:

```toml
dependencies = [
    "requests==2.32.0",
]
```

We don't normally write:

```toml
dependencies = [
    "requests==2.32.0",
    "certifi==...",
    "charset-normalizer==...",
    "idna==...",
    "urllib3==...",
]
```

because those packages are dependencies of `requests`.

Our project directly depends on:

```text
requests
```

and `requests` defines its own dependencies.

Therefore the dependency graph is:

```text
uv-learning
    │
    └── requests
          │
          ├── certifi
          ├── charset-normalizer
          ├── idna
          └── urllib3
```

`uv` resolves the complete graph.

---

# 6. What Happens When We Change a Dependency?

Initially we had:

```toml
dependencies = [
    "requests>=2.34.2",
]
```

Then we changed it to:

```toml
dependencies = [
    "requests==2.32.0",
]
```

using:

```bash
uv add "requests==2.32.0"
```

`uv` then:

1. Updated `pyproject.toml`
2. Re-resolved the dependency graph
3. Updated `uv.lock`
4. Synchronized the environment

Afterward:

```bash
uv tree
```

showed:

```text
uv-learning v0.1.0
└── requests v2.32.0
    ├── certifi
    ├── charset-normalizer
    ├── idna
    └── urllib3
```

This demonstrates that changing a project dependency can cause `uv` to recalculate the dependency graph.

---

# 7. The Complete Dependency Flow

The most important mental model is:

```text
                pyproject.toml
                       │
                       │
              Project requirements
                       │
                       ▼
                 UV RESOLVER
                       │
                       │
              Dependency resolution
                       │
                       ▼
                    uv.lock
                       │
                       │
             Resolved dependency graph
                       │
                       ▼
                    .venv
                       │
                       │
              Installed environment
```

Each component has a different responsibility.

| Component | Purpose |
|---|---|
| `pyproject.toml` | Declares project requirements |
| `uv` resolver | Resolves compatible dependency versions |
| `uv.lock` | Records the resolved dependency information |
| `.venv` | Contains the actual environment |
| `uv tree` | Displays the dependency graph |

---

# 8. Important Mental Model

Do not think:

```text
pyproject.toml = installed packages
```

Instead:

```text
pyproject.toml
    =
what my project requires
```

And:

```text
uv.lock
    =
what uv resolved
```

And:

```text
.venv
    =
the environment containing the installed packages
```

---

# 9. Commands Learned

### Add a dependency

```bash
uv add requests
```

### Add a specific version

```bash
uv add "requests==2.32.0"
```

### View dependency tree

```bash
uv tree
```

### Inspect project configuration

```bash
cat pyproject.toml
```

### Synchronize environment

```bash
uv sync
```

---

# 10. Key Takeaways

1. `pyproject.toml` contains the project's declared dependencies.

2. Dependencies explicitly declared by our project are **direct dependencies**.

3. Dependencies required by those packages are **transitive dependencies**.

4. `uv tree` shows the resolved dependency graph.

5. `uv.lock` records the result of dependency resolution.

6. `.venv` contains the packages installed for the current project environment.

7. Changing a dependency requirement can cause `uv` to re-resolve the dependency graph.

8. The dependency flow is:

```text
pyproject.toml
      ↓
    resolve
      ↓
   uv.lock
      ↓
    .venv
```

9. `pyproject.toml` describes **what the project allows/requires**, while `uv.lock` records **what was resolved**.