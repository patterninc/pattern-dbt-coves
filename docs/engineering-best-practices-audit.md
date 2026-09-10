# Engineering Best Practices Audit — pattern-dbt-coves

| | |
|---|---|
| **Repository** | `patterninc/pattern-dbt-coves` |
| **Audit date** | 2026-09-10 |
| **Auditor** | Claude — gauge-repo skill |
| **Rubric version** | `item-credit-v1` — 2026-09-04 (`references/best-practices.md`) |

## Repo profile

`pattern-dbt-coves` is a Python CLI tool (Poetry-managed, `dbt_coves` package) that automates dbt workflows — generating sources, staging models, and property files by introspecting Snowflake, Redshift, and BigQuery warehouses. It is a Pattern-maintained fork of `datacoves/dbt-coves`, distributed as a PyPI package (published by `.github/workflows/pypi_publish.yml` on push to `main`); nothing is deployed as a running service and the repo has no AWS footprint. There is no owned database — tests seed and read external warehouses using credentials encrypted in-repo with git-secret. The fork's own commit history shows a single committer (`patterninc-gha-runner`); Backstage metadata (`backstage.yaml`) assigns ownership to the business-intelligence team. The UI surface is a terminal CLI (rich/questionary), with no browser UI. The GitHub owner was verified as `patterninc` via `gh repo view` (`nameWithOwner: patterninc/pattern-dbt-coves`), so Pattern's inherited Wiz and Toolsmith controls apply.

## Scorecard

| Metric | Value |
|--------|-------|
| **Critical gates** | **RED** |
| **Adjusted compliance** | **55.3%** |

Critical gates are RED: `AGENTS.md` (item 2) is a Gap, and required CI checks before merge (item 16) and unit tests (item 23) are Partial. Adjusted compliance is calculated independently:

`(17 Met + 0.5 × 8 Partial) / (49 total − 11 justified N/A) = 21 / 38 = 55.3%`

### Status totals

| Status | Items |
|--------|------:|
| Met | 17 |
| Partial | 8 |
| Gap | 13 |
| N/A | 11 |
| **Total** | **49** |

### Per-category breakdown

| Category | Met | Partial | Gap | N/A |
|----------|----:|--------:|----:|----:|
| Documentation & Context | 1 | 2 | 4 | 2 |
| Guardrails & Enforcement | 7 | 2 | 3 | 1 |
| Testing & Feedback Loops | 4 | 3 | 4 | 2 |
| Environment & Tooling | 5 | 1 | 1 | 6 |
| Agent dispatch | 0 | 0 | 1 | 0 |
| **Total** | **17** | **8** | **13** | **11** |

## Documentation & Context

| # | Practice | Status | Evidence | Recommendation / rationale |
|---|----------|--------|----------|----------------------------|
| 1 | Skills / reusable prompt workflows | **Gap** | No `.claude/skills/`, `.claude/commands/`, or equivalent | Capture repeatable agent tasks (e.g., "add a generate subcommand", "run the warehouse test suite") as skills. |
| 2 | AGENTS.md | **Gap** | No `AGENTS.md`, `CLAUDE.md`, or `.cursorrules` | Add `AGENTS.md` covering Poetry setup, tox/pytest invocation, pre-commit expectations, git-secret handling, and PR title conventions. **Critical gate.** |
| 3 | Architecture decision records | **Gap** | No `docs/adr/` or `docs/decisions/` | Record fork-specific decisions (why the fork exists, divergence policy from upstream `datacoves/dbt-coves`, supported adapter matrix) in `docs/adr/`. |
| 4 | Runbooks | **Partial** | Release is automated (`publish.sh`, `.github/workflows/pypi_publish.yml`, `.bumpversion.cfg`) | Document rotation of the git-secret GPG key and PyPI credentials, and a failed-release recovery procedure. |
| 5 | API contract docs | **Not applicable** | — | CLI tool with no wire API; no service consumers to keep in sync. The CLI config surface is formally schematized in `schemas/config.json`. |
| 6 | README with setup & run instructions | **Met** | `README.md` — installation, full command reference, per-adapter settings examples | — |
| 7 | Changelog with migration notes | **Partial** | `changelog/CHANGELOG.md` with towncrier tooling (`[tool.towncrier]` in `pyproject.toml`) | Changelog is stale: last entry is 1.3.0-a.28 (2023-02-10) while `pyproject.toml` is at 1.5.1. Resume generating entries at release time. |
| 8 | On-call playbooks | **Not applicable** | — | Distributed CLI package with no operated production service; nothing to page on. Release troubleshooting belongs in runbooks (item 4). |
| 9 | CODEOWNERS | **Gap** | No `.github/CODEOWNERS`; `backstage.yaml` names owner `business-intelligence` | Add CODEOWNERS mapping the repo to the business-intelligence team so the org ruleset's required review auto-assigns the right reviewers. |

## Guardrails & Enforcement

| # | Practice | Status | Evidence | Recommendation / rationale |
|---|----------|--------|----------|----------------------------|
| 10 | Linters | **Met** | flake8 in `.pre-commit-config.yaml` (with flake8-docstrings) plus `.flake8` | — |
| 11 | Formatters | **Met** | black, isort, and prettier hooks in `.pre-commit-config.yaml`; `[tool.black]` / `[tool.isort]` in `pyproject.toml` | — |
| 12 | Type checking | **Partial** | `mypy.ini` exists and mypy is a dev dependency, but the mypy pre-commit hook is commented out ("temporarily due to abundant Type Annotations warnings") and mypy runs nowhere in CI | Burn down the annotation warnings and re-enable the mypy hook, or run mypy as a non-blocking CI step to stop the drift. |
| 13 | Pre-commit hooks | **Met** | `.pre-commit-config.yaml`; CI enforces via `pre-commit/action` in `main_ci.yml` | — |
| 14 | Commit message conventions | **Met** | `pr_lint.yml` enforces conventional-commit PR title prefixes; `CONTRIBUTING.md` documents conventional commits and squash-merge policy | — |
| 15 | Branch protection rules | **Met** | Org ruleset `require-pr-review` (active, `~DEFAULT_BRANCH`): PR required, 1 approval, stale-review dismissal, deletion and force-push blocked | — |
| 16 | Required CI checks before merge | **Partial** | `main_ci.yml` runs on every PR, but the visible ruleset contains no `required_status_checks` rule; classic branch protection could not be verified (API 403) | Add "Main CI" as a required status check on `main` so a red build blocks merge. **Critical gate.** |
| 17 | Dependency allow-lists / deny-lists | **Gap** | No dependency policy config | Low priority for a small CLI fork; if adopted, express the policy as a CI check over `pyproject.toml`. |
| 18 | License compliance scanning | **Gap** | No license-check job; Apache-2.0 package redistributed via PyPI | Add a license audit step (e.g., `pip-licenses` or `licensecheck`) to `main_ci.yml`. |
| 19 | Secret scanning | **Met** | Inherited Pattern Wiz policy (owner verified `patterninc`); test credentials additionally encrypted at rest via git-secret (`.gitsecret/`) | — |
| 20 | SAST / static analysis gates | **Met** | Inherited Pattern Wiz policy (owner verified `patterninc`) | — |
| 21 | Max complexity limits | **Gap** | `.flake8` sets no `max-complexity`; no radon/xenon | Add `max-complexity` to `.flake8` (flake8's mccabe checker is already available). |
| 22 | Import boundary enforcement | **Not applicable** | — | Single small package with a shallow module tree (`core`/`tasks`/`utils`/`ui`); no architectural layers whose boundaries need mechanical enforcement. |

## Testing & Feedback Loops

| # | Practice | Status | Evidence | Recommendation / rationale |
|---|----------|--------|----------|----------------------------|
| 23 | Unit tests | **Partial** | `tests/dbt_coves_flags_test.py` (one test); pytest via tox in CI | Only the flags parser is unit-tested; core generation/config logic has no isolated coverage (Codecov target is 10%). Add unit tests around `dbt_coves/tasks/generate` and `dbt_coves/config`. **Critical gate.** |
| 24 | Integration tests | **Met** | `tests/generate_sources_test.py` runs against live Snowflake, Redshift, and BigQuery in CI with git-secret-decrypted credentials | — |
| 25 | Snapshot / golden-file tests | **Met** | `tests/generate_sources_cases/*/expected/` golden outputs compared per adapter case (pytest-dictsdiff) | — |
| 26 | Contract tests | **Not applicable** | — | No service API and no machine consumers; the CLI's contract with users is covered by the golden-file cases. |
| 27 | End-to-end tests | **Met** | `generate_sources_test.py` drives the full CLI flow via subprocess against real warehouses — the CLI equivalent of browser e2e | — |
| 28 | Visual regression tests | **Not applicable** | — | Terminal CLI; no visual surface to screenshot-diff. |
| 29 | Test coverage thresholds | **Partial** | `codecov.yml` (target 10%, threshold 3%, patch off); coverage uploaded with `fail_ci_if_error: false` | The gate exists but is set too low to protect anything and cannot fail the build. Raise the target as unit coverage grows and make the upload step blocking. |
| 30 | Mutation testing | **Gap** | None | Low priority until unit coverage (item 23) is meaningful; consider `mutmut` afterwards. |
| 31 | Load / performance benchmarks | **Gap** | None | Low priority: generation time against large warehouse schemas is the one user-visible performance surface; a benchmark case would catch regressions but no SLA exists. |
| 32 | Flaky test quarantine | **Gap** | No retry/quarantine mechanism; suite depends on three live warehouses | Live-warehouse tests are inherently flake-prone; add `pytest-rerunfailures` or a quarantine marker so one transient warehouse error doesn't block merges. |
| 33 | Structured CI output | **Partial** | Coverage XML produced by tox and uploaded to Codecov | Test results themselves are not machine-readable; add `--junitxml` to the pytest invocation in `tox.ini` and publish it from `main_ci.yml`. |
| 34 | Deterministic test fixtures | **Met** | Each case in `tests/generate_sources_cases/` checks in its own `input/` SQL, `settings.yml`, and `expected/` outputs; test tables are created from those inputs each run | — |
| 35 | Smoke tests for deploys | **Gap** | `pypi_publish.yml` publishes but never verifies the artifact | Add a post-publish job that `pip install dbt-coves==<new-version>` in a clean environment and runs `dbt-coves --version`. |

## Environment & Tooling

| # | Practice | Status | Evidence | Recommendation / rationale |
|---|----------|--------|----------|----------------------------|
| 36 | Devcontainer config | **Gap** | No `.devcontainer/` | The test environment (Poetry, git-secret, GPG, dbt profiles) is fiddly to assemble; a devcontainer would make it reproducible for humans and agents. |
| 37 | One-command setup | **Partial** | `poetry install --with test` covers the package; `CONTRIBUTING.md` documents pre-commit setup | Full test setup is multi-step and undocumented in one place (git-secret reveal, `tests/create_profiles.py`, `~/.dbt/profiles.yml`). Wrap it in a `make dev` / `make test-env` target mirroring the CI steps. |
| 38 | Seed scripts for local databases | **Not applicable** | — | No local database; each test case seeds the external warehouse from its checked-in `input/data.sql`. |
| 39 | MCP servers for external tools | **Met** | Toolsmith-managed MCP access (inherited, owner verified `patterninc`) | — |
| 40 | Scoped secrets per environment | **Met** | Test warehouse credentials encrypted via git-secret (`tests/.env` in `.gitsecret/paths/mapping.cfg`); CI decryption key and PyPI publish credentials held as separate GitHub Actions secrets | — |
| 41 | Preview environments per PR | **Not applicable** | — | Distributed PyPI package; there is no deployment to preview. |
| 42 | Hot-reload / watch mode | **Not applicable** | — | Interpreted Python CLI; an editable install (`poetry install`) already gives instant code-to-run feedback with no build step to watch. |
| 43 | Structured logging (JSON) | **Not applicable** | — | Interactive CLI whose rich, human-readable terminal output (`dbt_coves/utils/log.py`, RichHandler) is the product; there is no production log aggregation to query. |
| 44 | Observable traces and metrics | **Met** | Mixpanel usage telemetry (`dbt_coves/utils/tracking.py`) — the applicable observability form for a distributed CLI | — |
| 45 | Feature flags with local overrides | **Not applicable** | — | Distributed CLI; behavior is controlled by CLI flags and config (`schemas/config.json`), and releases are versioned — a runtime flag service adds nothing. |
| 46 | Database migration tooling | **Not applicable** | — | No owned database schema to migrate. |
| 47 | Dependency update automation | **Met** | Org-wide Wiz (inherited, owner verified `patterninc`); `.github/dependabot.yml` also present | — |
| 48 | Reproducible builds (lockfiles) | **Met** | `poetry.lock` committed; Poetry build backend pinned in `pyproject.toml` | — |

## Agent dispatch

| # | Practice | Status | Evidence | Recommendation / rationale |
|---|----------|--------|----------|----------------------------|
| 49 | Agent-dispatch manifest | **Gap** | No `.agents/pattern-agents.json`; repo is Pattern-owned and Backstage-onboarded (`backstage.yaml`) | Add `.agents/pattern-agents.json` with `schema_version`, `github.repo`, ClickUp list, Slack channel, and skills plugins. No `aws[]` needed — the repo has no AWS footprint. |

## Prioritized recommendations

1. **[S] Gap — AGENTS.md (item 2, critical gate):** Add `AGENTS.md` documenting Poetry setup, tox/pytest test invocation, pre-commit hooks, git-secret credential handling, conventional-commit PR titles, and the towncrier changelog workflow.
2. **[S] Partial — required CI checks (item 16, critical gate):** Make the "Main CI" job a required status check on `main` (repo ruleset or branch protection) so failing builds block merge.
3. **[M] Partial — unit tests (item 23, critical gate):** Add unit tests for `dbt_coves/tasks/generate` and `dbt_coves/config` that run without warehouse credentials, so contributors and agents get fast local feedback.
4. **[S] Gap — agent-dispatch manifest (item 49):** Add `.agents/pattern-agents.json` with GitHub, ClickUp, Slack, Datadog, and skills metadata.
5. **[S] Gap — license compliance (item 18):** Add a dependency-license audit step to `main_ci.yml`.
6. **[S] Gap — CODEOWNERS (item 9):** Map the repo to the business-intelligence team in `.github/CODEOWNERS`.
7. **[S] Gap — complexity limits (item 21):** Set `max-complexity` in `.flake8`.
8. **[S] Gap — post-publish smoke test (item 35):** After PyPI publish, install the new version in a clean environment and run `dbt-coves --version`.
9. **[S] Gap — skills (item 1):** Capture repeatable agent workflows (release, warehouse test run) as reusable skills.
10. **[S] Gap — ADRs (item 3):** Record the fork's divergence policy from upstream `datacoves/dbt-coves` in `docs/adr/`.
11. **[M] Gap — flaky test quarantine (item 32):** Add retry/quarantine handling for the live-warehouse suite (e.g., `pytest-rerunfailures`).
12. **[M] Gap — devcontainer (item 36):** Containerize the dev/test environment (Poetry, git-secret, GPG, dbt profiles).
13. **[M] Gap — dependency allow/deny policy (item 17):** Low priority; codify a package policy as a CI check if desired.
14. **[L] Gap — mutation testing (item 30):** Defer until unit coverage is meaningful; then trial `mutmut` on `dbt_coves/tasks/generate`.
15. **[S] Partial — type checking (item 12):** Re-enable the mypy pre-commit hook (or a non-blocking CI step) and burn down annotation warnings.
16. **[S] Partial — changelog (item 7):** Resume towncrier changelog generation; the log stops at 1.3.0-a.28 while the package is at 1.5.1.
17. **[S] Partial — coverage thresholds (item 29):** Raise the Codecov target beyond 10% as unit tests land and make the upload step blocking.
18. **[S] Partial — structured CI output (item 33):** Emit JUnit XML from pytest in `tox.ini` and publish it from CI.
19. **[S] Partial — runbooks (item 4):** Document GPG/PyPI credential rotation and failed-release recovery.
20. **[M] Partial — one-command setup (item 37):** Add a `make dev` target that mirrors the CI bootstrap (poetry install, git-secret reveal, profile creation).

## Declined practices

| # | Practice | Rationale |
|---|----------|-----------|
| 5 | API contract docs | CLI tool with no wire API; the config surface is schematized in `schemas/config.json`. |
| 8 | On-call playbooks | Distributed CLI package; no operated production service to page on. |
| 22 | Import boundary enforcement | Single small package with a shallow module tree; no architectural layers to protect mechanically. |
| 26 | Contract tests | No service API or machine consumers; golden-file cases cover the CLI's output contract. |
| 28 | Visual regression tests | Terminal CLI; no visual surface. |
| 38 | Seed scripts for local databases | No local database; test cases seed external warehouses from checked-in SQL. |
| 41 | Preview environments per PR | Distributed PyPI package; nothing to deploy per PR. |
| 42 | Hot-reload / watch mode | Interpreted Python CLI; editable install already gives instant feedback. |
| 43 | Structured logging | Interactive CLI whose human-readable rich output is the product; no log aggregation pipeline. |
| 45 | Feature flags | Distributed, versioned CLI; behavior is controlled by CLI flags and config. |
| 46 | Database migration tooling | No owned database schema. |

Low-priority items such as load benchmarks (31) and mutation testing (30) were deliberately kept as Gaps — planned work, not declined practices — because they would still add value for this repo.

## Beyond the checklist

- **Golden-case test architecture**: each adapter case under `tests/generate_sources_cases/` bundles its own input SQL, settings, and expected output — a pattern that makes adding a new adapter or scenario mechanical.
- **Formal config schema**: `schemas/config.json` gives agents and IDEs a machine-readable definition of every CLI setting.
- **Encrypted-in-repo test credentials**: git-secret (`.gitsecret/`) keeps warehouse credentials versioned yet encrypted, letting CI decrypt with a single GPG key.
- **Backstage onboarding**: `backstage.yaml` registers the component with cost center, environment, and team ownership labels.
- **Multi-version dbt matrix**: `tox.ini` tests against both dbt-core 1.1 and 1.5, matching the README's version-compatibility promise.
- **Issue automation**: `.github/workflows/issue.yml` auto-labels new issues; issue and PR templates are provided.
