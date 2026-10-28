# matacs-th-2026-02
# Technical Assessment: AI Proposal Triage Design
**Candidate Name:** Muhammad Amir Jawed  
**Reference:** MATACS-TH-2026-02  

---

## 1. AI vs. Rules
In this triage design, tasks are divided strictly between AI and deterministic code based on their core strengths:

* **AI (LLM) Steps:**
  * **Document Parsing & Entity Extraction:** Extracting key features (licence numbers, insurance status, pricing, schedules, warranties, experience counts, team details) from unstructured PDF proposal text.
  * **Prompt Injection / Anomaly Detection:** Scanning the raw text for malicious instructions, automated override commands, or suspicious embedded prompts.
  * **Natural Language Justifications:** Generating a brief 1-sentence summary explaining why a proposal achieved its specific rank or status for reviewer visibility.

* **Deterministic Code / Rules Engine Steps:**
  * **Mandatory Criteria Evaluation (Pass/Fail):** Verifying hard conditions (e.g., presence of bidder's electrical licence, WSIB clearance, $5M insurance, budget ceiling <= $260k CAD, timeline <= 16 weeks) using boolean comparison logic.
  * **Mathematical Scoring & Normalization:** Calculating weighted scores (30% experience, 25% price, 15% timeline, 15% warranty, 15% team) using standard mathematical formulas.
  * **Sorting & Status Assignment:** Assigning status (`ranked`, `ineligible`, `flagged`) and ordering proposals by final calculated score.

**Why draw the line here?** LLMs are inherently probabilistic and prone to hallucination or standard arithmetic errors. Relying on AI for pass/fail checks or scoring math introduces unacceptably high error rates. Conversely, deterministic code cannot parse free-form language from diverse PDF layouts. Using AI purely for structured extraction and code for strict execution guarantees speed, reliability, and precision.

---

## 2. Consistency and Explainability
* **Repeatable Results:**
  * By enforcing strict JSON schemas for LLM extraction, using `temperature=0.0`, and executing all scoring deterministically via code, running the same proposal dataset multiple times yields **identical, 100% reproducible results**.
* **Explainable Scores & Reviewer Visibility:**
  * Every score is fully auditable through a clear breakdown. The system displays:
    1. **Pass/Fail Breakdown:** Explicit indicators for each mandatory check.
    2. **Score Attribution:** Itemized weighted points for Experience, Price, Timeline, Warranty, and Team.
    3. **Plain-Language Summary:** A single-sentence summary generated alongside the exact extracted parameters, allowing human reviewers to cross-check extracted numbers against the uploaded PDF document instantly.

---

## 3. Human in the Loop & Audit Trail
* **Human Checkpoints:**
  * **Pre-Award Final Decision:** The system never awards contracts automatically; it acts strictly as an assistant providing a pre-ranked list.
  * **Reviewer Verification & Overrides:** Reviewers can inspect `flagged` proposals (e.g., Proposal C's injection attempt) or `ineligible` proposals (e.g., Proposal B's deferred licence), manually verify extracted data, and apply a manual override with a required comment.
* **Audit Logging:**
  * Every transaction is logged immutably with a timestamp, user ID, system version, prompt hash, raw extracted JSON, calculated scores, final decision, and any human override reason.

---

## 4. Changing Criteria
When an admin modifies evaluation criteria or weightings after proposals have been scored:
* **Historical Immutability:** Previously finalized/awarded evaluations are locked and archived as historical records.
* **Re-evaluation Trigger (Code-Only Execution):**
  * Because raw extracted features (pricing, timeline, warranties, experience counts) are stored separately in the database as structured JSON, updating weights or threshold criteria **does not require re-running LLM prompts**.
  * The system re-executes the deterministic scoring script against the existing structured data instantaneously, saving API cost and avoiding extraction drift.
  * The reviewer receives a notification highlighting changed ranks resulting from the new weighting scheme.

---

## 5. Risks & Mitigation Safeguards

| Risk | Cause | Safeguard / Mitigation |
| :--- | :--- | :--- |
| **Prompt Injection** | Adversarial text embedded in PDFs (e.g., Proposal C asking to rank first). | System treats document text strictly as passive data inside the system prompt. LLM includes a dedicated injection check step; any detected override command automatically flags the proposal as `flagged` for human review. |
| **Hallucination / False Extraction** | LLM inventing non-existent details or mistaking subcontractor licence for bidder licence. | Strict JSON schema validation; prompt explicitly enforces `null` for unstated facts. Mandatory fields require human confirmation if confidence is low. |
| **Model Drift / Unrepeatability** | Temperature fluctuations or LLM model updates changing scores. | Set `temperature=0.0`. Decouple scoring logic completely from LLM into deterministic Python code. |
| **Data Bias** | Bias in subjective criteria evaluation. | Standardize scoring formulas for quantitative metrics (price, timeline, experience years) so human bias or LLM bias cannot skew rankings. |
| **API Costs & Latency** | Processing large multi-page PDF files repeatedly via LLM. | Cache extracted JSON features in database; perform all re-scoring and filtering in code without re-calling LLM APIs. |