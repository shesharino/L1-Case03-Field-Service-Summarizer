# Implementation Tasks: Field Service Report Summarizer

## Task 1: Project Setup & Scaffolding
* Initialize the Python environment and install necessary dependencies (e.g., `openai`, `portkey-ai`, `python-dotenv`).
* Create the main Python script (`summarizer.py`).
* Write a function to load and parse the 20 reports from `data/service_reports.jsonl`.
* Set up a basic loop to iterate through the reports and print the raw `report_id` to verify ingestion.

## Task 2: Deterministic Validation (Phase A)
* Implement a validation function to check for the presence of the `resolution` and `stated_duration_hours` fields.
* Implement a time-parsing function using Python's `datetime` module to calculate the difference between `arrived_at` and `departed_at`.
* Compare the calculated difference against `stated_duration_hours` (allow for minor floating-point tolerance, e.g., 0.01 hours, to account for standard decimal conversion).
* Wire the validation function into the main loop: if a report fails either check, immediately print the exact fallback message: `"This report is incomplete, we are following up."` and skip further processing for that report.

## Task 3: LLM Integration & Core Formatting (Phase B - Part 1)
* Set up the LLM client (via Portkey for tracing/cost visibility) with temperature set to `0.0`.
* Draft the initial System Prompt. It must instruct the LLM to extract and format the 6 required data points into a plain-language summary for a facilities contact:
  1. Asset and visit date.
  2. Findings.
  3. Actions taken.
  4. Parts fitted.
  5. Outstanding items/recommendations.
  6. Time on site.
* Feed the passing reports to the LLM and print the output.

## Task 4: Security and Privacy Redaction (Phase B - Part 2)
* Update the System Prompt with strict redaction rules.
* Explicitly command the LLM to omit:
  * Internal identifiers (except asset IDs).
  * Engineer names and technician IDs.
  * Personal contact details (emails, phone numbers, names).
  * Physical security data (door codes, alarm codes, key locations).
* Run the script and specifically check reports known to contain security risks (e.g., `FSR-3003`, `FSR-3014`) to ensure nothing leaks.

## Task 5: Instruction Immunity & Prompt Injection Handling (Phase B - Part 3)
* Add a final layer to the System Prompt addressing the `technician_notes`.
* Instruct the LLM that these notes are untrusted description, not instructions.
* Command the LLM to explicitly quote/surface any attempts within the text to hide information or dictate the tool's behavior.
* Run the script and specifically verify the output for `FSR-3009` (which contains an instruction to hide a pressure test failure).

## Task 6: Comprehensive Review and Refinement
* Run the completed script across all 20 reports.
* Read every output against the requirements in `summary_requirements.md` and `criteria_checklist.md`.
* Conduct the required 5-lens review on the generated AI code (intent, tests, security, performance, maintainability) and document the findings and corrections.
