# Pulsefinal

Pulsefinal brings together AI-assisted workflow and dashboard and analytics capabilities based on the code in this repository.

## Overview

This README documents the current implementation of `pulsefinal`. It is based on the checked-in source files, package manifests, and entry points in the repository.

## What It Covers

- AI-assisted workflow
- dashboard and analytics
- web application UI
- backend API

## Features

- Role-based dashboards and navigation
- Gemini/OpenAI integration points for AI-assisted processing
- Authentication or account flow screens

## Tech Stack

- Vite
- React
- TypeScript
- Tailwind CSS
- Python

## Code Highlights

- Entry points: index.html, backend/main.py, may/pulseIQ/main.py, src/App.tsx, src/main.tsx
- Routes/pages: src/pages/EmployeeDashboard.tsx, src/pages/LoginScreen.tsx, src/pages/ManagerDashboard.tsx
- Python dependencies are declared in the repository.
- JavaScript tooling and scripts are declared in package.json.

## Project Structure

- `index.html`
- `package.json`
- `tailwind.config.js`
- `tsconfig.json`
- `vite.config.ts`
- `backend/aggregate_features.py`
- `backend/anomaly_detection.py`
- `backend/api.py`
- `backend/compute_baselines.py`
- `backend/compute_burnout.py`
- `backend/deep_learning_model.py`
- `backend/generate_data.py`
- `backend/generate_report.py`
- `backend/main.py`
- `backend/manager_insights.py`
- `backend/parallel_roberta.py`
- `backend/predictive_ensemble.py`
- `backend/recommendations.py`

## Getting Started

Clone the repository and install the dependencies for the part of the project you want to run.

### Frontend / Node

```bash
npm install
npm run dev
```

### Available Scripts

- `dev`: `vite`
- `build`: `tsc && vite build`
- `lint`: `eslint . --ext ts,tsx --report-unused-disable-directives --max-warnings 0`
- `preview`: `vite preview`

### Python

```bash
cd backend
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
python main.py
```

## Python Dependencies

`backend/requirements.txt` includes: `fastapi`, `uvicorn`, `python-multipart`, `pandas`, `numpy`, `transformers          # HuggingFace RoBERTa sentiment model`, `torch                 # PyTorch backend for transformers`, `scikit-learn          # Isolation Forest, preprocessing, metrics`, `xgboost               # XGBoost classifier (burnout model)`, `shap                  # SHAP feature importance explanations`, `statsmodels           # ARIMA, seasonal decomposition, statistical tests`, `matplotlib`

## Development Notes

- Keep generated files, dependency folders, virtual environments, and build outputs out of commits.
- Add screenshots or deployment links here when the project is running in production.
- Update this README when entry points, environment variables, or setup steps change.
