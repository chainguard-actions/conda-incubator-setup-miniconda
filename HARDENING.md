<!-- markdownlint-disable -->

# Hardening Report: conda-incubator--setup-miniconda/v3.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **conda-incubator--setup-miniconda/v3.3.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files use `uses:` references pinned to mutable tags or version strings instead of immutable 40-character commit SHAs. Affected references include: `actions/checkout@v6.0.1` (tag) in all example workflows, `actions/cache@v5` (tag) in caching-envs-example.yml and caching-example.yml, and `actions/setup-node@v6` (tag) in lint.yml. These can be silently replaced by a supply-chain attacker.

Locations:

- `.github/workflows/caching-envs-example.yml:31`
- `.github/workflows/caching-envs-example.yml:44`
- `.github/workflows/caching-example.yml:29`
- `.github/workflows/caching-example.yml:33`
- `.github/workflows/caching-example.yml:55`
- `.github/workflows/caching-example.yml:62`
- `.github/workflows/example-1.yml:34`
- `.github/workflows/example-2.yml:30`
- `.github/workflows/example-2.yml:46`
- `.github/workflows/example-2.yml:62`
- `.github/workflows/example-3.yml:30`
- `.github/workflows/example-3.yml:50`
- `.github/workflows/example-4.yml:30`
- `.github/workflows/example-5.yml:33`
- `.github/workflows/example-5.yml:57`
- `.github/workflows/example-5.yml:75`
- `.github/workflows/example-6.yml:58`
- `.github/workflows/example-7.yml:33`
- `.github/workflows/example-8.yml:33`
- `.github/workflows/example-9.yml:33`
- `.github/workflows/example-10.yml:33`
- `.github/workflows/example-10.yml:57`
- `.github/workflows/example-11.yml:36`
- `.github/workflows/example-12.yml:30`
- `.github/workflows/example-13.yml:34`
- `.github/workflows/example-14.yml:42`
- `.github/workflows/example-15.yml:30`
- `.github/workflows/lint.yml:22`
- `.github/workflows/lint.yml:24`
- `.github/workflows/lint.yml:35`
- `.github/workflows/lint.yml:37`
- `.github/workflows/regression-checks.yml:33`
- `.github/workflows/regression-checks.yml:57`
- `.github/workflows/regression-checks.yml:93`

### missing-permissions (severity: medium)

No workflow file under .github/workflows/ defines a top-level `permissions:` key, and no individual job defines its own `permissions:` block. Without explicit permissions, workflows run with the default (potentially broad) token permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/caching-envs-example.yml:1`
- `.github/workflows/caching-example.yml:1`
- `.github/workflows/example-1.yml:1`
- `.github/workflows/example-2.yml:1`
- `.github/workflows/example-3.yml:1`
- `.github/workflows/example-4.yml:1`
- `.github/workflows/example-5.yml:1`
- `.github/workflows/example-6.yml:1`
- `.github/workflows/example-7.yml:1`
- `.github/workflows/example-8.yml:1`
- `.github/workflows/example-9.yml:1`
- `.github/workflows/example-10.yml:1`
- `.github/workflows/example-11.yml:1`
- `.github/workflows/example-12.yml:1`
- `.github/workflows/example-13.yml:1`
- `.github/workflows/example-14.yml:1`
- `.github/workflows/example-15.yml:1`
- `.github/workflows/example-16.yml:1`
- `.github/workflows/lint.yml:1`
- `.github/workflows/regression-checks.yml:1`

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate `${{ }}` expressions into shell commands (sub-rule a), allowing an attacker to inject arbitrary shell commands. Specific violations:

1. example-8.yml: `run: ${{ steps.setup-miniconda.outcome == 'failure' }}` — the entire run value is a template expression evaluated as a shell command.

2. example-11.yml: `if [[ ${{ matrix.os }} == 'ubuntu-latest' ]]; then` and `${{ steps.setup-miniconda.outcome == 'failure' }}` / `${{ steps.setup-miniconda.outcome == 'success' }}` — matrix and step-output expressions interpolated directly into bash.

3. example-14.yml: Inside a Python heredoc, `"${{ matrix.channels }}".split(",")` and `"${{ matrix.conda-remove-defaults }}" == "true"` — matrix values injected directly into Python source code before execution.

4. regression-checks.yml: `python -c "import sys; assert ... == '${{ matrix.python-version }}'"` — matrix value injected into a Python one-liner (appears in both issue-114 and issue-261 jobs).

Locations:

- `.github/workflows/example-8.yml:47`
- `.github/workflows/example-11.yml:47`
- `.github/workflows/example-11.yml:48`
- `.github/workflows/example-11.yml:50`
- `.github/workflows/example-14.yml:57`
- `.github/workflows/example-14.yml:58`
- `.github/workflows/regression-checks.yml:46`
- `.github/workflows/regression-checks.yml:103`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three finding types across 20 workflow files:

1. unpinned-uses: Pinned actions/checkout@v6.0.1 → SHA 8e8c483db84b4bee98b60c0593521ed34d9990e8, actions/cache@v5 → SHA caa296126883cff596d87d8935842f9db880ef25, actions/setup-node@v6 → SHA 249970729cb0ef3589644e2896645e5dc5ba9c38 across all affected files.

2. missing-permissions: Added `permissions: {}` top-level block to all 20 workflow files.

3. script-injection: (a) example-8.yml: moved steps.setup-miniconda.outcome to env var OUTCOME, replaced bare template expression run command with `[ "$OUTCOME" = "failure" ]`; (b) example-11.yml: moved matrix.os and outcome expressions to env vars MATRIX_OS/OUTCOME, rewrote conditional using those vars; (c) example-14.yml: moved matrix.channels and matrix.conda-remove-defaults to env vars INPUT_CHANNELS/INPUT_REMOVE_DEFAULTS, updated Python heredoc to use os.environ instead of inline template expressions; (d) regression-checks.yml: moved matrix.python-version to env var PYTHON_VERSION in both issue-114 and issue-261 jobs, updated python -c assertions to use os.environ['PYTHON_VERSION'].

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in .github/workflows/example-5.yml at line 40. Moved `${{ github.run_number }}` out of the `run:` shell command and into an `env:` block as `GITHUB_RUN_NUMBER: ${{ github.run_number }}`. The shell script now references it as `${GITHUB_RUN_NUMBER}` instead of directly interpolating the expression. The remaining `${{ github.run_number }}` usages in `with:` input values are not shell injection vectors and were left unchanged.

