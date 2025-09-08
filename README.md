# DocuSwift – Agentic AI Document Processing (Android + Python)

DocuSwift is a production-style, **Agentic AI** system that automates **OCR → key information extraction (KIE) → PII redaction → voice summary** across scanned, printed, and handwritten documents. The app uses a **Kotlin/Jetpack Compose** Android frontend and a **Python/FastAPI** backend, with a multilingual OCR pipeline and pluggable LLMs (**GPT-3.5, LLaMA, Gemma 2**) for robust extraction and reasoning.

---

## ✨ What it does
- **Multilingual OCR** (EN/HI/TA) → cleans text even from noisy scans
- **Agentic flow** (perceive → plan → tool-use → act → learn) with human-in-the-loop
- **Key field extraction** via LLMs; supports **model A/B tests and ensembles**
- **Auto-redaction** of PII (Aadhaar/PAN/phones/emails/accounts) for safe sharing
- **Voice interface**: optional text-to-speech summary of extracted data
- **Mobile-first UX**: capture via camera, view redacted output, export/share

---

## 🧠 Why Agentic AI here?
Documents are messy: languages change, layouts vary, scans are blurry. A rigid pipeline breaks easily. DocuSwift’s agent **decides dynamically**: when to re-OCR, when to translate, which model to call, how to validate output, and when to prompt the user.

---

## 🏗️ Architecture at a glance
**Android (Kotlin/Compose)** ↔ **FastAPI (Python)** ↔ **OCR + LLMs + Redaction**

Flow:
1) Upload/capture document (Android)  
2) Backend OCR (Tesseract/EasyOCR) + **OCR Success Score**  
3) If score ≥ 85% → proceed; 50–84% → ask user to proceed or re-upload; <50% → request clearer doc  
4) Document type classification (rules/ML)  
5) KIE via selected LLM (GPT-3.5 / LLaMA / Gemma 2)  
6) PII redaction (regex + NER)  
7) Return **JSON + redacted text/pdf + optional TTS**  
8) Human review → corrections logged for improvement

---

## 🔑 Key Features / USPs
- **OCR Success Score** (quality gate): density + dictionary match + context keywords
- **Model-agnostic**: swap/stack **GPT-3.5, LLaMA, Gemma 2**; supports single, pairwise, and tri-model ensembles
- **Multilingual** OCR/NLP: English, Hindi, Tamil (extensible)
- **Universal redaction**: PII masked regardless of document sector/type
- **Security by design**: password-gated uploads, access-scoped URLs, redacted-by-default responses

---

## 🧩 Tech Stack
- **Frontend:** Kotlin, Jetpack Compose, Retrofit, Android TTS (optional)
- **Backend:** Python 3.10+, FastAPI, Uvicorn
- **OCR:** Tesseract/EasyOCR (+ pre-processing)
- **LLMs:** GPT-3.5 (API), LLaMA (HF/Local), Gemma 2 (Colab/Kaggle/Local)
- **NLP/PII:** spaCy + regex
- **Data:** SQLite/JSON storage (demo), pluggable for cloud DB
- **Auth:** Simple token/password (demo), pluggable for OAuth/JWT

---

## 🧪 Models & Ensemble Strategy
**Final Model Set**
- **GPT-3.5** – fast, reliable, low cost (API)
- **LLaMA** – open-source, customizable (local/HF)
- **Gemma 2** – student-friendly, strong multilingual

**Evaluation Phases**
1. **Single-model**: compare accuracy, speed, robustness
2. **Two-model combos**: A→B refinement, split tasks, or voting
3. **All-three**: parallel voting or tiered pipeline (draft→validate→polish)

**Metrics**
- Field extraction accuracy (per-field F1)
- PII recall/precision
- Robustness on noisy/multilingual scans
- Latency & cost per document

---

## 📏 OCR Success Score (quality gate)
Weighted score ∈ [0,100]:
- **Text Density** (40%): non-space chars vs. expected length
- **Dictionary/Script Match** (30%): valid words/script ratio
- **Context Keywords** (30%): doc-type anchors present (e.g., “DOB”, “Account”, “Govt of India”)

Thresholds:
- **≥ 85%** → proceed
- **50–84%** → ask user to confirm proceed/re-upload
- **< 50%** → request clearer document

---

## 🔐 Data Handling & Security (demo)
- **Password at upload** to bind original doc to uploader
- Responses return **redacted** content by default
- Hashed filenames, signed URLs (short TTL)
- Local dev secrets via `.env` (never commit API keys)

> ⚠️ **Note:** Demo security is educational; harden for production (JWT, KMS, VPC, audits).

---

## 📱 Android App (screens)
- **Login** (password for upload access)  
- **Capture/Upload** (camera or file picker)  
- **Processing** (progress + OCR score/status)  
- **Results** (JSON fields, redacted preview, download/share)  
- **Voice summary** (optional)

---

## 🔌 API (FastAPI) – example endpoints
- `POST /upload` → multipart file + password → `{upload_id}`
- `POST /process` → `{upload_id, model:"gpt35|llama|gemma2"}` → `{ocr_score, doc_type, fields:{...}}`
- `POST /redact` → `{upload_id, fields}` → `{redacted_text, pii_report}`
- `GET /download/{upload_id}` → redacted PDF/text

Example output (trimmed):
```json
{
  "ocr_score": 91.7,
  "doc_type": "aadhar_card",
  "fields": {
    "name": "Ram Kumar",
    "dob": "1983-07-01",
    "aadhar": "XXXX XXXX 1234"
  },
  "pii_report": ["aadhar", "phone"]
}
