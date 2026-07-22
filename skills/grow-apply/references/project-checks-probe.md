# Project Checks Probe Reference

Configuration signals for detecting what static checks and integration tests a
project has configured. Used by `grow-apply` Step 5 (project checks) to decide
what to offer the user.

**This table is a starting point, NOT exhaustive.** Extend it to match what you
actually find in the project. New ecosystems, new tools, and project-specific
setups all add rows.

| Config signal | Category label | Example run |
|---|---|---|
| `pyproject.toml` `[tool.ruff]` / `[tool.flake8]` / `[tool.pylint]` | lint | `ruff check <changed files>` |
| `.eslintrc*` / `eslint` in `package.json` | lint | `eslint <changed files>` |
| `golangci.yml` / `golangci-lint` config | lint | `golangci-lint run <changed files>` |
| `clippy` config / `Cargo.toml` | lint | `cargo clippy` |
| `pyproject.toml` `[tool.mypy]` / `[tool.pyright]` | type check | `mypy <changed files>` |
| `tsconfig.json` | type check | `tsc --noEmit` |
| `pyproject.toml` `[tool.pytest] markers` with integration-ish names, `tests/integration/` dir, jest test groups, `cargo test --features` | integration tests | `pytest -m integration` (etc.) |
| `[tool.black]` / `.prettierrc` | format check | `black --check` / `prettier --check` |
| `pip-audit` / `npm audit` / `cargo audit` config | dependency security scan | `pip-audit` |
| coverage gates in `pyproject.toml` / `.coveragerc` | coverage gate | `pytest --cov` |

## Rules

- **Dynamic list, not a fixed set.** Probe and list whatever you actually find.
  One project may show 1 check, another may show 5. Show what exists, nothing
  more.
- **No config found -> skip the whole step silently.** Do not prompt, do not
  mention checks that aren't configured.
- **Never hardcode a tool name in the prompt.** Use the labels and run commands
  derived from what you probed.
- **Prefer running against changed files** when the tool supports file targeting
  (ruff/eslint/mypy do). The tool's rule set, ignores, and per-file-ignores
  still take effect from the project config - targeting files only narrows
  *which files are fed to the tool*, it does not change *which rules the tool
  applies*. Run project-wide only when the tool requires it (e.g. `tsc --noEmit`
  on the whole tsconfig).
## Prerequisites (env vars / feature gates)

Some checks only run when an external resource is available - integration tests
that skip unless an env var is set, or a feature flag is on. Before offering to
run, probe whether such prerequisites exist and whether they are currently met.

**Scan test config files** (NOT only `conftest.py`) for prerequisite patterns:

| Config file | Prerequisite pattern | Example |
|---|---|---|
| `conftest.py` (any level), `pytest.ini`, `pyproject.toml [tool.pytest]`, `setup.cfg`, `tox.ini` | `os.environ.get("XXX")` followed by `pytest.skip(...)` | `if not os.environ.get("MCP_INTEGRATION_DATABASE_URL"): pytest.skip(...)` |
| `jest.config.*`, `package.json` jest config | `process.env.XXX` followed by `describe.skip`/`it.skip` | `(!process.env.PG_URL ? describe.skip : describe)("integration", ...)` |
| Go test files (`*_test.go`) near `go.mod` | `os.Getenv("XXX")` followed by `t.Skip(...)` | `if os.Getenv("INT_DB") == "" { t.Skip(...) }` |
| `Cargo.toml`, `tests/*.rs` | `#[cfg(feature="integration")]` / `env!` gates | `#[cfg(feature="integration")]` without the feature enabled |
| `pyproject.toml [tool.pytest] markers` | marker description mentioning env var / prerequisite | `markers = ["integration: 需设 XXX 环境变量"]` |

**This table is a starting point, NOT exhaustive.** Extend the patterns to match
what you actually find in the project.

For each prerequisite found, **check whether it is currently satisfied**:
- env var -> is it set in the current environment?
- feature flag -> is it in the default feature set / enabled?

**Reporting:**
- If a check's prerequisite is unmet, mark it explicitly in the offer:
  "集成测试: ... (需设 {ENV_VAR}，当前未设 -> 将被跳过)"
- After running, if a check was skipped due to unmet prerequisites, report it
  as a **false-green warning**, NOT as "passed". Exit code 0 with skipped
  integration tests is the most dangerous outcome - the user thinks integration
  is verified when it was never run.
