# Implementation Roadmap & Milestone Progress

## Phase 1 — Foundation & Backend Services
- [x] Repository architecture & Git version control
- [x] FastAPI REST API with SQLAlchemy & SQLite/PostgreSQL
- [x] React 18 frontend with Tailwind CSS
- [x] Multi-format scan ingestion (PNG, JPG, DICOM via `pydicom`)
- [x] Patient management and study registry
- [x] JWT authentication and role-based access control (Doctor, Radiologist, Admin)
- [x] Security and compliance audit logging

## Phase 2 — Production Chest X-Ray AI Engine
- [x] TorchXRayVision DenseNet121 integration (trained on NIH, CheXpert, MIMIC)
- [x] 18-pathology thoracic classification with calibrated decision margins
- [x] Airspace opacity and consolidation detection for accurate pneumonia diagnosis
- [x] Platt-scaled probability calibration and severity classification (*Mild* / *Moderate* / *Severe*)
- [x] Automated "Needs Human Review" uncertainty flagging

## Phase 3 — Explainability & Clinical Reporting
- [x] Captum `LayerGradCam` explainability hooks
- [x] High-resolution false-color heatmap overlays on anatomical scans
- [x] Structured dynamic findings and impression generation
- [x] ReportLab publication-grade PDF report compiler with direct download
- [x] Differential diagnosis ranking with relative probability distribution

## Phase 4 — Multi-Modal Expansion (Research Baselines)
- [x] Modality Router (`AI/modality_router.py`) supporting XR, CT, MRI, and Ultrasound
- [x] DenseNet121 Brain MRI classifier (Glioma, Meningioma, Pituitary, No tumor)
- [x] DenseNet121 Breast Ultrasound classifier with aspect-ratio square padding
- [x] DenseNet121 Chest CT classifier with multi-histology categorization
- [x] Fail-closed governance gate (`ENABLE_BASELINE_MODELS=1`)
- [x] Checkpoint initialization script (`scripts/setup_baseline_weights.py`)

## Phase 5 — Metrics, Calibration & Conversational AI
- [x] Hold-out benchmark evaluation suite with 95% bootstrap confidence intervals (`weights/metrics/`)
- [x] Post-hoc temperature scaling and Brier score calibration
- [x] Dynamic injection of benchmark accuracy, CIs, and AUCs into baseline reports
- [x] Interactive Multimodal Clinical Chat Assistant powered by Google Gemini API
- [x] Comprehensive automated test suite (19 of 19 tests passing via `pytest`)

## Phase 6 — PACS & Future Extensions
- [x] Native DICOM parsing with anatomical windowing (Hounsfield units)
- [x] Orthanc PACS Docker Compose integration profile
- [ ] Interactive Cornerstone.js / OHIF viewer integration
- [ ] Bi-directional HL7 / FHIR messaging adapter
- [ ] Celery + Redis distributed task queue for high-volume hospital batch processing
