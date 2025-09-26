### UEBA for Fraud Detection

A Flask-based User and Entity Behavior Analytics (UEBA) system that ingests Zeek network telemetry, applies supervised and unsupervised detection (phishing, DGA, malicious URL/file downloads, anomalies, exfiltration), and aggregates results into a unified risk score per user/entity.


## Overview
- **Data source**: Zeek (Bro) logs; optional Elasticsearch/MySQL enrichments
- **Backend**: Flask APIs for retrieval, scoring, orchestration
- **Analytics**
  - Supervised: Phishing, DGA, malicious URL, malicious file download
  - Unsupervised: K-Means, Isolation Forest
  - Exfiltration: PCR (Producer–Consumer Ratio)
- **Risk scoring**: Linear weighted aggregation across modules
- **mBAT**: Train new models and build baseline user profiles

## Repository Structure
- `run.py`: Starts the Flask API server
- `riskscoring.py`: Scheduler-driven risk scoring loop
- `scoring/`, `scoring_parser.py`: Scoring utilities and parsing
- Modules: `Anomaly_detection/`, `exfil/`, `dga/`, `safe_browsing/`, `uri_checker/`, `User_agent/`, `StreamingPhish/`
- Aux: `mBAT/` (training/baselines), `win_activity/`, `freq_master/`, `app_checker/`
- `config/`: Configuration files and thresholds
- `db/`: Database-related assets
- `notebooks/`: Jupyter notebooks
- `requirements.txt`: Pinned dependencies
- `test_endpoints.py`: Example endpoint interactions

## Prerequisites
- Python 3.7–3.8 recommended (aligns with TensorFlow 2.2.0 and pinned deps)
- Zeek data available locally or via Elasticsearch
- Optional: MySQL if using DB features

## Installation
```bash
# Windows PowerShell (recommended)
python -m venv .venv
.\.venv\Scripts\Activate.ps1

python -m pip install --upgrade pip
pip install -r requirements.txt
```

## Configuration
- Update `config/` with:
  - Zeek log paths and/or Elasticsearch connection settings
  - Optional API keys (e.g., Safe Browsing) and detection thresholds
  - Database credentials in `db/` if using MySQL
- Ensure versions align with `requirements.txt` (e.g., `elasticsearch==7.10.1`)

## Data Setup (Required)
- **Unzip `conanomaly_input` before running locally**
  - The anomaly dataset is zipped and must be extracted.
  - Example (Windows PowerShell):
    ```bash
    Expand-Archive -Path .\Anomaly_detection\conanomaly_input.zip -DestinationPath .\Anomaly_detection\conanomaly_input -Force
    ```
  - Verify that `Anomaly_detection\conanomaly_input\` exists and contains the expected files.

## Usage

### Start the API
```bash
python run.py
```
- Adjust host/port in `run.py` or environment variables as needed.
- Once running, access the server via browser or `curl`.

### Run the Risk-Scoring Scheduler
```bash
python riskscoring.py
```
- Uses APScheduler’s BlockingScheduler to periodically aggregate module outputs and compute risk scores.

### Example Interactions
- Health/status (example; adjust host/port):
```bash
curl -s http://127.0.0.1:5000/
```
- Explore endpoints and payloads:
  - Review `test_endpoints.py` for sample calls
  - Inspect `run.py` and module blueprints for routes

## Key Dependencies (partial)
- API/runtime: `Flask==1.1.1`, `waitress==1.4.4`
- Data/ML: `pandas==1.2.1`, `numpy==1.19.5`, `scikit_learn==0.24.2`, `tensorflow==2.2.0`, `joblib==0.15.1`, `pyod==0.8.8`
- Ingestion/enrichment: `elasticsearch==7.10.1`, `zat==0.4.3`, `pysafebrowsing==0.1.1`, `tld==0.9.6`, `tldextract==2.2.2`, `user_agents==2.2.0`, `crawlerdetect==0.1.4`
- Scheduling: `APScheduler==3.7.0`
- DB: `mysql_connector_repackaged==0.3.1`
- Full list: see `requirements.txt`

## Notes
- Package versions are tightly pinned for reproducibility; prefer Python 3.7–3.8.
- Ensure Zeek data paths and Elasticsearch are reachable and version-compatible.
