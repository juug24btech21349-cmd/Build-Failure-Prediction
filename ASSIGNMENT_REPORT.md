# Build-Failure Prediction — Assignment Report

**Repository:** [juug24btech21349-cmd/Build-Failure-Prediction](https://github.com/juug24btech21349-cmd/Build-Failure-Prediction)  
**Scope:** Labs 0–2 from the supplied assignment format  
**Project:** `smartci` — a small Python application verified by automated tests and GitHub Actions

## Objective

The purpose of these labs is to create a reproducible Python project, write
tests for its behavior, and use continuous integration (CI) to detect a
regression automatically. A red CI run is the controlled failure signal in
this exercise: it demonstrates that the pipeline detects a known-bad change.
These labs establish a CI foundation for build-failure prediction; they do not
train or evaluate a machine-learning prediction model.

## Lab 0: Setup and project folder

### Steps

1. Install Visual Studio Code, Python 3.12, and Git. In VS Code, install the
   Python and GitHub Copilot extensions and sign in to Copilot.
2. Create and open the project folder named `smartci` (this assignment is
   stored in the supplied Build-Failure-Prediction repository).
3. Open the integrated terminal and create/activate a virtual environment:

   ```powershell
   python -m venv .venv
   .venv\Scripts\activate
   ```

4. Use **Python: Select Interpreter** from the Command Palette and select the
   interpreter inside `.venv`.
5. Create the project folders: `src`, `tests`, `scripts`, `ml`, `infra`,
   `ansible`, and `.github/workflows`.
6. Initialize Git (if the checkout is not already initialized) and add a
   `.gitignore` that excludes `.venv/`, `__pycache__/`, `.pytest_cache/`, and
   `build.log`.
7. Create or use the GitHub repository linked above. After adding the project,
   push the `main` branch so the repository can be viewed and Actions can run.
8. Copilot basics: accept an inline suggestion with Tab and open Copilot Chat
   with Ctrl+Alt+I. Review generated code and test assertions rather than
   accepting them without checking.

### What I learned

- A virtual environment isolates project dependencies from other Python
  projects.
- Git tracks source changes, while `.gitignore` keeps generated files and local
  environment data out of version control.
- Copilot can speed up drafting, but its output still needs to be checked
  against the requirement and expected behavior.

## Lab 1: The app and its tests

### Implementation steps

1. `src/pricing.py` provides `apply_discount(price, percent)`, rejects a
   percentage outside 0–100 with `ValueError`, and rounds the discounted price
   to two decimal places. `add_tax(price, rate=0.18)` applies the default 18%
   tax and rounds the result.
2. `src/cart.py` defines `Cart`. Items are stored as `(name, price, quantity)`
   tuples. `total()` sums item prices times quantities, applies the requested
   discount, and then adds tax.
3. `src/inventory.py` defines `Inventory`. Stock can be added or removed;
   removing more than the available amount raises `ValueError`.
4. `src/__init__.py` makes `src` a Python package.
5. The tests in `tests/` cover ordinary behavior, discount boundaries, invalid
   percentages, tax, a discounted cart, an empty cart, stock changes, and
   insufficient stock.
6. `requirements.txt` lists pytest and requests, and `pytest.ini` adds the
   project root to pytest's import path.
7. Install dependencies and run the test suite:

   ```powershell
   python -m pip install -r requirements.txt
   pytest -v
   ```

### What I learned

- Unit tests check small, specific behaviors and make regressions easier to
  locate.
- Boundary and invalid-input tests are important in addition to the basic
  success case.
- `Cart` imports the pricing functions, so the modules have a dependency:
  changing pricing behavior can affect cart totals.
- The expected discounted cart total is `106.20`: two pens at 50 each make a
  subtotal of 100, a 10% discount gives 90, and 18% tax gives 106.20.

### Checkpoint

The checkpoint is successful when `pytest -v` completes with all tests passing.
The tests are kept in the repository so the same checks can run locally and in
CI.

## Lab 2: Baseline CI pipeline

### Workflow steps

The workflow is defined in `.github/workflows/ci.yml`.

1. `on: [push, pull_request]` starts the workflow for pushes and pull requests.
2. The `build` job runs on a fresh `ubuntu-latest` GitHub-hosted runner.
3. `actions/checkout@v4` checks out the project. `fetch-depth: 0` retrieves the
   full Git history, which is useful for later history/diff-based exercises.
4. `actions/setup-python@v5` configures Python 3.12.
5. The dependency-install step installs packages from `requirements.txt`.
6. The **Run tests** step runs pytest with verbose output and saves the output
   to `build.log`. `set -o pipefail` ensures a pytest failure still fails the
   step even though its output is piped through `tee`.

### Why a red run is useful

A red run means the workflow detected that a check failed. In this experiment,
temporarily changing an expected test value from `100.0` to `101.0` makes the
test assertion fail. The red run proves that the pipeline is checking the code
and that a regression will not be reported as a successful build. The failing
assertion and test name help identify what needs investigation. After restoring
the correct expectation, a subsequent green run confirms the correction.

The intentional failure is an experiment only; the final branch should contain
the corrected test and pass CI. Do not leave a deliberately failing test in the
final version.

### Where to find the error in GitHub Actions

1. Open the repository on GitHub and select the **Actions** tab.
2. Select the red failed run in the workflow run list (the run has a red X and
   a failed status).
3. In the run summary, select the failed **build** job.
4. Expand the **Run tests** step. The console output shows the failing test,
   assertion, and pytest failure summary. The failed step is also marked with
   a red X.
5. To see the corrected result, return to **Actions** and open the later run
   with a green checkmark.

### Screenshots to attach

Capture these from the actual repository after the workflow runs; do not use
mocked or edited run results:

| Screenshot | GitHub page to capture | Suggested filename |
|---|---|---|
| Failed run overview | Actions → failed CI run, showing the red status and failed `build` job | `docs/screenshots/actions-red-run.png` |
| Failure details | Failed run → `build` → expanded **Run tests** step, with the pytest assertion in view | `docs/screenshots/actions-red-test-error.png` |
| Successful run | Actions → later successful CI run, showing the green status | `docs/screenshots/actions-green-run.png` |

Save screenshots to `docs/screenshots/` and add them to the submission. A real
GitHub Actions screenshot cannot exist until the workflow has been pushed and
run; no run output should be represented as actual evidence before that.

### What I learned

- A workflow is triggered by events and runs jobs made up of ordered steps.
- GitHub-hosted jobs use fresh runners, so dependencies must be installed as
  part of the workflow.
- CI provides repeatable checks on the hosted environment, independently of
  the developer's local machine.
- `tee` keeps test output visible while saving a log, and `pipefail` preserves
  the test command's failure status.
- A deliberate red run is evidence that the check can detect a regression;
  fixing the cause should be followed by a green run.

## Submission checklist

- [ ] Push the project to the repository linked at the top of this report.
- [ ] Confirm at least one green CI run in **Actions**.
- [ ] Perform the controlled failure experiment, capture the red run and
  failure step, then restore the expected value and push the fix.
- [ ] Confirm the later green run and attach the three screenshots listed
  above.
- [ ] Include this report with the GitHub repository link when submitting.

## Repository layout

```text
.
├── .github/workflows/ci.yml
├── ansible/
├── infra/
├── ml/
├── scripts/
├── src/
│   ├── __init__.py
│   ├── cart.py
│   ├── inventory.py
│   └── pricing.py
├── tests/
├── .gitignore
├── pytest.ini
├── requirements.txt
└── ASSIGNMENT_REPORT.md
```
