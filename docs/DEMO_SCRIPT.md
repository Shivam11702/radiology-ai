# PRESENTATION & DEMO SCRIPT — Multi-Modal Radiology AI (5 Minutes)

Use this script during live demonstrations, faculty reviews, or hackathon presentations.

---

## Pre-Demo Checklist
1. Verify backend and frontend are running:
   - Backend: `http://localhost:8000/docs`
   - Frontend: `http://localhost:5173`
2. Ensure `.env` has:
   - `ENABLE_BASELINE_MODELS=1`
   - `GEMINI_API_KEY=...`
3. Baseline weights exist in `weights/` (run `scripts/setup_baseline_weights.py` if not already initialized).
4. Login credentials: `doctor@radiology.ai` / `demo1234`.

---

## MINUTE 1 — The Clinical Problem & Our Solution

### 0:00 – 0:30 | Problem
> "Healthcare systems face severe radiologist shortages — in India, there is approximately 1 radiologist per 100,000 citizens. In emergency departments and rural clinics, patients wait hours or days for critical imaging interpretation, while diagnostic delays in acute respiratory disease and oncologic lesions cost lives.
> 
> Furthermore, most existing medical AI tools are opaque black boxes that offer no visual proof and fail to fit into actual hospital workflows."

### 0:30 – 1:00 | Solution
> "We built **Radiology AI Assistant** — an explainable, multi-modal clinical decision support platform.
> It doesn't replace doctors — it empowers them:
> 1. Multi-modal triage across **Chest X-Ray**, **Brain MRI**, **Chest CT**, and **Breast Ultrasound**.
> 2. Visual evidence via **Grad-CAM heatmaps** showing exactly where the algorithm identified pathology.
> 3. Automatic clinical report compilation with **PDF export**.
> 4. An integrated **Multimodal Clinical Chat Assistant** powered by Gemini for instant clinical reasoning."

---

## MINUTES 2 to 3.5 — LIVE DEMO WALKTHROUGH

### Step 1: Clinical Dashboard (15 sec)
- Point to dashboard analytics: total scans, pending reviews, modality breakdown chart, and recent study timeline.
- *"A modern, clinical-grade interface designed around the radiologist's daily workflow."*

### Step 2: Chest X-Ray Analysis & Explainability (60 sec)
1. Click **Analyze** in navigation.
2. Click **"Pneumonia Demo"** button (preloads patient metadata and scan).
3. Click **"Run AI Analysis"**.
4. Highlight key outputs:
   - **Primary Diagnosis**: *"Pneumonia detected with 89.7% confidence (Severe category)."*
   - **Grad-CAM Heatmap**: *"Look at the red activation zone over the right lower lobe — visual proof of airspace opacity, not a blind guess."*
   - **Differential Diagnosis**: Ranked probabilities with uncertainty margin.
   - **Clinical Report**: Automated findings and impression.
5. Click **"Download PDF"** to display the compiled ReportLab PDF.

### Step 3: Multi-Modal Expansion & Benchmark Transparency (30 sec)
1. Switch to **Brain MRI** or **Chest CT** sample:
   - Load `mri_tumor_demo.jpg` or `ct_nodule_demo.jpg`.
2. Run analysis:
   - Point out the **Benchmark Metrics Card**:
     *"For non-X-ray modalities, our system functions as an honest research baseline. It displays hold-out dataset accuracy (89.4% on Brain MRI with 95% confidence intervals and 0.975 Macro AUC) and prominently displays safety disclaimers."*

### Step 4: Multimodal Clinical Assistant (30 sec)
1. Open the **Clinical Assistant** side panel.
2. Ask: *"What radiographic findings support this diagnosis and what follow-up is indicated?"*
3. Show the instant, structured clinical reasoning response generated via Gemini API referencing imaging guidelines.

---

## MINUTE 4 — Technical Architecture & Rigor

> "Under the hood:
> - **Core Computer Vision**: Pretrained DenseNet121 via TorchXRayVision evaluated on 18 thoracic conditions with calibrated decision thresholds.
> - **Explainability**: Captum LayerGradCam extracting gradient-weighted feature maps.
> - **Evaluation Rigor**: Temperature-scaled logits, Brier score calibration, and bootstrap confidence intervals stored in `weights/metrics/`.
> - **Safety First**: Fail-closed architecture — baseline models require explicit activation (`ENABLE_BASELINE_MODELS=1`), and low-confidence predictions trigger automated 'Needs Human Review' flags.
> - **PACS & Integration Ready**: Full DICOM upload via `pydicom`, JWT role-based access, and Orthanc PACS compatibility."

---

## MINUTE 5 — Clinical Impact & Close

> "This platform compresses turnaround times from 30 minutes to under 5 seconds, provides explainable visual heatmaps doctors can trust, and bridges the gap between rural clinics and specialist care.
> 
> The AI assists; the human expert decides.
> 
> Thank you, and we welcome your questions."

---

## JUDGE Q&A — QUICK REFERENCE

**Q: How accurate is your model?**
> "Our production X-ray model achieves an AUC of 0.940 for Lung Opacity. Our Brain MRI baseline achieves 89.4% hold-out accuracy with 95% bootstrap confidence intervals [87.9% – 90.9%] and 0.975 Macro AUC. We also provide temperature scaling and Brier score calibration."

**Q: Can this replace radiologists?**
> "No, by design. This is clinical decision support. Every report includes mandatory disclaimers requiring licensed radiologist review. The system is designed to triage and accelerate, not act autonomously."

**Q: Why do baseline models have disclaimers?**
> "Ethical AI requires transparency. Our CT, MRI, and US models are trained on public academic datasets. Rather than pretending they are clinical grade, we fail closed by default, expose empirical benchmark metrics, and clearly mark them as research baselines."

**Q: How do you handle DICOM?**
> "Native `pydicom` parsing with Hounsfield windowing, pixel spacing extraction, and Docker profiles for Orthanc PACS integration."
