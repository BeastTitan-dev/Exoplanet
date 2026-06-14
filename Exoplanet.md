# ExoDetect AI — Exoplanet Transit Detection

AI-based data analysis pipeline that automatically detects exoplanet transit signals from noisy astronomical light curve data.

## Problem

Exoplanet detection through transit photometry requires identifying extremely small brightness variations in stars. In crowded fields, light curves suffer from:

- **Stellar blending** — foreground/background sources in the aperture
- **Detector noise** — intrinsic instrument response variability
- **Confounding phenomena** — eclipsing binaries, starspots, and intrinsic variability produce distinct but overlapping morphologies

ExoDetect AI detrends, searches for periodic dips, estimates orbital periods, and classifies the most likely astrophysical phenomenon with a transparent reasoning trace.

## Features

- CSV light curve upload (`time`, `flux` columns)
- One-click synthetic demo samples (exoplanet, binary, starspot, noise)
- Rolling-median detrending for crowded-field contamination
- Autocorrelation-based period estimation
- Dip detection with SNR thresholds
- Phenomenon classification: exoplanet transit, stellar eclipse, starspot, noise
- Phase-folded visualization
- Explainable AI reasoning trace
- Dark-themed React dashboard

## Tech Stack

| Layer | Stack |
|-------|-------|
| Frontend | React 18, Vite, Tailwind CSS |
| Backend | FastAPI, NumPy |
| Storage | JSON session store |
| Deploy | Docker, Docker Compose, Nginx |

## Architecture

```
frontend/          React dashboard (upload, charts, results)
backend/app/
  main.py          API routes
  services/
    lightcurve_service.py   CSV parsing + synthetic samples
    detection_service.py    Detrend → period search → classify
    storage_service.py      Session persistence
```

## API Endpoints

| Method | Path | Description |
|--------|------|-------------|
| GET | `/health` | Health check |
| GET | `/samples` | List demo sample types |
| GET | `/sample/{type}` | Raw sample data |
| POST | `/upload-lightcurve` | Upload CSV light curve |
| POST | `/load-sample` | Load synthetic sample into session |
| POST | `/analyze` | Run detection pipeline |

### Analyze Response

```json
{
  "transit_detected": true,
  "phenomenon": "exoplanet_transit",
  "confidence": 0.87,
  "period_days": 5.2,
  "noise_level": 0.0041,
  "events": [],
  "metrics": {},
  "lightcurve": { "time": [], "flux": [], "detrended": [] },
  "phase_fold": { "phase": [], "flux": [] },
  "reasoning": []
}
```

## Local Development

### Backend

```bash
cd backend
python -m venv .venv
.venv\Scripts\Activate.ps1          # Windows
pip install -r requirements.txt
uvicorn app.main:app --reload --port 8000
```

### Frontend

```bash
cd frontend
npm install
npm run dev
```

Open **http://localhost:5173**

Optional env: `VITE_API_BASE=http://localhost:8000`

## Docker Deployment

### Full stack

```bash
docker compose up --build
```

- Frontend: **http://localhost:5173**
- API docs: **http://localhost:8000/docs**

### Backend only

```bash
docker build -t exodetect-api .
docker run -p 8000:8000 exodetect-api
```

## CSV Format

```csv
time,flux
0.0,1.0002
0.05,0.9987
0.10,1.0011
```

- **time** — observation time in days (aliases: `t`, `bjd`, `mjd`, `phase`)
- **flux** — normalized brightness (aliases: `brightness`, `magnitude`)

Minimum 10 valid data points required.

## Demo Flow

1. Open the app → click **Exoplanet Transit** sample
2. Review classification, confidence, and orbital period
3. Inspect raw, detrended, and phase-folded light curves
4. Read the AI reasoning trace
5. Try **Eclipsing Binary** or **Noise Only** to compare classifications

## Production Notes

- Frontend is built with Vite and served via Nginx in Docker
- Set `VITE_API_BASE` at build time to your API URL
- Backend is stateless except for `backend/data/sessions.json`
- No GPU or external API keys required — fully deterministic pipeline
