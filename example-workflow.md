# GitHub Actions Workflow Reference

A repeatable structure for building CI/CD workflows.

---

## Anatomy of a workflow file

```
.github/
  workflows/
    your-workflow.yml
```

Every workflow file has three top-level sections:

```yaml
name: <displayed in GitHub UI>

on: <events that trigger the workflow>

jobs: <the actual work>
```

---

## Full annotated template

```yaml
name: My Workflow                          # Shown in the GitHub Actions tab

# ── TRIGGERS ──────────────────────────────────────────────────────────────────
on:
  pull_request:                            # Runs on every PR
    branches:
      - main                              # Only when targeting main
    paths:                                # Optional: only if these paths changed
      - '**.sh'
      - 'src/**'

  push:                                   # Also runs on direct pushes
    branches:
      - main

  workflow_dispatch:                       # Adds a manual "Run workflow" button in the UI

# ── JOBS ──────────────────────────────────────────────────────────────────────
jobs:
  my-job:                                  # Job ID (used for dependencies between jobs)
    name: Human-readable job name          # Shown in the GitHub UI
    runs-on: ubuntu-latest                 # The VM runner (see Runner Types below)

    # Optional: only run this job if a condition is true
    if: github.event_name == 'pull_request'

    # Optional: environment variables available to all steps in this job
    env:
      MY_VAR: hello

    steps:
      # ── STEP 1: Checkout ──────────────────────────────────────────────────
      # Almost always the first step. Clones your repo onto the runner.
      - name: Checkout code
        uses: actions/checkout@v4          # uses: runs a pre-built Action from the Marketplace

      # ── STEP 2: Use a Marketplace Action ─────────────────────────────────
      - name: Run some tool
        uses: owner/action-name@version
        with:                             # Inputs the Action accepts (check its README)
          option_one: value
          option_two: true

      # ── STEP 3: Run shell commands directly ───────────────────────────────
      - name: Run a script
        run: |
          echo "This runs in bash on the runner"
          ./my-script.sh

      # ── STEP 4: Use output from a previous step ───────────────────────────
      - name: Get a value
        id: my-step                        # Give the step an ID to reference its output
        run: echo "result=42" >> $GITHUB_OUTPUT

      - name: Use that value
        run: echo "The result was ${{ steps.my-step.outputs.result }}"

  # ── SECOND JOB (depends on the first) ─────────────────────────────────────
  downstream-job:
    name: Runs after my-job
    runs-on: ubuntu-latest
    needs: my-job                          # Won't start until my-job succeeds

    steps:
      - name: Do something after
        run: echo "my-job passed, now I run"
```

---

## Key concepts

### Triggers (`on`)

| Trigger | When it fires |
|---|---|
| `pull_request` | Opened, updated, or reopened PR |
| `push` | Commit pushed to a branch |
| `workflow_dispatch` | Manual button in the GitHub UI |
| `schedule` | Cron syntax: `cron: '0 9 * * 1'` (Mon 9am UTC) |
| `release` | When a GitHub release is published |

### Runner types (`runs-on`)

| Runner | Use for |
|---|---|
| `ubuntu-latest` | Default choice — fast and cheap |
| `macos-latest` | macOS-specific testing |
| `windows-latest` | Windows-specific testing |
| `self-hosted` | Your own machine (needs a runner agent installed) |

### `uses` vs `run`

| Keyword | What it does |
|---|---|
| `uses: actions/checkout@v4` | Runs a pre-built Action from the Marketplace |
| `run: ./my-script.sh` | Runs raw shell commands on the runner |

Always pin Actions to a version tag (`@v4`) or a commit SHA (`@abc1234`) — never use `@master` in production as it can silently change.

### Contexts and expressions

GitHub exposes runtime data through contexts, accessed with `${{ }}`:

```yaml
${{ github.actor }}          # Who triggered the workflow
${{ github.ref }}            # Branch or tag ref (e.g. refs/heads/main)
${{ github.sha }}            # Commit SHA
${{ github.event_name }}     # 'pull_request', 'push', etc.
${{ secrets.MY_SECRET }}     # Secrets set in repo Settings → Secrets
${{ env.MY_VAR }}            # Environment variable set in the workflow
${{ steps.my-step.outputs.result }}  # Output from a named step
```

### Secrets

Store sensitive values in **GitHub → repo Settings → Secrets and variables → Actions**, then reference them as:

```yaml
env:
  API_KEY: ${{ secrets.MY_API_KEY }}
```

Never hardcode secrets in the workflow file.

---

## Making a check required to merge

After the workflow runs at least once on a PR:

1. Go to **repo Settings → Branches → Branch protection rules**
2. Edit (or create) the rule for `main`
3. Enable **Require status checks to pass before merging**
4. Search for and add the job name (e.g. `Lint shell scripts`)
5. Optionally enable **Require branches to be up to date before merging**

The job name that appears in the status check list matches the `name:` field inside the job, not the workflow name.

---

## Common patterns

### Matrix builds (run a job across multiple versions)

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18, 20, 22]
    steps:
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm test
```

### Cache dependencies (speeds up repeat runs)

```yaml
- uses: actions/cache@v4
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
```

### Upload an artifact (save files produced by the job)

```yaml
- uses: actions/upload-artifact@v4
  with:
    name: my-report
    path: output/report.html
```

### Conditional step

```yaml
- name: Only on main
  if: github.ref == 'refs/heads/main'
  run: ./deploy.sh
```

---

## This repo's workflow

`.github/workflows/shellcheck.yml` — runs ShellCheck on every PR targeting `main`.

To add a new check, create a new `.yml` file in `.github/workflows/`. Each file is an independent workflow.
