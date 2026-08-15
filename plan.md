# Implementation Plan: Field Service Report Summarizer

## 1. Technical Stack & Approach
*   **Language:** Python 3.
*   **Input/Output:** The tool will read from `data/service_reports.jsonl` sequentially and write the outputs to the console (or a distinct output file/structure for easy review).
*   **LLM Choice & Routing:** The summarization and redaction will be handled by an advanced LLM (e.g. `gpt-4o`). Requests will be routed through Portkey to provide visibility into token usage, latency, and cost.
*   **Hybrid Logic (Code + AI):** Objective data validation (time math, missing fields) will be handled deterministically by Python. Semantic tasks (summarization, PII redaction, prompt injection detection) will be handled by the LLM.

## 2. Processing Pipeline

### Phase A: Deterministic Pre-processing (Python)
Before sending anything to the LLM, a Python script will evaluate the report for the strict data quality thresholds defined in the specification:
1.  **Completeness Check:** Verify that `resolution`, `arrived_at`, `departed_at`, and `stated_duration_hours` are present.
2.  **Contradiction Check:** Parse `arrived_at` and `departed_at` using Python's `datetime` module. Calculate the exact duration in hours and compare it to `stated_duration_hours`. 
3.  **Bypass/Flag:** If a report fails either check, the tool immediately outputs: *"This report is incomplete, we are following up."* and skips the LLM call entirely.

### Phase B: Semantic Processing (LLM)
If the report passes Phase A, it is passed to the LLM via an API call. The System Prompt will rigidly enforce the following rules:
1.  **Format Constraints:** Mandate the inclusion of the 6 required data points (asset, date, findings, actions, parts, time on site, recommendations).
2.  **Redaction Protocol:** Instruct the LLM to aggressively strip out all names, phone numbers, addresses, and physical security data (door/alarm codes, key locations).
3.  **Instruction Immunity:** Provide explicit prompt instructions to the LLM stating that the `technician_notes` field is untrusted input. If the text contains commands like "do not mention" or "hide this," the LLM must extract that attempt and print it verbatim in the summary.

## 3. Decisions & Rejected Alternatives

*   **Rejected Alternative 1: Pure LLM Processing.** Passing the raw JSON directly to the LLM and asking it to calculate the time differences to check for contradictions has been considered. 
    *   *Reason for rejection:* LLMs can hallucinate math or fail at exact time duration comparisons. Python's `datetime` module is 100% deterministic, faster, and cheaper.
*   **Rejected Alternative 2: Regex for PII/Security Redaction.** Using standard Regular Expressions to scrub phone numbers or 4-digit codes before sending the prompt to the LLM has been considered.
    *   *Reason for rejection:* Security instructions in free-text are semantic (e.g., "spare key is under the third flowerpot"). Regex cannot reliably catch this without massive false positives/negatives. The LLM is strictly better at semantic redaction.
*   **Decision on LLM Temperature:** The LLM temperature will be set strictly to `0.0`. We want highly deterministic, factual extraction and formatting without creative embellishment, minimizing the risk of the LLM "filling the gaps" on incomplete data.
