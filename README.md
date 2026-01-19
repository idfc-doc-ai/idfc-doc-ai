# IDFC-DOC-AI

**Intelligent Document AI for Invoice & Quotation Field Extraction**

> End-to-end, production-oriented Document AI system for extracting structured fields from multilingual, semi-structured invoices under real-world constraints.

---

## 🎯 Problem Scope

Financial documents (tractor loan quotations in this challenge) are:

* Layout-diverse
* Multilingual (English + Indic languages)
* Low quality (scans, photos, handwritten elements)

Manual extraction is slow and error-prone.
This system automates **field-level extraction + validation** with **high document-level accuracy**, low latency, and explainability.

---

## 📌 Fields Extracted

* Dealer Name (fuzzy match)
* Model Name (exact match)
* Horse Power (numeric)
* Asset Cost (numeric)
* Dealer Signature (presence + bounding box)
* Dealer Stamp (presence + bounding box)

---

## 🧠 Design Philosophy (Read This Carefully)

This is **not** a prompt-based VLM demo.

Key principles:

* Prefer **deterministic + explainable pipelines** over black-box LLM calls
* OCR + layout first, semantics second
* Vision models used **only where text fails** (stamp/signature)
* CPU-first design → predictable cost and latency
* Invoice-agnostic (no template hardcoding)

---

## 🏗️ Project Structure (What Each Folder Actually Does)

```
IDFC-DOC-AI/
│
├── data/           # Raw PDFs / images and derived artifacts
│
├── ocr/            # OCR engines, preprocessing, text + bbox extraction
│
├── extraction/     # Field-specific extraction logic (rules + heuristics)
│
├── detection/      # Vision-based detection (signature / stamp)
│
├── evaluation/     # Accuracy, IoU, and document-level metrics
│
├── utils/          # Shared helpers (normalization, matching, configs)
│
├── main.py         # Single entry point for end-to-end inference
├── requirements.txt
├── .gitignore
└── venv/           # Local virtual environment (not used in evaluation)
```

If a folder exists, it has a **single responsibility**.
No God-files. No spaghetti imports.

---

## 🔄 End-to-End Pipeline

```
PDF / Image
   ↓
OCR (text + bounding boxes)
   ↓
Layout grouping & cleanup
   ↓
Field candidate extraction
   ↓
Semantic validation & matching
   ↓
Signature & stamp detection
   ↓
Confidence aggregation
   ↓
Structured JSON output
```

---

## 🔍 Module Responsibilities

### `ocr/`

* Image preprocessing (resize, denoise, binarize)
* Multilingual OCR
* Outputs word-level text with bounding boxes

### `extraction/`

* Dealer name: keyword proximity + fuzzy match
* Model name: exact lookup against master list
* HP & Cost: regex + contextual filtering
* Rejects footer totals, headers, and noise

### `detection/`

* Lightweight vision models for:

  * Signature presence
  * Stamp presence
* Outputs bounding boxes
* Evaluated using IoU ≥ 0.5

### `evaluation/`

* Field-level correctness
* Document-Level Accuracy (DLA)
* IoU computation for visual fields

---

## ▶️ How to Run

```bash
pip install -r requirements.txt
python main.py --input data/ --output outputs/
```

`main.py` orchestrates the **entire pipeline**.
No hidden notebooks. No manual steps.

---

## 📤 Output Format

```json
{
  "doc_id": "invoice_001",
  "fields": {
    "dealer_name": "ABC Tractors Pvt Ltd",
    "model_name": "Mahindra 575 DI",
    "horse_power": 50,
    "asset_cost": 525000,
    "signature": {
      "present": true,
      "bbox": [100, 200, 300, 250]
    },
    "stamp": {
      "present": true,
      "bbox": [400, 500, 500, 550]
    }
  },
  "confidence": 0.96
}
```

---

## 📊 Performance Targets

| Metric                  | Result              |
| ----------------------- | ------------------- |
| Document-Level Accuracy | ≥ 95%               |
| Avg Processing Time     | ~3–5 sec / document |
| Cost per Document       | <$0.01 (CPU-only)   |

No cloud dependency.
No GPU assumption.

---

## 🧪 Handling No Ground Truth

No labels were provided. We handled this by:

* Manually annotating a **small, diverse subset**
* Bootstrapping pseudo-labels via high-confidence rules
* Cross-validating outputs from independent extraction paths
* Iterative error-driven refinement

This mirrors **real enterprise deployments**, not Kaggle-style setups.

---

## 🔎 Known Failure Modes (We’re Not Pretending Otherwise)

* Extremely blurred photos
* Heavy handwriting over numeric fields
* Overlapping stamp + signature regions

Mitigation strategy:

* Conservative confidence scoring
* Fail-safe defaults instead of hallucination
* Error surfaced explicitly, not hidden

---

## 💡 Why This Architecture Wins

* Modular → easy to debug and extend
* Explainable → every field traceable to OCR regions
* Scalable → invoice-type agnostic
* Cost-aware → survives real bank constraints

This is **production-thinking**, not hackathon theater.

---

## 👥 Team

**Team Convolve 4.0**
IDFC First Bank – Intelligent Document AI Challenge

---