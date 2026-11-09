# Judge Q&A Cheat Sheet — Clinical & Technical Defense

---

## 1. Accuracy, Evaluation & Calibration

**Q: How accurate are your models?**
- **Chest X-Ray (Production MVP)**: TorchXRayVision DenseNet121 achieves an AUC of **0.940** on Lung Opacity. Operating thresholds are clinically calibrated across 18 thoracic pathologies.
- **Brain MRI (Research Baseline)**: Evaluated on 1,600 hold-out images — **89.4% accuracy** (95% bootstrap CI: 87.9% – 90.9%), **0.975 Macro AUC**, temperature parameter $T = 1.253$.
- **Breast Ultrasound (Research Baseline)**: Evaluated on 150 hold-out images — **80.0% accuracy** (95% bootstrap CI: 73.3% – 86.7%), **0.922 Macro AUC**, temperature parameter $T = 0.699$.
- **Chest CT (Research Baseline)**: Evaluated on hold-out test set — **59.4% accuracy** (95% CI: 53.2% – 65.1%), **67.7% balanced accuracy**, **0.885 Macro AUC**.

**Q: Did you train your own models or use pretrained ones?**
- For Chest X-Ray, we utilize pretrained TorchXRayVision models trained on multi-center clinical cohorts (NIH, CheXpert, MIMIC-CXR, PadChest, OpenI) with clinical operating threshold calibration and airspace opacity logic.
- For CT, MRI, and Ultrasound, DenseNet121 architectures were trained on domain-specific public benchmarks (Kaggle Brain Tumor, BUSI, CT Lung Cancer), calibrated with temperature scaling, and evaluated using bootstrap confidence intervals stored in `weights/metrics/`.

**Q: Why do you report confidence intervals instead of just single point metrics?**
- Medical machine learning demands statistical rigor. Point estimates can be deceptive on limited test sets. 95% bootstrap confidence intervals (1,000 resamples) quantify estimation uncertainty and demonstrate transparency.

---

## 2. Clinical Safety & Governance

**Q: Can this system replace doctors?**
- **No.** This is an assistive clinical decision support system (CDSS). Every generated report and dashboard screen explicitly specifies that final clinical interpretation must be performed by a qualified radiologist or physician.

**Q: How do you prevent hallucinated or overconfident predictions?**
- **Platt & Temperature Calibration**: Raw logits are scaled to reduce overconfidence.
- **Uncertainty Margin & Flagging**: When predictions fall close to decision boundaries or have low confidence margins, the system automatically flags them as **"Needs Human Review"**.
- **Grad-CAM Explainability**: Clinicians can cross-verify whether the model attended to actual anatomic lesions or imaging artifacts.

**Q: What happens if someone uploads the wrong scan type (e.g. an Ultrasound image to the MRI model)?**
- Our baseline benchmark suites explicitly evaluated out-of-distribution / wrong-modality inputs. Baseline classifiers without out-of-distribution filters can exhibit high confidence on out-of-domain images. We document this limitation transparently in `docs/baseline_models.md` and report headers, and we maintain an explicit modality router and validation barrier.

**Q: Why are CT, MRI, and US labeled as "Research Baselines"?**
- Genuine clinical deployment requires prospective multi-center validation. By clearly distinguishing between the production MVP (Chest X-Ray) and academic research baselines (CT, MRI, US), we prevent clinical misrepresentation while demonstrating a complete end-to-end multi-modal pipeline.

---

## 3. Technical Architecture & Innovation

**Q: What makes your platform stand out from simple classifier scripts?**
1. **Explainability**: Captum LayerGradCam heatmaps pinpointing pathology regions.
2. **Clinical Reporting**: Automated structured PDF generation with findings, impressions, and benchmark telemetry.
3. **Conversational AI**: An interactive Multimodal Clinical Chat Assistant powered by Google Gemini for differential diagnosis discussions and imaging guideline references.
4. **PACS Compatibility**: Support for DICOM parsing with `pydicom`, windowing, and an Orthanc PACS Docker profile.
5. **Fail-Closed Architecture**: Robust security controls via `ENABLE_BASELINE_MODELS=1` and JWT role-based access.

**Q: How is DICOM handled?**
- Scans are parsed with `pydicom`. Rescaling slope/intercept and modality LUTs apply anatomical Hounsfield windowing (e.g., lung window, mediastinal window) before converting pixel data into normalized tensor representations.

---

## 4. Key Metrics & Numbers at a Glance

- **1 : 100,000**: Approximate radiologist-to-population ratio in India.
- **30 min → 5 sec**: Turnaround reduction for initial triage and preliminary report drafting.
- **18**: Thoracic pathology classes evaluated in the Chest X-Ray pipeline.
- **4 Modalities**: Chest X-Ray, Brain MRI, Chest CT, Breast Ultrasound.
- **19 of 19**: Test cases passing (100%) in `pytest` test suite.
