# PulseIQ

PulseIQ is a full-stack intelligence platform built to detect, track, and mitigate employee burnout using behavioral data and a suite of machine learning models. The project analyzes synthetic workspace data—such as Slack messages, Jira activity, and Git commits—to identify early warning signs of burnout before they become critical. It provides two main views: a high-level manager dashboard for team-wide risk assessment and an employee portal showing personal focus and recovery metrics.

## Table of Contents
- [About the Project](#about-the-project)
- [Key Features](#key-features)
- [Tech Stack](#tech-stack)
- [System Architecture](#system-architecture)
- [Folder Structure](#folder-structure)
- [Important Code Concepts](#important-code-concepts)
- [Architectural Decisions](#architectural-decisions)
- [Data Model](#data-model)
- [Main User Flows](#main-user-flows)
- [Setup Instructions](#setup-instructions)
- [Available Scripts](#available-scripts)
- [Configuration Notes](#configuration-notes)
- [Future Improvements](#future-improvements)
- [Learning Outcomes](#learning-outcomes)

## About the Project

Traditional burnout tracking usually relies on trailing indicators like quarterly surveys or sudden drops in output. PulseIQ takes a more proactive approach. The current implementation processes simulated daily activity data across different tools (communication, code, meetings) to calculate an ongoing "Burnout Probability" alongside several sub-scores, such as the Deep Work Index, Fragmentation Score, and Recovery Debt.

The application is structured to serve two specific roles:
- **Managers** need a way to see which team members are at risk, why they are at risk, and what interventions might help (e.g., reallocating tasks or reducing Zoom load).
- **Employees** need visibility into their own working patterns so they can understand how context switching and after-hours work impact their baseline productivity.

At this stage, PulseIQ relies on synthetic data generation scripts to simulate a real engineering environment, running its models locally to output CSVs which the FastAPI backend then serves to the React frontend.

## Key Features

### Role-Based Dashboards
The React frontend splits routing based on the user's role. Managers see a team overview with risk tiering and historical trend charts. Employees see an individualized dashboard focusing on their specific daily metrics and behavioral sub-scores.

### Behavioral Burnout Engine
The core of the system calculates burnout risk based on behavioral sub-scores. For instance, the Fragmentation Score tracks context switching based on Jira and Slack activity, while Recovery Debt measures after-hours and weekend work.

### Machine Learning Suite
The backend coordinates a batch pipeline (`run_ml_parallel.py`) that includes:
- **Time-Series Analysis**: Uses EWMA (Exponentially Weighted Moving Average) and ARIMA forecasting to predict future burnout trends.
- **Anomaly Detection**: Uses Scikit-Learn's Isolation Forest to flag when an employee deviates significantly from their normal behavioral clusters.
- **Deep Learning Prediction**: Uses an LSTM network to identify nonlinear patterns over a trailing window of activity.
- **Predictive Ensemble**: Combines outputs from the behavioral engine, time-series forecasting, and LSTM into a single weighted risk score.

### AI-Driven Narratives and Recommendations
The backend integrates with the Gemini API to generate text summaries of employee states. It also automatically drafts supportive Slack/Email messages for managers to send when a specific intervention is recommended by the engine.

## Tech Stack

| Layer | Technology | Purpose |
| --- | --- | --- |
| Frontend | React / Vite / TypeScript | Renders the role-based dashboards and charts |
| Styling | Tailwind CSS | Provides utility classes for responsive component design |
| Backend API | FastAPI / Uvicorn | Serves local ML data to the frontend via REST endpoints |
| Data Processing | Pandas / NumPy | Aggregates daily features and computes baseline statistics |
| NLP Sentiment | HuggingFace RoBERTa / PyTorch | Analyzes sentiment in Slack messages |
| Machine Learning | Scikit-Learn / XGBoost | Powers anomaly detection and behavioral classification |
| Time-Series | statsmodels | Computes ARIMA and seasonal decomposition forecasting |
| LLM Integration | Google GenAI (Gemini) | Generates narrative summaries and drafts intervention messages |

## System Architecture

PulseIQ is orchestrated by a central launcher (`run.py`) that coordinates both the Python data pipeline and the web servers.

```txt
Python Data Generators (Slack, Jira, Git, etc.)
  ↓
Data Processing & Feature Engineering (Pandas)
  ↓
ML Pipeline (Isolation Forest, LSTM, ARIMA)
  ↓
CSV Data Artifacts (pulseiq_data/)
  ↓
FastAPI Backend (Reads CSVs, queries Gemini API)
  ↓
React Frontend (Fetches JSON endpoints, renders charts)
```

The data flow is currently file-based. The pipeline processes raw syntheic logs into daily features, computes metrics, and outputs a series of CSV files. The FastAPI application then reads these artifacts into memory to serve incoming requests from the Vite application.

## Folder Structure

```txt
backend/
  api.py                      FastAPI endpoints and LLM integration
  main.py                     Orchestrates the sequential ML pipeline steps
  generate_data.py            Creates synthetic raw workspace activity
  aggregate_features.py       Processes raw data into daily metrics
  compute_burnout.py          Calculates behavioral sub-scores
  time_series_analysis.py     Runs ARIMA forecasting
  anomaly_detection.py        Runs Scikit-Learn Isolation Forest
  deep_learning_model.py      Runs TensorFlow/PyTorch LSTM networks
  predictive_ensemble.py      Combines model outputs
src/
  pages/
    EmployeeDashboard.tsx     Individualized metric view
    ManagerDashboard.tsx      Team overview and intervention view
    LoginScreen.tsx           Basic role selection entry point
  api.ts                      Frontend fetch wrappers for backend endpoints
  types.ts                    TypeScript interfaces for the data model
  App.tsx                     Main routing shell checking user role
Docs/                         Detailed LaTeX and Markdown documentation of the ML steps
run.py                        Root launcher script for the pipeline and servers
```

## Important Code Concepts

### Pipeline Orchestration
The ML suite is complex and sequential. `backend/main.py` acts as a registry for the individual steps (generation, feature building, baseline computation, etc.), ensuring they run in the correct order while allowing specific phases to be skipped via command-line flags if the data is already generated.

### File-Based Persistence
Because PulseIQ focuses on complex ML transformations, it currently avoids a traditional relational database. Instead, the Python scripts output structured CSVs into a `pulseiq_data/` directory. The FastAPI backend functions as a read-only presentation layer over these files.

### Ask Pulse (RAG Simulation)
The backend features an `/api/llm/ask_pulse` endpoint. When a manager asks a question, the API gathers the current state from the local CSVs (employee stats, pending suggestions) and injects it into a prompt for the Gemini model, allowing the LLM to answer questions about the team using up-to-date context.

### Recharts Integration
The frontend relies heavily on Recharts to visualize the time-series data generated by the backend. The dashboards map the historical daily scores and forecasts into Area and Radar charts, allowing users to see the trailing trends visually.

## Architectural Decisions

### Python Backend / Node Frontend Split
The backend is written in Python to leverage Pandas, Scikit-Learn, and HuggingFace Transformers, which are the industry standards for this type of data processing and machine learning. The frontend uses React and Vite because they offer a faster, more type-safe development experience for building complex interactive dashboards compared to traditional templating engines.

### Centralized run.py Launcher
Managing a separate data pipeline, an API server, and a frontend dev server can be cumbersome. The `run.py` script simplifies this by using Python's `subprocess` module to spin up all necessary environments simultaneously. It monitors the child processes and ensures they all shut down cleanly when the user exits the script.

### Local CSV Artifacts Over a Database
Using CSV files as the handoff mechanism between the ML pipeline and the API is intentional. At this prototype stage, the primary challenge is fine-tuning the data engineering and model weights. Writing the intermediate states to disk makes it much easier to inspect the output of step 3 before running step 4, without needing to manage database migrations or schema changes.

## Data Model

The frontend expects several specific entities from the backend, defined in `src/types.ts`:

- **EmployeeStat**: The core summary for an individual. It tracks their current productivity, burnout percentage, status (`healthy`, `warning`, `critical`), and sub-scores like `fragmentationScore` and `recoveryDebt`. It also includes the ensemble risk tier.
- **Suggestion**: An actionable recommendation generated by the engine (e.g., "Schedule Focus Blocks"). It tracks the reason for the suggestion and the manager responsible for enacting it.
- **EmployeeHistory**: A time-series entry tracking an employee's index scores over a specific date, used for drawing the historical trend charts.
- **Anomaly**: Represents an event flagged by the Isolation Forest, including the specific `triggerFeature` and `isolationScore`.
- **Forecast**: Holds the ARIMA prediction metrics, including the 7-day and 14-day forecasted probability averages.
- **EnsembleSummary**: Provides a macro view of the entire organization's risk distribution across the different tiers.

## Main User Flows

### Manager Dashboard View
1. The user logs in and selects the "manager" role.
2. The React frontend mounts `ManagerDashboard.tsx` and fetches the team's `EmployeeStat` list and `EnsembleSummary` from the API.
3. The manager reviews the Risk Distribution and the specific employees flagged as "CRITICAL" or "HIGH" risk.
4. The manager views the `Suggestion` list for recommended interventions.
5. If the manager has a question, they use the "Ask Pulse" chat interface, which queries the LLM endpoint to provide context-aware answers based on the current data.

### Employee Personal View
1. The user logs in and selects an "employee" profile.
2. The app mounts `EmployeeDashboard.tsx`.
3. The dashboard fetches the employee's specific `EmployeeHistory`, `Forecast`, and `Anomalies`.
4. The employee reviews their personal Radar chart to see if they are over-indexing on fragmentation or recovery debt, allowing them to adjust their upcoming schedule.

## Setup Instructions

### Prerequisites
- Node.js (v16 or higher)
- Python 3.9+

### Installation
Clone the repository, then install the dependencies for both environments.

```bash
# Install frontend dependencies
npm install

# Install backend dependencies
pip install -r backend/requirements.txt
```

### Running Locally
You can run the entire pipeline, backend, and frontend with a single command.

```bash
python run.py
```

This will run the ML data generation, start the FastAPI server on port 8000, and start the Vite frontend on port 5173.

If you have already generated the ML data and just want to start the servers faster, you can skip the pipeline:
```bash
python run.py --skip-pipeline
```

## Available Scripts

| Command | Description |
| --- | --- |
| `npm run dev` | Starts the Vite development server independently |
| `npm run build` | Compiles TypeScript and builds the frontend for production |
| `npm run lint` | Runs ESLint over the frontend source code |
| `python run.py` | Runs the ML pipeline, backend API, and frontend server |
| `python backend/main.py` | Runs only the ML pipeline steps |

## Configuration Notes

- `vite.config.ts`: Configures the React application build and specifies the development server port.
- `tailwind.config.js`: Defines the utility classes and custom color palette used throughout the dashboards.
- `tsconfig.json` & `tsconfig.node.json`: Configures the TypeScript compiler for the React frontend, enforcing strict type checking.

## Future Improvements

- **Database Migration**: Transition the data persistence from local CSV artifacts to a proper time-series database (like InfluxDB) or PostgreSQL to support multi-tenant data over longer periods.
- **Live Integrations**: Replace the synthetic data generation scripts with actual OAuth integrations into Slack, Jira, and GitHub APIs to process real team data.
- **Authentication**: Implement real user authentication (e.g., JWTs or Supabase Auth) instead of the current simple dropdown role selector.
- **Unit Testing**: Add a test suite (pytest for the backend, Vitest for the frontend) to ensure the data transformations and UI components remain stable as the models evolve.
- **Model Tuning**: Further tune the XGBoost and LSTM hyperparameters based on labeled real-world datasets rather than the current synthetic baselines.

## Learning Outcomes

This project demonstrates a solid grasp of how to bridge complex data science with modern web development. It shows the ability to orchestrate a multi-step machine learning pipeline involving classical statistics, anomaly detection, and deep learning, while also exposing that data through a clean, role-based REST API. On the frontend, it highlights how to translate dense numerical output into actionable, highly visual React dashboards that cater to different user domains. It reflects strong product thinking by focusing not just on identifying a problem (burnout), but on providing the user with AI-assisted workflows to actually solve it.
