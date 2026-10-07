# uv — `uv.lock`, `uv sync`, and Recreating the Environment

## 1. Why Does `uv.lock` Exist?

`pyproject.toml` describes the project's dependency requirements.

For example:

```toml
[project]
dependencies = [
    "requests==2.32.0",
]
```

The project says:

> I need `requests` version 2.32.0.

However, `requests` itself has dependencies:

```text
requests
├── certifi
├── charset-normalizer
├── idna
└── urllib3
```

`uv` resolves the complete dependency graph and records the resolved dependency state in:

```text
uv.lock
```

---

# 2. `pyproject.toml` vs `uv.lock`

These files have different purposes.

### `pyproject.toml`

Describes:

> **What the project requires**

Example:

```toml
dependencies = [
    "requests>=2.34.2",
]
```

This is a project-level dependency requirement.

---

### `uv.lock`

Describes:

> **What dependency versions were resolved**

For example:

```text
requests 2.34.2
certifi ...
charset-normalizer ...
idna ...
urllib3 ...
```

The lock file represents the resolved dependency graph.

---

# 3. Dependency Resolution Flow

The overall flow is:

```text
pyproject.toml
       │
       │ Project requirements
       ▼
   UV Resolver
       │
       │ Resolves dependency graph
       ▼
    uv.lock
       │
       │ Used to construct environment
       ▼
     .venv
       │
       ▼
Installed packages
```

A useful mental model:

```text
pyproject.toml = What I require

uv.lock        = What uv resolved

.venv          = What is installed
```

---

# 4. `uv.lock` Is Not the Virtual Environment

These are completely different things.

### `uv.lock`

A file containing dependency resolution information.

```text
uv.lock
```

### `.venv`

An actual Python virtual environment containing installed packages.

```text
.venv/
├── bin/
├── lib/
└── ...
```

Therefore:

```text
uv.lock ≠ .venv
```

---

# 5. Deleting `.venv`

A virtual environment can be deleted.

Example:

```bash
rm -rf .venv
```

After deletion:

```text
uv_learning/
├── .git/
├── .gitignore
├── .python-version
├── pyproject.toml
├── src/
└── uv.lock
```

The dependency information is still available.

---

# 6. Recreating `.venv`

Run:

```bash
uv sync
```

`uv` creates the virtual environment again and installs the project's dependencies.

Example output:

```text
Using CPython 3.12.13
Creating virtual environment at: .venv
Resolved 6 packages
Installed 6 packages
```

The resulting environment contains:

```text
certifi
charset-normalizer
idna
requests
urllib3
uv-learning
```

The important point is:

> `.venv` is reproducible and can be recreated when needed.

---

# 7. Why Commit `uv.lock` to Git?

For an application/project, the lock file is normally committed to the repository.

Example:

```text
Git repository
│
├── pyproject.toml       ✅ commit
├── uv.lock              ✅ commit
├── .python-version      ✅ usually commit
├── src/                 ✅ commit
└── .venv/               ❌ don't commit
```

The reason is that `uv.lock` is part of the project's dependency state.

Another developer can clone the repository and run:

```bash
uv sync
```

to recreate the environment.

---

# 8. Why Don't We Commit `.venv`?

`.venv` contains the actual installed environment.

It is:

- machine-specific
- generated
- potentially large
- reproducible from the project configuration and lock file

Therefore, `.venv` should normally be excluded through `.gitignore`.

Example:

```gitignore
.venv/
```

Instead of committing `.venv`, commit:

```text
pyproject.toml
uv.lock
```

---

# 9. `uv lock`

Command:

```bash
uv lock
```

Purpose:

> Resolve/update the project's lock file.

Example:

```bash
uv lock
```

Output:

```text
Resolved 6 packages in 18ms
```

Important:

`uv lock` is primarily about the **lock file**.

It does not mean:

> Install everything into my environment.

---

# 10. `uv sync`

Command:

```bash
uv sync
```

Purpose:

> Synchronize the virtual environment with the project's dependency state.

It can:

- create `.venv`
- install dependencies
- update the environment
- remove packages that shouldn't be present
- use the project's resolved dependency state

Example:

```bash
uv sync
```

If `.venv` does not exist:

```text
Creating virtual environment at: .venv
```

Then packages are installed.

---

# 11. Git and uv Are Separate

Running:

```bash
uv lock
```

does not automatically modify Git tracking.

For example:

```bash
git status
```

may show:

```text
Untracked files:

    uv.lock
```

This is because:

```text
uv
│
└── manages Python dependencies

Git
│
└── manages source-code/version history
```

To track the lock file:

```bash
git add uv.lock
```

---

# 12. Practical Experiment

### Step 1 — Check Git

```bash
git status
```

Initially:

```text
uv.lock
```

may appear as an untracked file.

---

### Step 2 — Remove the environment

```bash
rm -rf .venv
```

Verify:

```bash
ls -la
```

`.venv` should be gone.

---

### Step 3 — Recreate the environment

```bash
uv sync
```

Output:

```text
Creating virtual environment at: .venv
Installed 6 packages
```

Verify:

```bash
ls -la
```

`.venv` should exist again.

---

# 13. Complete Mental Model

Think of a `uv` project like this:

```text
                  PROJECT
                     │
                     ▼
             pyproject.toml
                     │
                     │
              requirements
                     │
                     ▼
               UV RESOLVER
                     │
                     ▼
                  uv.lock
                     │
                     │
              resolved state
                     │
                     ▼
                  uv sync
                     │
                     ▼
                   .venv
                     │
                     ▼
             installed packages
```

Git sits alongside this process:

```text
                 Git Repository
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
   pyproject.toml   uv.lock       src/
       commit         commit       commit

                     .venv
                       │
                       ▼
                  DO NOT COMMIT
```

---

# 14. Key Takeaways

1. `pyproject.toml` describes project requirements.

2. `uv.lock` records the resolved dependency state.

3. `.venv` contains the actual installed environment.

4. `uv.lock` and `.venv` are different things.

5. `.venv` can be deleted and recreated.

6. `uv sync` can recreate `.venv`.

7. `uv lock` focuses on resolving/updating the lock file.

8. `uv sync` focuses on synchronizing the environment.

9. `uv.lock` should normally be committed for application projects.

10. `.venv` should normally not be committed.

11. Git and `uv` are independent systems.

12. The core flow is:

```text
pyproject.toml
       ↓
   uv resolver
       ↓
    uv.lock
       ↓
    uv sync
       ↓
     .venv
```