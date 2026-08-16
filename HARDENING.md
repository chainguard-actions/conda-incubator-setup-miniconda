<!-- markdownlint-disable -->

# Hardening Report: conda-incubator--setup-miniconda/v4.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **conda-incubator--setup-miniconda/v4.0.1** was hardened automatically. 3 finding(s) were identified and resolved across 4 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation inside run: blocks. In docs.yml, ${{ github.event.pull_request.number }} is interpolated directly into shell command arguments (--alias and --message flags of netlify deploy), allowing an attacker to inject shell metacharacters via a PR number. In example-11.yml, ${{ matrix.os }} and ${{ steps.setup-miniconda.outcome == '...' }} are interpolated directly inside a bash run: block. In example-14.yml, ${{ matrix.channels }} and ${{ matrix.conda-remove-defaults }} are interpolated directly inside a Python heredoc run: block.

Locations:

- `.github/workflows/docs.yml:68`
- `.github/workflows/docs.yml:69`
- `.github/workflows/example-11.yml:52`
- `.github/workflows/example-11.yml:53`
- `.github/workflows/example-14.yml:63`
- `.github/workflows/example-14.yml:64`

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable tag-based refs instead of immutable 40-character SHA digests, making them vulnerable to supply-chain attacks. Unpinned references include: actions/checkout@v6.0.2, actions/setup-node@v6, actions/cache@v5, actions/upload-pages-artifact@v5, actions/deploy-pages@v5, marocchino/sticky-pull-request-comment@v3, codecov/codecov-action@v6, actions/upload-artifact@v7. These appear across caching-envs-example.yml, caching-example.yml, docs.yml, example-1.yml through example-15.yml, example-17.yml, lint.yml, regression-checks.yml, and tests.yml.

Locations:

- `.github/workflows/caching-envs-example.yml:30`
- `.github/workflows/caching-envs-example.yml:44`
- `.github/workflows/caching-example.yml:28`
- `.github/workflows/caching-example.yml:33`
- `.github/workflows/caching-example.yml:64`
- `.github/workflows/docs.yml:38`
- `.github/workflows/docs.yml:39`
- `.github/workflows/docs.yml:48`
- `.github/workflows/docs.yml:53`
- `.github/workflows/docs.yml:75`
- `.github/workflows/example-1.yml:35`
- `.github/workflows/example-2.yml:28`
- `.github/workflows/example-3.yml:30`
- `.github/workflows/example-4.yml:30`
- `.github/workflows/example-5.yml:35`
- `.github/workflows/example-6.yml:52`
- `.github/workflows/example-7.yml:35`
- `.github/workflows/example-8.yml:36`
- `.github/workflows/example-9.yml:36`
- `.github/workflows/example-10.yml:34`
- `.github/workflows/example-11.yml:39`
- `.github/workflows/example-12.yml:35`
- `.github/workflows/example-13.yml:41`
- `.github/workflows/example-14.yml:39`
- `.github/workflows/example-15.yml:35`
- `.github/workflows/example-17.yml:30`
- `.github/workflows/lint.yml:22`
- `.github/workflows/lint.yml:32`
- `.github/workflows/regression-checks.yml:35`
- `.github/workflows/tests.yml:28`
- `.github/workflows/tests.yml:29`
- `.github/workflows/tests.yml:32`
- `.github/workflows/tests.yml:34`

### missing-permissions (severity: medium)

21 workflow files have no top-level permissions: key and no job-level permissions on every job. Without explicit permissions, workflows run with the default token permissions (which may include write access to repository contents), violating the principle of least privilege. Affected files: caching-envs-example.yml, caching-example.yml, example-1.yml through example-17.yml, lint.yml, and regression-checks.yml.

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
- `.github/workflows/example-17.yml:1`
- `.github/workflows/lint.yml:1`
- `.github/workflows/regression-checks.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three finding types across 23 workflow files:

1. script-injection: Fixed in docs.yml (PR number moved to env var PR_NUMBER), example-11.yml (matrix.os and setup-miniconda.outcome moved to env vars with proper bash comparisons), example-14.yml (matrix.channels and matrix.conda-remove-defaults moved to env vars, Python script uses os.environ instead of string interpolation). Also fixed example-8.yml which had the same pattern.

2. unpinned-uses: Pinned all mutable tag-based action refs to full 40-char SHAs: actions/checkout@v6.0.2→de0fac2e, actions/setup-node@v6→249970729, actions/cache@v5→caa296126, actions/upload-pages-artifact@v5→fc324d35, actions/deploy-pages@v5→cd2ce8fc, marocchino/sticky-pull-request-comment@v3→5770ad5e, codecov/codecov-action@v6→fb8b3582, actions/upload-artifact@v7→043fb46d.

3. missing-permissions: Added 'permissions: contents: read' top-level block to all 21 affected files (caching-envs-example.yml, caching-example.yml, example-1.yml through example-17.yml, lint.yml, regression-checks.yml). docs.yml and tests.yml already had permissions blocks.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed all three script-injection findings:
1. regression-checks.yml (issue-114 job, ~line 53): Moved `${{ matrix.python-version }}` to step `env: PYTHON_VERSION:` block; updated Python assertion to use `os.environ['PYTHON_VERSION']` instead of shell interpolation.
2. regression-checks.yml (issue-261 job, ~line 107): Same fix applied to the identical pattern in the second job.
3. example-5.yml (example-5-linux job, line 40): Moved `${{ github.run_number }}` to step `env: RUN_NUMBER:` block; updated curl command to use `${RUN_NUMBER}` instead of the template expression.
The `installer-url:` `with:` field references to `github.run_number` are YAML action inputs (not shell `run:` commands) and are not script-injection vectors, so they were left unchanged.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed two script-injection findings by quoting unquoted variable expansions:
1. hardened/action/.github/workflows/example-5.yml (line 43): Changed `curl -L ${MINIFORGE_URL} > /tmp/some-built-installer-${RUN_NUMBER}.sh` to `curl -L "${MINIFORGE_URL}" > "/tmp/some-built-installer-${RUN_NUMBER}.sh"` — properly quotes both ${MINIFORGE_URL} and ${RUN_NUMBER} (sourced from github.run_number).
2. hardened/action/.github/workflows/example-16.yml (line 123): Changed `mkdir -p ${TEST_DIR}` to `mkdir -p "${TEST_DIR}"` — properly quotes ${TEST_DIR} (sourced from runner.temp) to prevent word splitting and metacharacter injection.

### Iteration 4

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Determine installer file name' step in .github/workflows/example-16.yml (line 97). Added newline sanitization before writing to GITHUB_OUTPUT: introduced a `safe` variable computed via `printf '%s' "$INSTALLER_FILE" | tr -d '\n\r'` and used `safe` in the echo statement instead of the raw `INSTALLER_FILE`. Also properly quoted `"${GITHUB_OUTPUT}"` in the redirection.

