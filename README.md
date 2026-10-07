# Learn uv in One Week

A short, hands-on plan for learning [uv](https://docs.astral.sh/uv/), the fast Python package and project manager.
**1 hour a day for 7 days.** Each day has a little reading and mostly typing commands.

## How each 1-hour session works

| Time      | What you do                                     |
|-----------|-------------------------------------------------|
| 10 min    | Read the "Concept" section                      |
| 40 min    | Do the "Hands-on" steps in `playground/`        |
| 10 min    | Answer the "Check yourself" questions + notes   |

Do all exercises inside `playground/` (it is git-ignored).

## The week

| Day | Topic                                   | You will be able to…                                  |
|-----|-----------------------------------------|-------------------------------------------------------|
| 1   | [Setup & first run](days/day1_setup.md)              | Install uv, manage Python versions, run a script      |
| 2   | [Projects](days/day2_projects.md)                    | Create a project, add packages, understand `uv.lock`  |
| 3   | [Dependencies](days/day3_dependencies.md)            | Use dev deps, groups, upgrades, `uv tree`, export     |
| 4   | [Single-file scripts](days/day4_scripts.md)          | Write scripts that carry their own dependencies       |
| 5   | [Tools & pip interface](days/day5_tools_and_pip.md)  | Use `uvx`, install CLI tools, migrate from pip        |
| 6   | [Packaging & workspaces](days/day6_packaging.md)     | Build a package, use workspaces with many packages    |
| 7   | [Real use cases](days/day7_real_world.md)            | Use uv in Docker and CI, build a small capstone app   |

Quick reference: [cheatsheet.md](cheatsheet.md)

## Work network note (TLS proxy)

On a company network you might see `invalid peer certificate: UnknownIssuer`.
Tell uv to trust your system certificates:

```bash
echo 'export UV_SYSTEM_CERTS=1' >> ~/.zshrc && source ~/.zshrc
```

Tested with uv 0.12.0.
