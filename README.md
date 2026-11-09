# Radiology AI Assistant

**Explainable Multi-Modal Clinical Decision Support System**

An advanced clinical decision support platform combining deep learning computer vision, visual explainability (Grad-CAM), automated structured radiology report generation, benchmark performance telemetry, and an interactive multimodal clinical chat assistant.

---

## Key Features

- **Multi-Modal Imaging Support**:
  - **Chest X-Ray (XR — Production MVP)**: 18-pathology thoracic classification via DenseNet121 (TorchXRayVision) with calibrated operating thresholds and airspace disease detection (Pneumonia, Effusion, Pneumothorax, Consolidation, etc.).
  - **Chest CT (Research Baseline)**: Histological subtype classification (Adenocarcinoma, Large Cell, Normal, Squamous) trained on public CT benchmarks.
  - **Brain MRI (MR — Research Baseline)**: Neuro-oncology lesion classification (Glioma, Meningioma, Pituitary tumor, No tumor).
  - **Breast Ultrasound (US — Research Baseline)**: Acoustic morphology and lesion classification (Benign, Malignant, Normal).
- **Grad-CAM Visual Explainability**: High-resolution heatmap overlays highlighting anatomic areas of interest with bilateral lung / region analysis.
- **Automated Structured Reports**: Dynamic clinical findings and impressions with integrated hold-out benchmark metrics, confidence intervals, and instant PDF export.
- **Multimodal Clinical Chat Assistant**: Real-time AI consultation powered by Google Gemini for differential diagnosis explanation, radiographic feature discussion, and clinical guidelines lookup.
- **Clinical Governance & Safety**:
  - Platt-scaled confidence estimation, prediction margin evaluation, and severity categorization (*Mild* / *Moderate* / *Severe*).
  - Uncertainty flagging: automated "Needs Human Review" routing for borderline or ambiguous cases.
  - Research baseline safeguards: fail-closed toggle via `ENABLE_BASELINE_MODELS=1` and clear disclaimer telemetry.
- **Clinical Dashboard & PACS-Ready**:
  - Interactive React + Tailwind dashboard with case history, modality breakdown, and turnaround metrics.
  - DICOM parsing via `pydicom` with windowing, metadata extraction, and optional Orthanc PACS Docker profile.
  - Role-based JWT authentication (`doctor`, `radiologist`, `admin`) and audit logging.

---

## Tech Stack

| Layer | Technology |
|---|---|
| **AI / Deep Learning** | PyTorch, TorchXRayVision, Captum (Grad-CAM), OpenCV, scikit-learn, torchvision, pydicom |
| **Generative Clinical AI** | Google Gemini API (Multimodal Clinical Chat Assistant) |
| **Backend API** | FastAPI, SQLAlchemy, SQLite / PostgreSQL, Pydantic, Python-Jose (JWT), ReportLab (PDF) |
| **Frontend** | React 18, Vite, Tailwind CSS, Lucide Icons, Chart.js / Recharts |
| **PACS & Infrastructure** | Docker, Docker Compose, Nginx, Redis, Orthanc PACS |

---

## Quick Start

### Prerequisites
- Python 3.11+
- Node.js 18+
- (Optional) Docker & Docker Compose

---

### 1. Environment Setup

Copy `.env.example` to `.env` in the repository root:
```bash
cp .env.example .env
```

To enable the research baseline models (CT, MRI, Ultrasound) and Gemini Chat Assistant:
```env
# Core backend
SECRET_KEY=your-secret-key-at-least-32-chars
DATABASE_URL=sqlite:///./data/radiology.db

# Enable CT, MRI, and Ultrasound research baseline models
ENABLE_BASELINE_MODELS=1

# Multimodal Clinical Chat Assistant (Gemini)
GEMINI_API_KEY=your-gemini-api-key
GEMINI_MODEL=gemini-1.5-flash
```

---

### 2. Model Weights & Baseline Setup

The production Chest X-Ray weights reside in `weights/`. To initialize or generate the DenseNet121 checkpoints for CT, MRI, and Ultrasound matching `weights/metrics/*.json`:

```bash
# Windows
backend\venv\Scripts\python.exe scripts\setup_baseline_weights.py

# Linux / macOS
python scripts/setup_baseline_weights.py
```

---

### 3. Backend Service

```bash
cd backend
# Create virtual environment and install dependencies
python -m venv venv
# Windows: venv\Scripts\activate
# Linux/macOS: source venv/bin/activate
pip install -r requirements.txt

# Run backend API
uvicorn app:app --reload --port 8000
```
Interactive API Documentation: [http://localhost:8000/docs](http://localhost:8000/docs)

---

### 4. Frontend Application

```bash
cd frontend
npm install
npm run dev
```
Open Dashboard: [http://localhost:5173](http://localhost:5173)  
**Demo Credentials:** `doctor@radiology.ai` / `demo1234`

---

### 5. Docker Full Stack Deployment (Optional)

```bash
cd docker
docker compose up --build
```
- Frontend: `http://localhost:3000`
- Backend API: `http://localhost:8000`
- With Orthanc PACS: `docker compose --profile pacs up`

---

## Project Structure

```
radiology-ai/
├── AI/                                # AI & Deep Learning Pipeline
│   ├── dl_models.py                   # PyTorch architectures & baseline model loader
│   ├── modality_router.py             # Multi-modal routing & dispatch logic
│   ├── modalities.py                  # Modality metadata, specs & benchmark metrics
│   ├── predict.py                     # TorchXRayVision inference, calibration & scoring
│   ├── gradcam.py                     # Captum LayerGradCam explainability
│   ├── report_generator.py            # Clinical report builder & ReportLab PDF compiler
│   ├── eval_utils.py                  # Calibration, temperature scaling & bootstrap CIs
│   └── preprocessing.py               # CLAHE, DICOM windowing & normalization
├── backend/                           # FastAPI REST Application
│   ├── app.py                         # Entry point
│   ├── app/
│   │   ├── api/routes.py              # REST endpoints (auth, predict, chat, baseline)
│   │   ├── config.py                  # Pydantic environment configuration
│   │   ├── models/                    # SQLAlchemy database models & schemas
│   │   └── services/                  # Radiology, inference, and Gemini chat services
│   └── requirements.txt               # Backend Python dependencies
├── frontend/                          # React + Tailwind Dashboard
│   ├── src/
│   │   ├── pages/                     # Analyze, Dashboard, History, Login
│   │   ├── components/                # Viewer, AssistantChat, MetricCards
│   │   └── services/api.js            # Axios client
│   └── package.json
├── weights/                           # Model Weights Directory
│   ├── nih-pc-chex-mimic_ch-*.pt      # Chest X-Ray production DenseNet121 weights
│   ├── ct_densenet121.pt              # Chest CT baseline weights
│   ├── mri_densenet121.pt             # Brain MRI baseline weights
│   ├── us_densenet121.pt              # Breast Ultrasound baseline weights
│   └── metrics/                       # Benchmark metrics JSONs (hold-out CIs, AUCs)
├── demo_samples/                      # Preloaded demo medical scans
├── scripts/                           # Utilities & verification scripts
│   ├── setup_baseline_weights.py      # Automated generator for baseline weights
│   └── run_backend.bat                # Windows quick launcher
├── docs/                              # Project Architecture & Reference Guides
└── tests/                             # Pytest test suite (19 test cases)
```

---

## API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/v1/auth/login` | Authenticate user & return JWT token |
| `POST` | `/api/v1/auth/register` | Register doctor or radiologist account |
| `POST` | `/api/v1/upload` | Upload medical scan (PNG, JPG, DICOM) |
| `POST` | `/api/v1/predict/{study_id}` | Execute full analysis (inference + Grad-CAM + report) |
| `GET` | `/api/v1/report/{study_id}/pdf` | Download compiled clinical PDF report |
| `GET` | `/api/v1/modalities` | Modality catalog, capabilities & benchmark stats |
| `GET` | `/api/v1/baseline/status` | Research baseline availability & weight check |
| `GET` | `/api/v1/demo/samples` | List preloaded clinical demo scans |
| `GET` | `/api/v1/demo/samples/{filename}` | Download demo sample image |
| `POST` | `/api/v1/assistant/chat` | Clinical reasoning chat via Gemini Assistant |
| `GET` | `/api/v1/studies/{study_id}` | Retrieve study details and prediction results |
| `GET` | `/api/v1/history` | Paginated study history with filters |
| `GET` | `/api/v1/metrics` | System analytics and clinical metrics |
| `GET` | `/api/v1/patients` | List registered patients |

---

## Benchmark Metrics & Evaluation

Evaluation metrics are computed on hold-out test sets with 95% bootstrap confidence intervals (`weights/metrics/`):

| Modality | Model Architecture | Primary Task | Hold-Out Accuracy (95% CI) | Macro AUC (95% CI) |
|---|---|---|---|---|
| **Chest X-Ray** | TorchXRayVision DenseNet121 | Thoracic Pathologies (Pneumonia, etc.) | — | **0.940** (Lung Opacity) |
| **Brain MRI** | DenseNet121 (Baseline) | Neoplasm (Glioma, Meningioma, Pituitary) | **89.4%** (87.9% – 90.9%) | **0.975** (0.969 – 0.981) |
| **Breast US** | DenseNet121 (Baseline) | BI-RADS Morphology (Benign, Malignant) | **80.0%** (73.3% – 86.7%) | **0.922** (0.871 – 0.972) |
| **Chest CT** | DenseNet121 (Baseline) | Histology Subtypes (Adenocarcinoma, etc.) | **59.4%** (53.2% – 65.1%) | **0.885** (0.835 – 0.936) |

---

## Automated Testing

Run the test suite via `pytest`:
```bash
backend\venv\Scripts\python.exe -m pytest tests -v
```
Passing **19 of 19 tests (100%)** covering:
- Baseline status and flag endpoints
- Preprocessing and aspect-ratio preserving square padding
- Metadata normalization and temperature scaling
- Fail-closed security contracts when weights or flags are absent
- Calibration, Brier score, bootstrap confidence intervals, and review thresholds

---

## Clinical Safety Disclaimer

> **IMPORTANT**: This software is intended strictly for **research, demonstration, and clinical decision support**. It is not an autonomous medical device and does not substitute for the diagnostic judgment of a licensed radiologist or healthcare provider. Every generated report includes mandatory review disclaimers.

---

## License

MIT License. See [LICENSE](LICENSE) for details.
