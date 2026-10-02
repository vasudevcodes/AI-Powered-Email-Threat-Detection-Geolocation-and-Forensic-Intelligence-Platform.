# Comparative Research Analysis: Paper 1 vs. Paper 2

**Comparative Evaluation of:**
1. **Paper 1 (2025):** *A Hybrid Approach for Phishing Email Detection using AI and Cryptography* (Kadiatou Diallo & Puneet Thapar, IEEE ICPCSN-2025)
2. **Paper 2 (2026):** *Transformer-Driven NLP Framework for High-Precision Phishing Email Identification* (Shafiya Noorain, Dr. Jyoti Metan, Tamirah Sharieff, Zoya Harmain, IEEE CONIT-2026)

---

## 1. Direct Side-by-Side Comparison Matrix

| Evaluation Dimension | Paper 1 (Diallo & Thapar, 2025) | Paper 2 (Noorain et al., 2026) |
| :--- | :--- | :--- |
| **Primary Focus** | Transport security, sender authentication & shallow ML | Deep semantic NLP, multi-modal threat intel & user interaction |
| **Cryptographic Authentication** | **Extensive:** SPF, DKIM, DMARC, S/MIME, PGP, attachment digital signatures | **None:** Completely omitted |
| **Email Body NLP Method** | **Shallow Heuristics:** Keyword scanning, spelling checks, personalization checks mapped to numeric vector | **Deep Transformers:** BERT, RoBERTa, DeBERTa (fine-tuned) with self-attention & WordPiece |
| **URL Analysis Technique** | Text-vs-link mismatch, domain age lookup (WHOIS), shortener detection | Lexical feature correlation + Transformer-based URL token classification |
| **Network & IP Intelligence** | Static IP blacklist query / threat database mention | **AbuseIPDB API integration**, GeoIP mapping, VPN/Proxy exit-node anomaly detection |
| **Attachment Security** | **Present:** File extension scan, script scan (JS/VBA), digital signature verification | **Omitted:** Deferred to future work |
| **Machine Learning Models** | Random Forest (primary classifier), XGBoost (mentioned) | BERT, RoBERTa, DeBERTa (fine-tuned language models) |
| **Ensemble / Fusion Logic** | Majority voting across Random Forest decision trees | Multi-modal **Decision Fusion Layer** via weighted ensemble scoring |
| **User Interaction & Defense** | Passive gateway actions: spam folder, warning banner, quarantine/block | **Active & Proactive:** React/Electron UI, color-coded badges, real-time AI reply assistant |
| **Datasets Used** | **None:** Purely conceptual/theoretical framework | **Empirical Benchmark:** Enron (~500k), Nazario (~50k), PhishTank (~150k), AbuseIPDB (~300k) |
| **Reported Results** | No numerical performance figures reported (theoretical only) | **Paper Reported:** Precision 0.96, Recall 0.94, F1 0.95, ROC-AUC 0.98 on NVIDIA A100 |
| **Deployment Architecture** | High-level conceptual workflow (Figs. 1–5) | Production microservice stack: FastAPI, React.js, Electron, Docker |
| **Key Architectural Strength** | Stops domain spoofing and verifies sender/document integrity deterministically | Detects sophisticated, zero-day, adversarial, and evasive linguistic phishing lures |
| **Critical Vulnerability / Blind Spot** | Blind to compromised legitimate accounts; weak on adversarial text; no empirical tests | Blind to forged `From` headers and spoofed servers without cryptographic verification |

---

## 2. Research Accuracy & Result Attribution Disclaimers

> [!IMPORTANT]
> **Strict Research Attribution Rule:**
> - The metrics reported in Paper 2 (**Precision: 0.96, Recall: 0.94, F1-Score: 0.95, ROC-AUC: 0.98**) represent the academic findings of Noorain et al. on an enterprise cluster equipped with an **Intel Xeon CPU and NVIDIA A100 GPU** evaluated on their specific dataset snapshots.
> - **These figures MUST NEVER be cited as our project's achieved accuracy.**
> - Our implementation will be independently benchmarked on our hardware target (**NVIDIA RTX 3050 6GB Laptop GPU, 16GB RAM**), and all resulting performance metrics (Precision, Recall, F1, Latency) will be empirically measured and documented separately in our test reports.

---

## 3. Four-Tier Feature Lineage Matrix

The table below delineates the origin of every capability in our platform:

| Feature Category | Capability / Module | Lineage Classification | Detailed Technical Description & Provenance |
| :--- | :--- | :---: | :--- |
| **Transport & Auth** | SPF DNS Verification (RFC 7208) | **A. Directly Adopted from Paper 1** | Evaluates authorized sending MTAs via DNS TXT records. |
| **Transport & Auth** | DKIM Signature Verification (RFC 6376) | **A. Directly Adopted from Paper 1** | Cryptographic verification of header and body hashes using public keys published in DNS. |
| **Transport & Auth** | DMARC Policy Evaluation (RFC 7489) | **A. Directly Adopted from Paper 1** | Validates SPF/DKIM alignment with `From:` header domain. |
| **Transport & Auth** | S/MIME & PGP Signature Checking | **A. Directly Adopted from Paper 1** | Validates PKI X.509 certificates and GPG digital signatures on signed email payloads. |
| **Attachments** | Static Extension & Magic Byte Verification | **A. Directly Adopted from Paper 1** | Flags dangerous executables (`.exe`, `.bat`, `.scr`) and mismatching binary signatures without executing code. |
| **Attachments** | Macro & Script Heuristic Scanning | **A. Directly Adopted from Paper 1** | Static detection of VBA macros (`.docm`, `.xlsm`) and suspicious JavaScript blocks without code execution. |
| **NLP & Semantics** | Transformer Contextual Text Analysis | **B. Directly Adopted from Paper 2** | Evaluates body text using self-attention mechanisms to detect psychological coercion and urgency. |
| **NLP & Semantics** | Subword WordPiece Tokenization | **B. Directly Adopted from Paper 2** | Subword encoding robust against zero-width spaces, character homoglyphs, and obfuscations. |
| **Network Intel** | IP Threat Reputation Scoring | **B. Directly Adopted from Paper 2** | Evaluates IP threat scores (*demoted to an optional plugin in our platform*). |
| **Network Intel** | GeoIP & Proxy / VPN Detection | **B. Directly Adopted from Paper 2** | Maps IP to geographic location and flags proxy/VPN/Tor exit-node routing anomalies. |
| **Interface** | Color-Coded Threat Indicators | **B. Directly Adopted from Paper 2** | Visual badges (Safe, Suspicious, Phishing) in the UI for rapid human comprehension. |
| **Architecture** | FastAPI Backend Serving | **B. Directly Adopted from Paper 2** | Asynchronous REST endpoints for high-throughput email analysis. |
| **Email Ingestion** | RFC 5322 MIME Disassembly | **C. Combined Feature** | Extracts headers, multipart bodies, URLs, and attachments (common to Paper 1 & 2). |
| **URL Analysis** | Multi-Vector Link Inspection | **C. Combined Feature** | Combines Paper 1's anchor-vs-destination mismatch check and shortener detection with Paper 2's lexical metrics (`url_length`, dots, hyphens, `@`, `%`) and PhishTank lookup. |
| **Decision Logic** | Multi-Modal Decision Fusion | **C. Combined Feature** | Integrates Paper 1's authentication/attachment flags with Paper 2's weighted ensemble score ($S_{nlp}, S_{url}, S_{ip}$). |
| **Transport Forensics**| Evidence-Based 6-State Auth Evaluation | **D. Additional Engineering Contribution** | Replaces binary pass/fail with discrete forensic states: `PASS`, `FAIL`, `NONE`, `NOT CONFIGURED`, `NOT VERIFIABLE`, `TEMPORARY ERROR`. Prevents classifying emails as phishing solely due to missing auth. |
| **Transport Forensics**| Hop-by-Hop Transmission Delay & Route Trace | **D. Additional Engineering Contribution** | Reconstructs the complete MTA relay path across multiple `Received:` headers to identify transit delays and relay spoofing. |
| **Transport Forensics**| Ambiguity-Tolerant Originating IP Handling | **D. Additional Engineering Contribution** | Categorizes IP evidence into `EXTRACTED_PUBLIC`, `PROVIDER_GATEWAY`, `INTERNAL_PRIVATE`, `UNAVAILABLE` without penalizing masked webmail hops. |
| **Security Policy** | Strict Non-Execution of Attachments & URLs | **D. Additional Engineering Contribution** | Strictly passive and static inspection; zero file execution, zero macro execution, zero HTTP GET calls to unknown links. |
| **Model Serving** | RTX 3050 GPU & CPU INT8 Deployment | **D. Additional Engineering Contribution** | Adapts Paper 2's heavy A100 pipeline to run on an RTX 3050 6GB GPU (`deberta-v3-small` in FP16) with a verified CPU INT8 quantization fallback. |
| **Map Visualization** | Leaflet + OpenStreetMap + Offline SVG Fallback | **D. Additional Engineering Contribution** | 100% free web map complying with OSM tile usage policies, with an embedded offline SVG schematic fallback when disconnected. |
| **Risk Modeling** | Anti-Double-Counting Weighted Scoring | **D. Additional Engineering Contribution** | Mathematically orthogonal penalty allocation ensuring that related signals do not unfairly penalize a single anomaly twice. |
| **Data Architecture** | Strictly Partitioned Dataset Pipeline | **D. Additional Engineering Contribution** | Isolated data pipelines for Text, URLs, and IPs with deduplication, text normalization, and strict leakage prevention. |
| **Reporting** | Automated PDF & JSON Forensic Dossier Export | **D. Additional Engineering Contribution** | Generates court-ready and SOC-ready structured JSON data and formatted PDF incident reports using free ReportLab Community Edition. |
| **Offline Operation** | 100% Free Software & Offline Operation | **D. Additional Engineering Contribution** | Operates completely offline with local GeoLite2/DB-IP databases, local PhishTank CSV caches, and zero paid API keys. |

---

## 4. Free Software vs. Free Online Services with Restrictions

To maintain transparency and prevent service disruption, the platform distinguishes:
1. **Free & Open-Source Software (FOSS):** Runs locally on your machine with no usage caps, no accounts, and no network dependencies (Python, PyTorch, Transformers, FastAPI, React, Leaflet, ReportLab).
2. **Free Online Services with Restrictions:**
   - **OpenStreetMap Tile Servers:** Free community service. Must comply with OSM Tile Usage Policy (requires custom descriptive User-Agent, browser caching, no bulk scraping). Offline SVG fallback provided when disconnected.
   - **Public DNS Resolvers (8.8.8.8, 1.1.1.1):** Standard DNS resolution for SPF/DKIM. Cached via local TTL in-memory cache. If offline, non-punitive state (`TEMPORARY ERROR`) is returned.
   - **AbuseIPDB (Optional Free Tier):** Non-mandatory community feed (1,000 requests/day with registration). If no API key is provided or the system is offline, the platform operates normally with a neutral score ($S_{ip}=0.0$).
