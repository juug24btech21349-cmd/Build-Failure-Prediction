# Build-Failure Prediction

This repository contains the practical work for Labs 0–2: a small Python
application, pytest coverage, and a GitHub Actions workflow that runs the test
suite on pushes and pull requests.

## Run locally

```powershell
python -m venv .venv
.venv\Scripts\activate
python -m pip install -r requirements.txt
pytest -v
```

See [ASSIGNMENT_REPORT.md](ASSIGNMENT_REPORT.md) for the step-by-step lab
procedure, concepts learned, and the GitHub Actions failure experiment.
