# Personal Agent Design Document: SEO Refresh Scout
**Intern:** Sathwik Peddi | **Track:** General AI Fluency (ML Focus) | **Assignment:** FL-06

---

## 🎯 1. Job to be Done & User Profile

### The Job to be Done (JTBD)
When a content editor or ML intern decides to review an organic search page for a refresh, they must manually cross-reference Google Search Console (GSC) performance, inspect top competitor layouts on search engine results pages (SERPs), identify content gaps, and draft updated sections. This manual workflow takes 1–2 hours per page.

**The SEO Refresh Scout** is a personal AI agent that automates this workflow:
1.  Accepts a target page URL and a primary keyword query.
2.  Fetches historical search visibility (impressions, avg position, CTR) for that page.
3.  Scrapes the top 3 ranking competitor pages for the primary query.
4.  Performs a semantic gap analysis (identifying missing subtopics, headings, and intent structures).
5.  Outputs a structured **Refresh Action Plan** containing recommended header expansions, metadata updates (Title/Meta Description), and internal linking targets.

### The User & Frequency
*   **User:** Sathwik Peddi (ML Intern / Content Operations Lead).
*   **Frequency:** Used 3–5 times per week during editorial backlog planning.
*   **Scope:** 10 build hours (achievable using Python scripting, Serper API, and an LLM API).

---

## 🛠️ 2. Tools, Data, & Access Plan

To perform its job, the agent requires access to search performance data and live SERP structures:

| Data/Tool | Purpose | Access Plan |
|---|---|---|
| **Google Search Console (GSC)** | Fetch historical impressions, average position, and clicks for the target URL. | Read from local CSV exports (starter playground layout) or connect via the official Google API Client Python Library using a service account JSON credential. |
| **Google Search / Serper API** | Retrieve the URLs of the top 3 ranking competitors for the target keyword. | Access via a free-tier API key from **Serper.dev** (2,500 free queries, fully sufficient for development). |
| **Web Scraper (HTTP/bs4)** | Scrape HTML headings and content from competitor URLs. | Implemented locally in Python using `requests` and `BeautifulSoup` with user-agent headers. |
| **Hugging Face / LLM API** | Perform semantic gap analysis and generate content recommendations. | Connect via the `google-genai` SDK or `openai` SDK using an API key stored securely in a local `.env` file. |

---

## 📋 3. Agent Instructions (System Prompt)

```text
You are the SEO Refresh Scout, a specialized AI agent designed to audit organic search pages and produce actionable refresh plans.

Your task is to analyze a target URL's historical performance, compare its content structure against top-ranking competitors, and identify critical semantic gaps.

Follow this execution loop:
1. Parse the input URL and keyword.
2. Retrieve performance trends: Highlight if the average position is in the "striking distance" (positions 4–20) or showing MoM decay.
3. Analyze competitor structures: Compare heading outlines (H2/H3) of the top 3 competitors. Identify key subtopics, user intent, or answers that the competitor covers but the target page lacks.
4. Generate a Refresh Action Plan.

Output format constraints:
- Must begin with a "Performance Context" section summarizing impressions and position.
- Must list exactly 3 "Semantic Gaps" with heading recommendations.
- Must provide updated SEO Meta Tag recommendations (Title under 60 chars, Meta Description under 160 chars).
- Do not make up facts; if a competitor's page cannot be scraped, note the access failure and skip.
```

---

## 🧪 4. Pre-Build Evaluation Cases (FL-03 Style)

To verify the agent's correctness before deploying, we will run it against 5 distinct evaluation cases:

### Case 1: Standard Striking Distance Page
*   **Input:** URL: `/blog/what-is-duckdb`, Keyword: `"duckdb guide"`, GSC Stats: `avg_position = 7.2`, `impressions_90d = 5,400`.
*   **Expected Behavior:** The agent should identify the position (7.2) as a high-opportunity striking-distance candidate, scrape the top 3 DuckDB guides, and note if they cover topics like "DuckDB vs Pandas" or "Parquet integration" that the target page lacks.
*   **Success Metric:** Action plan contains at least 2 specific subtopics missing from the target page.

### Case 2: Noisy Low-Volume Page
*   **Input:** URL: `/blog/my-test-post`, Keyword: `"ml internship tips"`, GSC Stats: `avg_position = 45.0`, `impressions_90d = 3`.
*   **Expected Behavior:** The agent should flag this as a low-volume page. It should recommend a structural rewrite rather than a minor semantic refresh because the page lacks base search visibility.
*   **Success Metric:** Outputs a warning note highlighting that impressions are below the volume threshold (threshold = 100).

### Case 3: Blocked Scraper (Robots.txt / Cloudflare)
*   **Input:** URL: `/blog/analytics-setup`, Keyword: `"google analytics 4 tutorial"`, Competitor 1: `https://medium.com/...` (returns 403 Forbidden).
*   **Expected Behavior:** The agent should handle the scrape failure gracefully, report the error, and base the semantic audit on the remaining accessible competitors.
*   **Success Metric:** Does not crash; outputs the report with a warning tag: `[Scrape Failure: Competitor 1]`.

### Case 4: Perfect Match (No Major Gaps)
*   **Input:** URL: `/blog/python-virtual-env`, Keyword: `"python virtual environment tutorial"`. Top competitors match the target page headings 1-to-1.
*   **Expected Behavior:** The agent should report that the semantic gap is minimal. It should shift recommendations toward freshness optimization (updating python version references) and improving CTR.
*   **Success Metric:** Recommends CTR/Freshness audit instead of heading changes.

### Case 5: Missing GSC Data
*   **Input:** URL: `/blog/new-post` (No GSC record found, new page).
*   **Expected Behavior:** The agent should recognize that GSC data is missing (new content), bypass the performance analysis section, and proceed purely with SERP competitor comparison.
*   **Success Metric:** Outputs report noting `GSC Data: [None / New Page]`.

---

## 🛡️ 5. Risks & Guardrails

To prevent unintended behavior, the agent will have the following strict guardrails:

1.  **Read-Only Operations:** The agent is strictly read-only. It has no capabilities to modify, overwrite, or delete local code files or live pages.
2.  **Scraping Rate-Limit:** Implement a minimum delay of 1.5 seconds between competitor scrape requests to respect website bandwidth and avoid triggering IP blocks.
3.  **Credential Protection:** Never log or print the GSC Service Account JSON keys or LLM API keys. All keys must be loaded at runtime from `.env` and immediately masked.
4.  **No Hallucinations:** If a competitor's page cannot be read, the agent must never invent or assume what that page covers.

---

## ⚖️ 6. Platform Choice & Justification

### Selected Platform: Scripted Python Agent
I chose to build this as a **Scripted Python Agent** running locally or via a terminal interface:
*   **How it works:** A Python script utilizing the `google-genai` SDK, `googleapiclient` (GSC API), `requests`/`BeautifulSoup` (scraping), and structured output formatting.

### Alternative Considered: Custom GPT (ChatGPT / Claude Project)
*   **Why it was rejected:** While a Custom GPT or Claude Project is faster to set up initially, it is highly restricted. It cannot easily connect to local CSV files (our GSC starter playground), cannot run automated scrapers dynamically across arbitrary search results without paid third-party web search plugins, and does not allow us to script the 5 pre-build evaluation cases for automated testing.
*   **Technical Rationale:** A scripted Python agent fits my background as an ML Engineer, operates completely on free local execution, and can be easily integrated into a terminal script or Jupyter notebook inside my existing internship codebase.
