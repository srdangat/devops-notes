# GitHub Actions Cheatsheet

---

## Workflow Structure

```yaml
name: my-workflow
on:
  push:
    branches: [main]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: echo "Hello!"
```

---

## Triggers

```yaml
on:
  push:
    branches: [main]           # on push to main
  pull_request:
    branches: [main]           # on PR to main
  schedule:
    - cron: "0 0 * * *"        # daily at midnight
  workflow_dispatch:            # manual trigger
```

---

## Jobs & Dependencies

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - run: npm test

  deploy:
    needs: test                 # waits for test to pass
    runs-on: ubuntu-latest
    steps:
      - run: ./deploy.sh
```

---

## Common Actions

```yaml
- uses: actions/checkout@v4                        # clone repo
- uses: actions/setup-node@v4
  with:
    node-version: '18'                             # setup Node
- uses: actions/setup-python@v5
  with:
    python-version: '3.11'                         # setup Python
```

---

## Env Variables & Secrets

```yaml
env:
  APP_NAME: my-app                                 # workflow-level

jobs:
  build:
    env:
      MODE: release                                # job-level
    steps:
      - env:
          GREETING: hello                          # step-level
        run: echo "$GREETING $APP_NAME"

      - run: ./deploy.sh
        env:
          API_KEY: ${{ secrets.API_KEY }}          # secret
```

---

## Expressions & Contexts

```yaml
${{ github.ref_name }}          # branch name
${{ github.sha }}               # commit SHA
${{ github.actor }}             # who triggered it
${{ runner.os }}                # Linux / Windows / macOS
${{ secrets.MY_SECRET }}        # secret value
${{ env.MY_VAR }}               # env variable
${{ steps.my_step.outputs.x }}  # output from a step
```

### Conditionals

```yaml
- run: ./deploy.sh
  if: github.ref_name == 'main'

- run: echo "failed!"
  if: failure()

- run: ./cleanup.sh
  if: always()
```

---

## Artifacts

```yaml
- uses: actions/upload-artifact@v4
  with:
    name: build-output
    path: dist/

- uses: actions/download-artifact@v4
  with:
    name: build-output
    path: dist/
```

---

## Caching

```yaml
- uses: actions/cache@v4
  with:
    path: node_modules
    key: node-${{ runner.os }}-${{ hashFiles('package-lock.json') }}
    restore-keys: |
      node-${{ runner.os }}-
```

---

## Matrix Strategy

```yaml
strategy:
  fail-fast: false
  matrix:
    os: [ubuntu-latest, windows-latest]
    node: [18, 20]

runs-on: ${{ matrix.os }}
# Access with ${{ matrix.node }}
```

---

## Step Outputs

```yaml
- name: Set output
  id: my_step
  run: echo "result=42" >> $GITHUB_OUTPUT

- name: Use output
  run: echo "${{ steps.my_step.outputs.result }}"
```

---

## Built-in Functions

| Function | Use |
|---|---|
| `success()` | all previous steps passed |
| `failure()` | any previous step failed |
| `always()` | run regardless of outcome |
| `hashFiles('file')` | hash of a file (for cache keys) |
| `contains(str, sub)` | string contains check |
| `startsWith(str, pre)` | string prefix check |
