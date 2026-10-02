# Implementation Roadmap: AI-Powered Email Threat Detection, Geolocation and Forensic Intelligence Platform

---

## 1. Engineering Discipline & Free-Only Mandates

This roadmap defines the implementation plan for the **PhishIntel Platform**, tailored specifically for Windows on an **NVIDIA GeForce RTX 3050 6GB GPU with 16GB RAM** at **zero software cost**.

### Strict Operational Constraints
1. **100% Free & Open-Source:** Zero paid APIs, zero subscriptions, zero paid billing profiles, and zero commercial licenses.
2. **Offline & Air-Gap Resilience:** Core capabilities (parsing, 6-state auth, lexical URL analysis, static attachment inspection, transformer NLP, and IP geolocation) function with zero network access.
3. **No Assumed Benchmarks:** Latency, accuracy, and VRAM figures will not be assumed. They will be empirically measured and documented on your RTX 3050 laptop GPU and CPU.
4. **Separate CPU INT8 Fallback:** Dynamic INT8 quantization on CPU will be implemented and validated as a distinct fallback pathway.
5. **Strict Non-Execution Security Policy:** Attachments and extracted URLs are evaluated **strictly statically**. The platform will **never execute attachments, run macros, or make outbound HTTP requests to unknown links**.
6. **Tolerant IP Extraction:** The engine explicitly handles ambiguous or unavailable originating IPs (webmail proxies, RFC 1918 internal subnets, missing `Received` headers) without penalizing emails as malicious.
7. **Empirical Independence:** Performance will be reported transparently on actual held-out test splits without claiming 100% accuracy from small samples.
8. **Phase-by-Phase Gatekeeping:** **Phase 1 is executed first.** Phase 2 will not commence until Phase 1 has been completed, tested, and approved.

---

## 2. Phase-by-Phase Implementation Roadmap

```mermaid
gantt
    title PhishIntel Phase-Wise Implementation Roadmap
    dateFormat  YYYY-MM-DD
    section Phase 1 (CURRENT)
    Phase 1: Ingestion & RFC 5322 Parsing   :active, p1, 2026-10-05, 4d
    section Phase 2-3 (Awaiting Approval)
    Phase 2: 6-State Email Authentication   :p2, after p1, 5d
    Phase 3: IP Geolocation & URL Forensics :p3, after p2, 5d
    section Phase 4-6 (Awaiting Approval)
    Phase 4: Attachment Static Forensics    :p4, after p3, 4d
    Phase 5: Transformer NLP (GPU & CPU)    :p5, after p4, 6d
    Phase 6: Explainable Risk & Reporting   :p6, after p5, 5d
    section Phase 7-9 (Awaiting Approval)
    Phase 7: FastAPI Backend Serving        :p7, after p6, 5d
    Phase 8: React Dashboard & Leaflet Map  :p8, after p7, 6d
    Phase 9: Empirical Benchmark Report     :p9, after p8, 4d
```

---

### Phase 1: Environment Setup, RFC 5322 Email Parser & Hop Extractor (REFINED & AUDITED)
- **Goal:** Ingest and parse raw `.eml` and `.msg` email files into structured forensic components using 100% local, free Python standard library and FOSS tools.
- **Components Implemented & Refined:**
  - `src/core/models.py`: Defines `ParsedEmail`, `IngestionContext`, and discrete `IPStatus` categories (`INTERNAL_PRIVATE`, `LOOPBACK`, `LINK_LOCAL`, `CGNAT`, `DOCUMENTATION`, `RESERVED_MULTICAST`, `PROVIDER_GATEWAY`, `EXTRACTED_PUBLIC`, `UNAVAILABLE`).
  - `src/core/parser.py`: Robust MIME parser accepting optional `IngestionContext`, separating candidate boundary IP from verified boundary IP evidence.
  - `src/core/hop_extractor.py`: Chronologically parses `Received:` hops, identifies delay anomalies, prevents forged boundary headers from claiming authenticity on unauthenticated uploads, and classifies IPs across fine-grained RFC standards.
  - `src/core/body_extractor.py`: Pure-Python HTML sanitizer powered by Python's built-in `html.parser` (replacing deprecated Bleach, zero binary/DLL dependencies, AppLocker-safe, PSF license). Strips active script/style/svg/math tags, neutralizes remote tracking beacons (`<img src="...">` -> `data-blocked-src`), enforces `rel="noopener noreferrer"`, properly closes malformed tags, and securely falls back to `html.escape()`.
  - `src/core/url_extractor.py`: Identifies and extracts all embedded hyperlinks from HTML anchor tags and plaintext body text. **Strict non-execution; zero HTTP requests.**
  - `src/core/attachment_extractor.py`: Extracts attachment metadata (filename, declared MIME type, size, SHA-256 hash, magic bytes). **Strict non-execution; zero file execution.**
- **Verification Gate:**
  - Run `tests/test_parser.py` (18 automated tests across 7 `.eml` fixtures).
  - All 18 tests passed in 0.21s with zero warnings, zero socket calls, and 100% local execution.
  - **Free & Offline Audit:** 100% free open-source software; verified zero network calls during parsing.

---

### Phase 2: Evidence-Based Cryptographic Verification Engine (Awaiting Approval)
- **Goal:** Implement non-punitive, 6-state forensic authentication checks for SPF, DKIM, and DMARC using free DNS resolution libraries.
- **Key Deliverables:**
  - `src/auth/spf_validator.py`: Resolves SPF TXT records against originating IP.
  - `src/auth/dkim_validator.py`: Verifies DKIM cryptographic RSA/Ed25519 signatures.
  - `src/auth/dmarc_validator.py`: Evaluates SPF/DKIM identifier alignment.
  - `src/auth/engine.py`: Maps authentication results into six discrete forensic states:
    `PASS`, `FAIL`, `NONE`, `NOT CONFIGURED`, `NOT VERIFIABLE`, `TEMPORARY ERROR`.
  - `src/auth/dns_cache.py`: In-memory TTL cache preventing duplicate DNS lookups.
- **Verification Gate:**
  - Verify missing SPF/DKIM returns `NONE` or `NOT CONFIGURED` with zero penalty.
  - Verify DNS timeouts return `TEMPORARY ERROR` without crashing.

---

### Phase 3: IP Geolocation, Network Intelligence & URL Forensic Engine (Awaiting Approval)
- **Goal:** Determine originating sender geolocation and evaluate URL risk without requiring paid API keys or mandatory external services.
- **Key Deliverables:**
  - `src/intel/geoip_lookup.py`: 100% offline geolocation resolver using free **MaxMind GeoLite2 City & ASN** / **DB-IP Lite** `.mmdb` database file.
  - `src/intel/abuseipdb_client.py`: *Optional* plugin for AbuseIPDB free tier. Gracefully disabled when no key is configured.
  - `src/intel/url_forensics.py`:
    - Computes lexical features (`url_length`, dot count, hyphen count, `@`, hex chars, raw IP hosts).
    - Checks for anchor-vs-destination mismatch (`<a href="phish.com">paypal.com</a>`).
    - Matches links against a local cached copy of the **PhishTank community database**.
    - **Never initiates outbound HTTP connections to unknown URLs.**
- **Verification Gate:**
  - Offline GeoLite2 lookups return coordinates in $<5$ ms; URL analyzer detects lexical anomalies with internet disconnected.

---

### Phase 4: Attachment Static Forensics Engine (Awaiting Approval)
- **Goal:** Perform static analysis on file attachments to detect malicious executables and script payloads with zero execution and zero sandbox fees.
- **Key Deliverables:**
  - `src/attachments/magic_detector.py`: Validates declared file extensions against actual file magic bytes (detecting disguised executables like `statement.pdf.exe`).
  - `src/attachments/macro_scanner.py`: Static inspection using open-source `oletools` to detect embedded VBA macros and auto-executing triggers (`AutoOpen`, `Shell`). **Never executes macros.**
  - `src/attachments/archive_inspector.py`: Inspects contents of `.zip` or `.rar` archives statically.
- **Verification Gate:**
  - Static detection of macro-enabled documents, extension-mismatch binaries, and benign files without code execution.

---

### Phase 5: Transformer NLP Pipeline (GPU & CPU Pathways) (Awaiting Approval)
- **Goal:** Deploy a transformer model to classify email text semantics locally on RTX 3050 GPU and CPU with zero cloud API calls.
- **Key Deliverables:**
  - `src/nlp/preprocessor.py`: Text cleaning, synthetic header stripping, and WordPiece tokenization.
  - `src/nlp/gpu_pipeline.py`: Hugging Face PyTorch pipeline utilizing **`microsoft/deberta-v3-small`** (or fine-tuned `distilbert-base-uncased`) with **FP16 mixed precision** (`torch.cuda.amp`) on the RTX 3050 GPU.
  - `src/nlp/cpu_fallback.py`: Distinct CPU pipeline using dynamic INT8 quantization (`torch.quantization.quantize_dynamic`) executing on system RAM.
  - `src/nlp/semantic_scorer.py`: Computes continuous $S_{nlp} \in [0.0, 1.0]$ confidence score and extracts attention keyword triggers.
- **Verification Gate:**
  - Empirically measure and report actual VRAM consumption and inference latency on RTX 3050 GPU.
  - Empirically verify that the CPU INT8 fallback pathway operates reliably without CUDA.

---

### Phase 6: Explainable Risk Engine & Report Generator (Awaiting Approval)
- **Goal:** Calculate an anti-double-counting composite risk score and generate PDF and JSON forensic reports using free open-source tools.
- **Key Deliverables:**
  - `src/risk/config_loader.py`: Loads configurable factor weights ($w_{nlp}=0.30$, $w_{auth}=0.25$, $w_{url}=0.20$, $w_{att}=0.15$, $w_{ip}=0.10$) and thresholds from `config.yaml`.
  - `src/risk/engine.py`: Computes composite risk score $Score_{Final}$ with strict orthogonal penalty assignment, preventing double-counting.
  - `src/risk/attributor.py`: Generates human-readable factor attribution breakdown cards.
  - `src/reports/json_exporter.py`: Serializes complete forensic state to standardized JSON.
  - `src/reports/pdf_generator.py`: Generates publication-grade PDF forensic dossiers using free open-source **ReportLab Community Edition**.
- **Verification Gate:**
  - Confirm correct weighted score calculation; verify generation of valid `.pdf` and `.json` reports.

---

### Phase 7: FastAPI Backend Service (Awaiting Approval)
- **Goal:** Expose all forensic capabilities via high-performance asynchronous REST endpoints using open-source FastAPI.
- **Key Deliverables:**
  - `src/api/main.py`: FastAPI entry point with CORS middleware.
  - `src/api/routes/scan.py`: `POST /api/v1/scan` supporting multipart `.eml` upload and raw text.
  - `src/api/routes/reports.py`: `GET /api/v1/reports/{id}/pdf` and `GET /api/v1/reports/{id}/json`.
  - `src/api/schemas.py`: Pydantic request/response validation models.
- **Verification Gate:**
  - Confirm HTTP 200 responses, schema validation, and sub-150 ms response times.

---

### Phase 8: Modern React Forensic Dashboard & Leaflet Map (Awaiting Approval)
- **Goal:** Build an intuitive, responsive web interface for security analysts using React, Leaflet, and OpenStreetMap.
- **Key Deliverables:**
  - `frontend/src/components/UploadDropzone.tsx`: Drag-and-drop `.eml` upload.
  - `frontend/src/components/ThreatOverview.tsx`: Color-coded risk meter (Safe / Suspicious / Malicious).
  - `frontend/src/components/AuthEvidenceCard.tsx`: 6-state badges for SPF, DKIM, DMARC.
  - `frontend/src/components/GeoMap.tsx`: Map showing sender geolocation using **Leaflet and OpenStreetMap** (respecting OSM tile policies, with offline SVG fallback).
  - `frontend/src/components/UrlAttachmentTable.tsx`: Tabular breakdown of extracted links and static attachment findings.
  - `frontend/src/components/ReportDownloadBar.tsx`: Download buttons for PDF and JSON forensic dossiers.
- **Verification Gate:**
  - Verify UI upload, forensic card rendering, map display, and PDF report download.

---

### Phase 9: Empirical Benchmarking & Research Independence (Awaiting Approval)
- **Goal:** Conduct empirical evaluation on our hardware target and document project metrics separately from research paper results.
- **Key Deliverables:**
  - Benchmark pipeline on held-out test split of legitimate and phishing samples.
  - Empirically measure and report actual Precision, Recall, F1-Score, False Positive Rate, and Latency on the RTX 3050 GPU and CPU.
  - Document actual results and limitations transparently.

---

## 3. Strict Zero-Cost & Open-Source Compliance Checklist

```
┌────────────────────────────────────────────────────────────────────────┐
│                   ZERO-COST COMPLIANCE VERIFICATION                    │
├────────────────────────────────────────────────────────────────────────┤
│ [x] 1. Zero Paid APIs: No OpenAI, Grok, Gemini, or VirusTotal keys.    │
│ [x] 2. 100% Local Models: DeBERTa runs via local HuggingFace / PyTorch.│
│ [x] 3. Zero Paid Maps: Leaflet + OpenStreetMap (no Mapbox/Google keys).│
│ [x] 4. Free Geolocation: Local GeoLite2 / DB-IP .mmdb database.        │
│ [x] 5. Free Threat Feeds: Local PhishTank CSV cache.                   │
│ [x] 6. AbuseIPDB Demoted: Strictly optional enricher; not core.        │
│ [x] 7. Strict Non-Execution: Zero execution of attachments/URLs.       │
│ [x] 8. Free PDF Engine: ReportLab Community Edition (BSD license).     │
│ [x] 9. Demonstrable without Billing: 100% runnable with no payment info│
└────────────────────────────────────────────────────────────────────────┘
```
