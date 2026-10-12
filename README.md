# UseReady Assignment 1: AI-Based Metadata Extraction from Rental Agreements

[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)
[![Google Gemini API](https://img.shields.io/badge/Google%20GenAI-Gemini%20API-orange.svg)](https://ai.google.dev/)
[![FastAPI](https://img.shields.io/badge/FastAPI-1.0.0-green.svg)](https://fastapi.tiangolo.com/)


An end-to-end, template-agnostic, multimodal metadata extraction system for rental agreements developed entirely within **a single, runnable Jupyter Notebook (`Rental_Agreement_Metadata_Extraction.ipynb`)**.

The system processes both **digital DOCX documents** and **scanned PNG images**, extracting six key metadata fields using **Google Gemini's contextual LLM and multimodal vision capabilities**, validated with **Pydantic**, evaluated on exact-match recall, and exposed via an **in-notebook FastAPI RESTful API**.

---

## Table of Contents
1. [Problem Statement](#1-problem-statement)
2. [Target Metadata Fields](#2-target-metadata-fields)
3. [Solution Architecture](#3-solution-architecture)
4. [Technology Stack](#4-technology-stack)
5. [Project Structure](#5-project-structure)
6. [Security & Gemini API Configuration](#6-security--gemini-api-configuration)
7. [Quickstart Setup](#7-quickstart-setup)
8. [How to Run the Notebook from Beginning to End](#8-how-to-run-the-notebook-from-beginning-to-end)
9. [Generating `predictions.csv`](#9-generating-predictionscsv)
10. [Reproducing Strict and Normalized Recall Calculations](#10-reproducing-strict-and-normalized-recall-calculations)
11. [Evaluation Results & Analysis](#11-evaluation-results--analysis)
12. [FastAPI REST Service (In-Notebook & Standalone)](#12-fastapi-rest-service-in-notebook--standalone)
13. [Automated Testing Suite](#13-automated-testing-suite)
14. [Troubleshooting Guide](#14-troubleshooting-guide)
15. [Limitations & Future Enhancements](#15-limitations--future-enhancements)

---

## 1. Problem Statement

Real estate agreements and commercial contracts vary widely in phrasing, formatting, layout, and document quality. Conventional approaches based on regular expressions (RegEx), coordinate bounding boxes, or rigid templates fail when:
- Clauses are reordered or worded differently across legal jurisdictions.
- Party designations change (e.g., "Lessor", "House Owner", "Landlord", "First Party").
- Scanned documents contain image skew, compression noise, or handwriting artifacts.

**Assignment Requirement:**
Build an AI/ML system that extracts metadata from rental agreements irrespective of template or layout without using regex, document-specific hardcoded conditions, or static positional rules.

---

## 2. Target Metadata Fields

The system extracts exactly six structured metadata fields:

| # | Field Name | CSV Column Header | Description & Format | Example |
|---|---|---|---|---|
| 1 | **Agreement Value** | `Aggrement Value` | Monthly rent as an integer numeric string (currency & commas stripped). | `12000` |
| 2 | **Agreement Start Date** | `Aggrement Start Date` | Tenancy commencement date strictly in `DD.MM.YYYY` format. | `01.04.2008` |
| 3 | **Agreement End Date** | `Aggrement End Date` | Tenancy expiration / end date strictly in `DD.MM.YYYY` format. | `31.03.2009` |
| 4 | **Renewal Notice (Days)** | `Renewal Notice (Days)` | Notice period in days (e.g., 1 month = 30, 2 months = 60). | `60` |
| 5 | **Party One** | `Party One` | Landlord / Lessor / Owner. Personal titles stripped, initials preserved. | `Hanumaiah` |
| 6 | **Party Two** | `Party Two` | Tenant / Lessee. Personal titles stripped, initials preserved. | `Vishal Bhardwaj` |

---

## 3. Solution Architecture

```text
 ┌─────────────────────────────────────────────────────────────┐
 │                   Input Rental Agreement                    │
 │            (.docx, .pdf.docx, or scanned .png)              │
 └──────────────────────────────┬──────────────────────────────┘
                                │
                                ▼
 ┌─────────────────────────────────────────────────────────────┐
 │                 Format & Ingestion Dispatch                 │
 │  - DOCX: Extract paragraphs & table cells in reading order  │
 │  - PNG:  Validate image dimensions & pass to Vision engine  │
 └──────────────────────────────┬──────────────────────────────┘
                                │
                                ▼
 ┌─────────────────────────────────────────────────────────────┐
 │                 Google Gemini API Reasoning                 │
 │  - Context-aware legal extraction prompt                    │
 │  - Zero regex / zero document-specific heuristic rules      │
 │  - Output constrained to structured JSON schema             │
 └──────────────────────────────┬──────────────────────────────┘
                                │
                                ▼
 ┌─────────────────────────────────────────────────────────────┐
 │              Structured Validation & Storage                │
 │  - Pydantic schema validation & empty-string fallback       │
 │  - Batch generation & export to predictions.csv             │
 └──────────────────────────────┬──────────────────────────────┘
                                │
                ┌───────────────┴───────────────┐
                ▼                               ▼
 ┌───────────────────────────┐     ┌───────────────────────────┐
 │ Strict & Normalized Eval  │     │ Embedded FastAPI Service  │
 │ - Exact-match Per-Field   │     │ - GET /health             │
 │ - Normalized Diagnostic   │     │ - POST /extract           │
 │ - Matplotlib visual chart │     │ - In-notebook TestClient  │
 └───────────────────────────┘     └───────────────────────────┘
```

---

## 4. Technology Stack

- **Language:** Python 3.10+ (tested on Python 3.11.16)
- **Primary Interface:** Jupyter Notebook (`Rental_Agreement_Metadata_Extraction.ipynb`)
- **AI Model Provider:** Google Gemini API via official `google-genai` SDK (`gemini-3.5-flash-lite`, with multi-model fallback)
- **Document Processing:** `python-docx` for `.docx` and `.pdf.docx` documents
- **Image Processing:** `Pillow` (`PIL`) for `.png` image verification and multimodal vision input
- **Data Validation:** `pydantic` v2 for strict schema definition and type safety
- **Data Analysis & Evaluation:** `pandas`, `numpy`, `matplotlib`
- **Web Service:** `fastapi`, `uvicorn`, `python-multipart`, `httpx`
- **Configuration & Security:** `python-dotenv`

---

## 5. Project Structure

As mandated by the assignment guidelines, the **entire core implementation resides inside a single Jupyter Notebook**. No external standalone scripts are required:

```text
Rental-metadata-extraction/
├── Rental_Agreement_Metadata_Extraction.ipynb   # Complete runnable implementation (16 sections)
├── predictions.csv                             # Generated predictions on test agreements
├── evaluation_report.txt                       # Detailed exact-match evaluation metrics
├── evaluation_recall_chart.png                 # Visualization of per-field recall
├── Rental_Agreement_Extraction_Data.docx       # Supporting assignment documentation
├── Demo.pdf                                    # Project demonstration & export report
├── requirements.txt                            # Pinned dependency requirements
├── .env.example                                # Configuration template (no secrets)
├── .gitignore                                  # Git exclusion rules (protects .env, .venv & data/)
└── README.md                                   # Comprehensive documentation
```

> **Note on Data Directory:** Raw agreement documents and labels (`data/`) are excluded from GitHub tracking via `.gitignore` to preserve privacy and confidentiality of rental agreements. All reproducible extraction outputs, predictions, and evaluation metrics are preserved directly in `predictions.csv` and `evaluation_report.txt`.

---

## 6. Security & Gemini API Configuration

### Security Policies
- **Zero credential leaks:** Never commit `.env` or paste raw API keys into notebook cells or markdown files.
- The repository `.gitignore` explicitly excludes `.env`, `*.env`, and `.venv/`.
- In Google Colab environments, the notebook securely fetches credentials via Colab Secrets (`google.colab.userdata`).

### Environment Variables
Configure your `.env` file using `.env.example` as a template:

```env
# Google Gemini API Configuration
GEMINI_API_KEY=your_gemini_api_key_here
GEMINI_MODEL=gemini-3.5-flash-lite

# Optional GCP Project Metadata
GCP_PROJECT_NAME=UserReady-1
GCP_PROJECT_NUMBER=704541872626
```

### Model Selection & Free-Tier Quota Handling
The system defaults to `gemini-3.5-flash-lite`. When testing with Google AI Studio keys, rate limits or quota boundaries may occur. The notebook implements an active fallback probing mechanism:
```python
for fb in ["gemini-3.5-flash-lite", "gemini-3.1-flash-lite", "gemini-3-flash-preview", "gemini-3.6-flash"]:
    # Probes model availability and smoothly switches with transparent logging
```
This fulfills the assignment directive:
> *"If unavailable, explain the issue and use a currently supported Flash model only after clearly documenting the change. Do not silently substitute a model."*

---

## 7. Quickstart Setup

```powershell
# 1. Install dependencies
pip install -r requirements.txt

# 2. Configure API key
Copy-Item .env.example .env
# Edit .env with your actual GEMINI_API_KEY

# 3. Launch notebook
jupyter notebook Rental_Agreement_Metadata_Extraction.ipynb
```

---

## 8. How to Run the Notebook from Beginning to End

Launch Jupyter Notebook or JupyterLab:

```bash
jupyter notebook Rental_Agreement_Metadata_Extraction.ipynb
```
*(Or open directly in VS Code / Cursor and select the `.venv` kernel).*

### Sequential Execution Flow
The notebook contains 16 sequential, numbered sections:
1. **Section 1: Introduction** – Problem definition, field requirements, and architecture.
2. **Section 2: Install Dependencies** – Verifies installation of required packages.
3. **Section 3: Imports & Configuration** – Loads `.env`, validates Gemini API connection, and checks model quota.
4. **Section 4: Inspect Dataset** – Examines `data/train.csv`, `data/test.csv`, and file extensions.
5. **Section 5: Document Processing** – Implements `read_docx()` and `read_image()` readers.
6. **Section 6: Gemini Metadata Extraction** – Pydantic schema (`RentalMetadata`) and `extract_metadata()` with exponential backoff retries.
7. **Section 7: Single Document Test** – Interactive extraction test on one document.
8. **Section 8: Batch Processing** – Batch extraction over all files in `data/test/` exporting `predictions.csv`.
9. **Section 9: Strict Evaluation** – Computes per-field exact-match recall against `data/test.csv`.
10. **Section 10: Normalized Evaluation** – Diagnostic comparison with whitespace/punctuation normalization.
11. **Section 11: Visualization** – Generates and saves `evaluation_recall_chart.png`.
12. **Section 12: Save & Export Reports** – Writes summary to `evaluation_report.txt`.
13. **Section 13: REST API Service (FastAPI)** – Defines FastAPI app and validates endpoints using `TestClient`.
14. **Section 14: Automated Tests** – 10 unit tests with mocked Gemini responses ($0 cost, 100% offline).
15. **Section 15: Troubleshooting Guide** – Detailed error remediation guide.
16. **Section 16: Final Summary & Limitations** – Summary of key takeaways and production considerations.

To run all cells: Click **Kernel ➔ Restart & Run All**.

---

## 9. Generating `predictions.csv`

Section 8 of the notebook iterates over all files in `data/test/`:
- `156155545-Rental-Agreement-Kns-Home.pdf.docx`
- `228094620-Rental-Agreement.pdf.docx`
- `24158401-Rental-Agreement.png`
- `95980236-Rental-Agreement.png`

For each document, it invokes `extract_metadata()` and formats the output to match the assignment ground truth schema:
```csv
File Name,Aggrement Value,Aggrement Start Date,Aggrement End Date,Renewal Notice (Days),Party One,Party Two
156155545-Rental-Agreement-Kns-Home,12000,15.12.2012,14.11.2013,30,V.K.NATARAJ,VYSHNAVI DAIRY SPECIALITIES Private Ltd
228094620-Rental-Agreement,15000,07.07.2013,06.06.2014,30,KAPIL MEHROTRA,B.Kishore
24158401-Rental-Agreement,12000,01.04.2008,31.03.2009,60,Hanumaiah,Vishal Bhardwaj
95980236-Rental-Agreement,9000,01.04.2010,31.03.2011,30,S.Sakunthala,V.V.Ravi Kian
```
The file is exported directly to `predictions.csv` in the root folder.

---

## 10. Reproducing Strict and Normalized Recall Calculations

### Strict Exact-Match Recall Formula
$$\text{Strict Recall (Field)} = \frac{\text{Matches}}{\text{Total Evaluated Documents}} \times 100\%$$
- A match is counted **only if** `predicted_string == ground_truth_string` strictly.
- Handled in **Section 9** of the notebook.

### Normalized Diagnostic Recall Formula
- Strips extraneous leading/trailing spaces and stray punctuation artifacts (`.`, `,`, `"`) while preserving legitimate initials (e.g., `V.K.NATARAJ`).
- Handled in **Section 10** of the notebook.

---

## 11. Evaluation Results & Analysis

### Summary Results on `data/test/` (4 Documents, 24 Field Comparisons)

| Metadata Field | Strict Recall (%) | Strict Matches | Normalized Recall (%) | Normalized Matches |
|---|---|---|---|---|
| **Aggrement Value** | **100.00%** | 4 / 4 | **100.00%** | 4 / 4 |
| **Aggrement Start Date** | **100.00%** | 4 / 4 | **100.00%** | 4 / 4 |
| **Aggrement End Date** | **100.00%** | 4 / 4 | **100.00%** | 4 / 4 |
| **Renewal Notice (Days)** | **100.00%** | 4 / 4 | **100.00%** | 4 / 4 |
| **Party One** | **100.00%** | 4 / 4 | **100.00%** | 4 / 4 |
| **Party Two** | **75.00%** | 3 / 4 | **100.00%** | 4 / 4 |
| **Overall Field Score** | **95.83%** | **23 / 24 fields** | **100.00%** | **24 / 24 fields** |

### Key Evaluation Insights:
1. **Primary Strict Recall (95.83%):** 5 out of 6 fields achieved a perfect 100.00% strict match across all test agreements.
2. **The Party Two Discrepancy (75.00% strict vs. 100.00% normalized):**
   - In document `228094620-Rental-Agreement`, the ground truth row in `data/test.csv` has the value `".B.Kishore "` (containing a typographical leading period and trailing space).
   - The model cleanly extracted `"B.Kishore"`.
   - Under strict string comparison, `".B.Kishore "` $\neq$ `"B.Kishore"`, yielding 75.00% strict recall.
   - Under normalized evaluation, both resolve to `"B.Kishore"`, achieving 100.00% recall.
3. **Preservation of Evaluation Integrity:** The strict recall metric is reported as **95.83%** as mandated by assignment rules without altering ground truth labels.

---

## 12. FastAPI REST Service (In-Notebook & Standalone)

The FastAPI web service is defined in notebook **Section 13**.

### Endpoints
- `GET /health`: Returns service status, active model, and supported file extensions.
- `POST /extract`: Accepts a multipart form upload (`file`: `.docx` or `.png`). Returns extracted metadata dictionary.

### In-Notebook Verification (Verified via `TestClient`)
Because the assignment requires all implementation to remain inside the notebook, endpoint verification is executed directly in Section 13b without external port conflicts:
```python
from fastapi.testclient import TestClient

api_client = TestClient(app)

# 1. Health check
response = api_client.get("/health")
print(response.json())
# Returns: {"status": "healthy", "service": "Rental Agreement Metadata Extractor", ...}

# 2. Extract metadata
with open("data/test/24158401-Rental-Agreement.png", "rb") as f:
    response = api_client.post("/extract", files={"file": ("24158401-Rental-Agreement.png", f, "image/png")})
print(response.json())
# Returns: {"filename": "24158401-Rental-Agreement.png", "status": "success", "metadata": {...}}
```

### Standalone Server Execution
If standalone server execution outside the notebook is required, export the notebook using nbconvert:
```bash
jupyter nbconvert --to script Rental_Agreement_Metadata_Extraction.ipynb
uvicorn Rental_Agreement_Metadata_Extraction:app --host 127.0.0.1 --port 8000 --reload
```
Interactive Swagger API documentation is available at `http://127.0.0.1:8000/docs`.

---

## 13. Automated Testing Suite

Implemented directly in **Section 14** of the notebook. All unit tests run in seconds using `unittest.mock` to mock Gemini API calls ($0 cost, 100% offline).

### Tests Included:
1. **Compound Extension Stripping:** Verifies `.pdf.docx`, `.docx`, and `.png` handling.
2. **Pydantic Schema Validation:** Validates `RentalMetadata` fields and instantiation.
3. **Normalization Logic:** Verifies whitespace and outer punctuation stripping without corrupting internal initials.
4. **Image Verification:** Verifies image opening and header validation with Pillow.
5. **DOCX Parsing:** Verifies paragraph and tabular extraction with `python-docx`.
6. **Mocked Gemini Extraction:** Verifies extraction output mapping with mocked LLM response.
7. **Strict Recall Formula:** Mathematical verification of the recall computation.
8. **Fallback Handling:** Ensures empty string fallback on unparseable/error responses.
9. **FastAPI Rejection:** Verifies unsupported file types receive `HTTP 415 Unsupported Media Type`.
10. **Predictions Schema Alignment:** Ensures `predictions.csv` columns match `data/test.csv` headers exactly.

**Result:** `10 / 10 tests passed successfully.`

---
 
## 14. Troubleshooting Guide

| Issue | Cause | Solution |
|---|---|---|
| **`GEMINI_API_KEY is not configured`** | Missing `.env` file or empty key. | Create `.env` using `.env.example` and set `GEMINI_API_KEY`. |
| **`429 RESOURCE_EXHAUSTED`** | Daily quota limit reached on the active model. | Notebook automatically probes and switches to alternative verified Flash models (`gemini-3.5-flash-lite`, `gemini-3.1-flash-lite`). |
| **`HTTP 415 Unsupported Media Type`** | Uploaded file is not `.docx` or `.png`. | Upload only supported agreement formats (`.docx` or `.png`). |
| **`Script Execution Error (PowerShell)`** | PowerShell execution policy restriction. | Run `Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass` before activating `.venv`. |

---

## 15. Limitations & Future Enhancements

1. **Test Set Scale:** The test suite evaluates 4 agreements. Expanding testing to larger datasets across different Indian states will validate stamp duty and regional naming variations.
2. **Low-Resolution Scans:** Extremely low-DPI or noisy mobile captures can benefit from automated deskewing and adaptive contrast enhancement prior to vision input.
3. **Multi-Party Signatories:** Agreements with multiple landlords or multiple tenants can be enhanced with an array-based schema for all signatories.
