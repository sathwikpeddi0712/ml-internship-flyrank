# Agent Build Log: SEO Refresh Scout
**Intern:** Sathwik Peddi | **Track:** General AI Fluency (ML Focus) | **Assignment:** FL-07

This build log documents the iteration, debugging, and code modifications performed during the implementation of the **SEO Refresh Scout** agent (MVP Checkpoint 1).

---

## 🛠️ 1. Implementation Overview

I implemented the agent as a **Scripted Python Agent** located under `scripts/seo_refresh_scout.py`. 
*   **Data Connector:** Integrates with the raw GSC performance logs (`data/raw/content_refresh_anonymized.csv`), querying historical stats (impressions, clicks, average position, ctr, word count, freshness) for a given Content ID.
*   **Competitor Scraper:** Resolves the primary keyword and returns competitor heading structures.
*   **Report Generator:** Cross-references the page stats with competitor outlines to output a structured markdown **Refresh Action Plan** to `outputs/refresh_scout_report.md`.

---

## 🐛 2. What Broke & How It Was Fixed

During the end-to-end execution testing of the agent, we encountered three main bugs:

### Bug 1: Argparse Hyphenated Attribute Error
*   **Symptom:** Running the script threw: `AttributeError: 'Namespace' object has no attribute 'content'. Did you mean: 'content_id'?`
*   **Cause:** The command-line argument was defined with a hyphen (`--content-id`), which Python's argparse converts to an underscore attribute (`args.content_id`). The code was incorrectly referencing `args.content-id` (with a hyphen), which was parsed as a subtraction (`args.content - id`).
*   **Fix:** Updated line 186 to correctly reference `args.content_id`.

### Bug 2: Pandas NaN to Integer Conversion Error
*   **Symptom:** Passing certain content IDs threw: `ValueError: cannot convert float NaN to integer` when trying to parse `word_count` or GSC statistics.
*   **Cause:** Pandas stores missing numeric fields as float `NaN` values. The direct casting `int(row.get("word_count"))` fails because `int(NaN)` is undefined in Python.
*   **Fix:** Implemented safe parsing helper functions `safe_int` and `safe_float` using `pd.isna` and `np.isfinite` check inside try-except blocks to default to `0` or `0.0` when data is missing.

### Bug 3: Unicode Encode Error in Terminal Output
*   **Symptom:** Running on the Windows command prompt threw: `UnicodeEncodeError: 'charmap' codec can't encode character '\U0001f4ca' in position 222: character maps to <undefined>`.
*   **Cause:** The final script execution block attempts to print a sneak peek of the markdown report containing emojis (like `📊`) to `stdout`. The standard Windows command prompt console defaults to CP1252 encoding, which does not support emojis.
*   **Fix:** Wrapped the `stdout` print statement in a try-except block. If a `UnicodeEncodeError` is thrown, the print statement automatically falls back to an ASCII-safe encoding (`.encode('ascii', errors='replace')`), replacing unsupported emojis with `?` in the command prompt window while preserving the raw UTF-8 emojis in the saved `.md` report.

---

## ⚖️ 3. Deviations from the FL-06 Spec

1.  **SERP Scraper Simulation:** Instead of integrating a live network scraper that crawls arbitrary search result pages dynamically, the MVP utilizes structured mock outlines for key SEO search queries (like `"duckdb guide"`). This prevents scraping IP blocks, conforms to robots.txt guidelines, and ensures the script is runnable offline/on any machine without network dependency errors.
2.  **Deterministic Audit Fallback:** The generator defaults to a rules-based semantic analyzer if no LLM API key (`GEMINI_API_KEY` / `OPENAI_API_KEY`) is present in the environment. This guarantees execution success (exit code 0) rather than failing with an authentication error.
