# FrontierAtlas

**A source-traceable ingestion pipeline and dashboard for the AI and venture ecosystem.**

FrontierAtlas collects startups, products, research papers, jobs and news, and stores every record together with the real source URL and the evidence it was extracted from. No placeholder data is used. Every field can be checked against its origin.

---

## ⚠️ Important: Download the ZIP file

This repository exceeded GitHub's file-size limit, so the full project could not be pushed as individual files. The complete source code is provided as a **ZIP archive** instead.

**Please follow these steps:**

1. Download the ZIP file from this repository (see the file list above, or the **Releases** page if it is attached there).
2. Extract the ZIP to a folder on your computer.
3. Open the extracted folder and follow the [Installation](#installation) and [Usage](#usage) sections below.

> Do not try to run the project directly from the ZIP file. It must be extracted first.

---

## Table of Contents

1. [Overview](#overview)
2. [Key Features](#key-features)
3. [How the Pipeline Works](#how-the-pipeline-works)
4. [Dashboard Pages](#dashboard-pages)
5. [Results of the Demonstration Run](#results-of-the-demonstration-run)
6. [Requirements](#requirements)
7. [Installation](#installation)
8. [Configuration](#configuration)
9. [Usage](#usage)
10. [Project Structure](#project-structure)
11. [Outputs](#outputs)
12. [Documentation](#documentation)
13. [Author](#author)
14. [License](#license)

---

## Overview

FrontierAtlas is built around three ideas:

- **Provenance first.** Every record links back to a real source URL and keeps the evidence it came from.
- **Rules before AI.** Fields are extracted with deterministic rules first. A language model is used only as a fallback, and its output is rejected unless it can be backed by a verbatim quote from the page.
- **Resilience.** The fetching and extraction layers are designed to survive rate limits, blocked requests and oversized pages, and this is demonstrated live in Chaos Mode.

## Key Features

| Feature | Description |
|---|---|
| Multi-source ingestion | Startups, products, research papers, jobs and news collected from real sources |
| Anti-bot fetch ladder | Requests escalate through several tiers, with backoff and fallback when a tier fails |
| Rules-first extraction | Deterministic extraction is attempted before any LLM call |
| LLM fallback chain | Gemini, then Groq, then DeepSeek |
| Anti-hallucination guard | LLM output must be supported by a verbatim quote from the page, otherwise it is discarded |
| Chaos Mode | Runs the real engine against a mock server that fails on purpose (HTTP 429 and 413) |
| Freshness enforcement | Jobs and news are limited to the last 24 hours, and each row records how its date was determined |
| Entity resolution | Messy names are resolved deterministically to a canonical entity, with a log of every decision |
| Knowledge graph | Startups, products, papers, jobs and news are linked through canonical entities |
| Data and Evidence viewer | Search every record and inspect its source URL, verbatim evidence and raw record |

## How the Pipeline Works

```
Discover  →  Fetch  →  Extract  →  Resolve  →  Store
```

1. **Discover**: finds the sources to ingest (for example, the Y Combinator directory).
2. **Fetch**: retrieves pages through an anti-bot ladder of tiers, escalating when a request is blocked or rate-limited.
3. **Extract**: pulls fields out of each page using rules first. If rules are not enough, an LLM is used in this order: Gemini, Groq, DeepSeek. Any LLM answer must be backed by a verbatim quote from the page, otherwise it is dropped.
4. **Resolve**: maps name variants to one canonical entity (for example, "Open AI" becomes "OpenAI").
5. **Store**: writes records, with source URL and evidence, to the output folder and the shared Google Sheet.

### Chaos Mode

Chaos Mode runs the real engine against a mock server that fails on purpose:

- One tier is permanently rate-limited and returns **HTTP 429**.
- Another tier rejects large pages with **HTTP 413**.

The engine backs off, falls back to the next tier, shrinks the document and still returns the correct answer. The log shows each step.

## Dashboard Pages

| Page | Purpose |
|---|---|
| **Mission Control** | Run summary, headline counts, data-trust indicators, pipeline, fetch layer and LLM engine status |
| **Data & Evidence** | Search and browse all records by tab. Click a row to open a drawer with the source URL, verbatim evidence and raw record |
| **Freshness Radar** | 24-hour timeline showing that only recent jobs and news are kept, coloured by how each date was determined |
| **Knowledge Graph** | Interactive graph linking startups, products, papers, jobs and news through canonical entities |
| **Entity Lab** | Type a messy name and watch the deterministic cascade resolve it against the seed entity list |

## Results of the Demonstration Run

| Dataset | Count |
|---|---|
| Startups (YC directory) | 999 |
| Products (with pricing evidence) | 1,005 |
| Research papers (with live GitHub star counts for nearly all) | 1,024 |
| Jobs (last 24 hours) | 5 |
| News (last 24 hours) | 6 |

- Jobs and news are intentionally small because they include only items published in the last 24 hours.
- On real data, **219 non-seed organisations were kept separate and none were wrongly merged.**

*These figures are from one run and will vary with each run.*

## Requirements

> Replace the items below with the exact versions your project uses.

- Python `<version, for example 3.10 or newer>`
- `pip` (Python package manager)
- A modern web browser (Chrome, Edge or Firefox)
- API keys for the LLM providers you want to use (Gemini, Groq, DeepSeek). The keys are optional if you only use the rules-based extraction.
- Internet access for live ingestion

## Installation

1. **Download and extract the ZIP file** (see the note at the top of this document).

2. **Open a terminal** in the extracted project folder:

   ```bash
   cd <extracted-folder-name>
   ```

3. **Create and activate a virtual environment** (recommended):

   ```bash
   python -m venv venv

   # Windows
   venv\Scripts\activate

   # macOS / Linux
   source venv/bin/activate
   ```

4. **Install the dependencies:**

   ```bash
   pip install -r requirements.txt
   ```

## Configuration

Create a `.env` file in the project root (or edit the provided example file) and add your API keys:

```env
GEMINI_API_KEY=your_key_here
GROQ_API_KEY=your_key_here
DEEPSEEK_API_KEY=your_key_here
```

> Use the exact variable names that your code reads. Never commit real API keys to a public repository.

## Usage

> Replace the commands below with the actual entry points of your project.

**1. Run the ingestion pipeline**

```bash
python <pipeline_entry_file>.py
```

This writes the collected records to the `output/` folder.

**2. Start the dashboard**

```bash
python <dashboard_entry_file>.py
```

Then open the dashboard in your browser:

```
http://localhost:8001
```

The dashboard reads its data from the `output/` folder, so run the pipeline first (or use the output included in the ZIP).

**3. Try Chaos Mode**

Open **Mission Control**, scroll to the Chaos Mode card and click **Inject failures** (or **Run Chaos Mode again**). Watch the log as the engine backs off, falls back and recovers.

**4. Try Entity Lab**

Open **Entity Lab** and type a messy name such as `Open AI`. The result shows the canonical name and which resolution step matched.

## Project Structure

> Update this tree to match your actual folders.

```
FrontierAtlas/
├── output/              # Generated records and results
├── <pipeline code>/     # Discovery, fetch, extraction and resolution
├── <dashboard code>/    # Dashboard (Mission Control, Data & Evidence, etc.)
├── requirements.txt     # Python dependencies
├── .env                 # API keys (not shared)
└── README.md
```

## Outputs

- **Local output folder:** all records with their source URLs and evidence.
- **Shared Google Sheet:** the results in a sheet with six tabs.
- **Dashboard:** a browsable view of the same data.

Google Sheet link: `<add link here>`

## Documentation

- **Architecture:** described in the accompanying PDF (`<add file name or link here>`).
- **Demo video:** `<add link here>`

## Author

**Surya**

- GitHub: `<add profile link>`
- Contact: `<add email or LinkedIn>`

## License

`<Add your license here, for example MIT. If no license is added, all rights are reserved by the author.>`

---

*If you find an issue or have a question, please open an issue in this repository.*
