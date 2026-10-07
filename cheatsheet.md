# uv Cheat Sheet

| I want to…                         | Command                                        |
|------------------------------------|------------------------------------------------|
| Install a Python version           | `uv python install 3.12`                       |
| Pin Python for a project           | `uv python pin 3.12`                           |
| Start a new project                | `uv init myapp` (`--package` / `--lib`)        |
| Add / remove a package             | `uv add requests` / `uv remove requests`       |
| Add a dev-only package             | `uv add --dev pytest`                          |
| Install everything from the lock   | `uv sync` (`--no-dev` for prod)                |
| Run anything in the project env    | `uv run python app.py` / `uv run pytest`       |
| Upgrade one / all packages         | `uv lock --upgrade-package X` / `uv lock --upgrade` |
| See the dependency tree            | `uv tree`                                      |
| Export requirements.txt            | `uv export --format requirements-txt`          |
| Script with its own deps           | `uv add --script s.py rich` then `uv run s.py` |
| Run a tool once                    | `uvx ruff check`                               |
| Install a tool globally            | `uv tool install ruff`                         |
| pip-style install                  | `uv venv && uv pip install -r requirements.txt`|
| Migrate requirements.txt           | `uv init --bare && uv add -r requirements.txt` |
| Bump version / build / publish     | `uv version --bump minor` / `uv build` / `uv publish` |
| Format / type check / audit        | `uv format` / `uv check` / `uv audit`          |
| Clear cache                        | `uv cache clean`                               |
| Corporate TLS proxy errors         | `export UV_SYSTEM_CERTS=1`                     |
