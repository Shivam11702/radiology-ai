# Radiology AI Assistant — System Architecture

This document describes the high-level system architecture, service interactions, AI pipeline workflows, data models, and clinical governance mechanisms of the Radiology AI platform.

---

## 1. High-Level Architecture Diagram

```mermaid
flowchart TD
    subgraph Client["Client Layer"]
        UI["React 18 SPA (Vite + Tailwind CSS)"]
        ChatUI["Clinical Chat Assistant Widget (Gemini)"]
        Viewer["DICOM / Image Viewer & Heatmap Overlay"]
    end

    subgraph Gateway["API Gateway (FastAPI)"]
        Auth["JWT Auth & RBAC (Doctor, Radiologist, Admin)"]
        UploadHandler["Scan Ingestion (PNG, JPG, DICOM via pydicom)"]
        StudyService["Study & Patient Service"]
        AuditLog["Security & Compliance Audit Logger"]
        AssistantService["Gemini Multimodal Chat Service"]
    end

    subgraph Inference["AI & Explainability Engine"]
        Router["Modality Router (modality_router.py)"]
        
        subgraph Models["Model Zoo"]
            XR["Chest X-Ray (Production MVP)<br/>DenseNet121 (TorchXRayVision)<br/>18 Pathologies + Airspace Calibration"]
            CT["Chest CT (Research Baseline)<br/>DenseNet121 + Temperature Scaling"]
            MRI["Brain MRI (Research Baseline)<br/>DenseNet121 + Temperature Scaling"]
            US["Breast Ultrasound (Research Baseline)<br/>DenseNet121 + Square Padding"]
        end

        GradCAM["Explainability: Captum LayerGradCam"]
        MetricsEngine["Benchmark Metrics Engine (weights/metrics/)"]
        ReportGen["Report Engine (ReportLab PDF Compiler)"]
    end

    subgraph Storage["Data & Storage Layer"]
        DB[("SQLite / PostgreSQL Database")]
        FileStore["Local File Storage (/data/uploads, /reports, /heatmaps)"]
        PACS["Orthanc PACS (DICOMweb / Docker Profile)"]
    end

    UI --> Gateway
    ChatUI --> AssistantService
    AssistantService -->|Context & Reasoning| Gateway
    Gateway --> Router
    Router --> XR
    Router --> CT
    Router --> MRI
    Router --> US
    Models --> GradCAM
    Models --> ReportGen
    MetricsEngine --> ReportGen
    Gateway --> DB
    Gateway --> FileStore
    Gateway -.-> PACS
```

---

## 2. Multi-Modal Pipeline Execution Flow

When a clinician requests analysis (`POST /api/v1/predict/{study_id}`):

1. **Scan Retrieval & Validation**:
   - The file is retrieved from disk. DICOM files are parsed with `pydicom`, normalized, windowed, and converted to high-fidelity visual representations.
2. **Modality Routing**:
   - Based on user selection, study metadata, or DICOM modality headers, the request is dispatched via `AI/modality_router.py`.
3. **Model Inference**:
   - **X-Ray (XR)**: Run through TorchXRayVision DenseNet121. Calibrated operating thresholds evaluate 18 thoracic pathologies. Specific airspace consolidation logic evaluates Pneumonia with severity scoring (*Mild* / *Moderate* / *Severe*), confidence estimation, and prediction margin.
   - **CT / MRI / Ultrasound (Research Baselines)**: Evaluated through modality-specific DenseNet121 classifiers. The baseline feature flag (`ENABLE_BASELINE_MODELS=1`) is checked; if disabled or weights are missing, the pipeline safely **fails closed** with an HTTP 503 response. Softmax probabilities are adjusted using dataset-fitted temperature parameters.
4. **Visual Explainability (Grad-CAM)**:
   - For all modalities, Captum `LayerGradCam` extracts feature maps from the final dense block. A false-color Jet colormap is blended with the original grayscale image to produce an explainable heatmap overlay.
5. **Clinical Report Generation**:
   - `AI/report_generator.py` compiles dynamic findings and impression statements. For research baselines, hold-out benchmark metrics (accuracy, 95% bootstrap confidence intervals, and macro AUC) are injected directly into the findings text alongside clinical disclaimers.
   - ReportLab compiles a publication-grade PDF stored in `data/reports/report_{study_id}.pdf`.
6. **Persistence & Response**:
   - Predictions, confidence metrics, differential diagnoses, and file paths are saved in the database and returned to the frontend.

---

## 3. Database Schema

```mermaid
erDiagram
    users ||--o{ audit_logs : generates
    patients ||--o{ studies : has
    studies ||--o{ predictions : receives
    studies ||--o{ reports : generates

    users {
        int id PK
        string email UK
        string hashed_password
        string full_name
        string role
        boolean is_active
        datetime created_at
    }

    patients {
        int id PK
        string mrn UK
        string full_name
        int age
        string gender
        datetime created_at
    }

    studies {
        int id PK
        int patient_id FK
        string modality
        string file_path
        string original_filename
        string status
        datetime created_at
    }

    predictions {
        int id PK
        int study_id FK
        string primary_label
        float confidence
        string severity
        string uncertainty_level
        json differential_diagnosis
        json all_findings
        string heatmap_path
        datetime created_at
    }

    reports {
        int id PK
        int study_id FK
        text findings
        text impression
        string pdf_path
        datetime generated_at
    }

    audit_logs {
        int id PK
        int user_id FK
        string action
        string details
        string ip_address
        datetime timestamp
    }
```

---

## 4. Multimodal Clinical Chat Assistant Architecture

The clinical chat assistant (`POST /api/v1/assistant/chat`) provides on-demand radiologic reasoning:

```mermaid
sequenceDiagram
    autonumber
    actor Doctor as Clinician / Radiologist
    participant UI as Chat Widget (React)
    participant API as Backend Service (FastAPI)
    participant DB as Study Database
    participant Gemini as Google Gemini API

    Doctor->>UI: "What radiographic findings support pneumonia here?"
    UI->>API: POST /api/v1/assistant/chat {study_id, message, history}
    API->>DB: Fetch study findings, primary diagnosis, confidence, and metrics
    API->>Gemini: Prompt with Clinical System Persona + Study Context + Image Telemetry
    Gemini-->>API: Clinical Explanation & Guidelines Reference
    API-->>UI: Return formatted markdown response
    UI-->>Doctor: Display streaming response in dashboard
```

---

## 5. Security & Governance

- **Fail-Closed Baseline Gate**: Non-validated research baseline models cannot run unless explicitly enabled in environment variables (`ENABLE_BASELINE_MODELS=1`) and verified on disk.
- **Audit Logging**: Every patient creation, image upload, analysis execution, and report download is timestamped and recorded in `audit_logs`.
- **Role-Based Access Control (RBAC)**: Role checks verify user permissions across `doctor`, `radiologist`, and `admin` scopes.
- **Data Protection**: Medical scans and generated reports are stored on controlled filesystem paths with restricted direct access.
