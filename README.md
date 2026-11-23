# FragilityAtlas

**FragilityAtlas: A global seismic fragility database with DeepSeek-assisted literature retrieval and PDF-based function extraction.** This English release combines a read-only browser client with an optional integrated local FastAPI/Uvicorn service. Fragility functions are the research objects; publications provide their provenance.

**Current release:** V1  
**Release scope:** English-language browser interface, published fragility database, and optional local Literature Agent and PDF-processing service.

## Key features and pages

- **Basic Information:** counts by country, ground-motion intensity measure (IM), and building type. Country counts can overlap where a function applies to more than one country.
- **Fragility Database:** filter source-paired functions by country, building class, IM, and damage state; inspect numerical function expressions, source-specific damage definitions, DOI, curve parameters, and calculated damage-state exceedance probability.
- **Japan Classic Curves:** filter verified Japan records, compare source-specific thresholds in small multiples, and export SVG curves. Original source labels and IM units are retained in the data exports.
- **Literature Agent Workbench:** DeepSeek-assisted literature retrieval, search-provider execution, URL deduplication, PDF download or local upload, PDF page localization, document-specific function extraction, candidate CSV/DOCX/JSON output, and human source verification.

## Architecture and data boundary

Double-clicking `out/index.html` now opens the three read-only data pages directly from disk. In `file://` mode the client loads `out/offline-data.js`, generated from the same public-safe JSON snapshot; filters, curve plots, and CSV/JSON/SVG exports work without a web server. The Literature Agent Workbench still requires the local Python service for search, PDF upload, extraction, and source verification.

`out/index.html` and its JavaScript/CSS files form the browser client. `server.py` serves these files and read-only database APIs on loopback through FastAPI/Uvicorn. `agent_workflow.py` runs local jobs. The `agent_tools/collector/` package uses a configured DeepSeek API for query generation and relevance assistance, then Serper or Kuntin for searching and accessible-PDF download. `agent_tools/fragility_pipeline.py` and local PDF tools locate pages and produce candidate function records; adapted papers have document-specific parsers. Unsupported or scanned layouts require candidate-page review and human verification. DeepSeek does not itself extract all PDF parameters.

The published database is the verified JSON snapshot in `out/curves.json` plus the three `out/curves-records-*.json` chunks. `out/pairing-ledger.json` preserves source-pairing evidence. Agent jobs write only to the local, ignored `agent_workspace/`; a candidate record never updates the published snapshot automatically. A new published snapshot requires a separate source-verification and controlled rebuild process, followed by independent review. Historical research build scripts are preserved in the Chinese source backup and are not part of this portable public runtime.

## Database scope and counting units

The supplied snapshot records **32 literature sources**, **2,018 applicability records**, **1,820 deduplicated curve records** (`Curve_ID`), and **585 curve groups** (`Group_ID`). The 1,820 curve records comprise **1,764 continuous probability functions** and **56 discrete exceedance-probability tables or damage probability matrices**, as classified by the source dataset. These are different units of count: one curve can have multiple country applicability records, and one building curve group can contain multiple damage thresholds. The counts describe this frozen literature-derived snapshot, not all published seismic fragility research worldwide. The data retain source-specific building classes, damage labels, modeling methods, and original bibliographic metadata. Flood and tsunami intensity measures are outside this release's earthquake-function scope.

## Requirements and installation

Use a Python installation compatible with the minimum package versions in `requirements.txt`. This release does not require Node.js to run. A modern browser is required for the client. No CUDA dependency is declared. The source does not specify a locked Python/package environment.

### Windows PowerShell

```powershell
py -3 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe server.py
```

If `py` is unavailable, use your installed `python` command. The included `start-workbench.cmd` runs `python server.py` after dependencies are installed.

### Linux/macOS

```sh
python3 -m venv .venv
.venv/bin/python -m pip install -r requirements.txt
.venv/bin/python server.py
```

Open **http://127.0.0.1:8876/**. The health endpoint is **http://127.0.0.1:8876/api/health**. The service accepts only local hosts and same-origin Agent operations. Do not expose it directly as a public network service.

## API and search-provider configuration

You may enter your own DeepSeek API key for an individual retrieval task in the Literature Agent Workbench. For local persistent configuration, copy `agent_tools/.env.example` to `agent_tools/.env` and fill in your own values. The `.env` file is ignored by Git and must remain private. The code also accepts a `FRAGILITY_AGENT_CONFIG` environment variable pointing to a local configuration file. Never commit credentials or browser profiles.

`COLLECTOR_SEARCH_ENGINE` may be `serper`, `kuntin`, or `both`. Serper requires `SERPER_API_KEY`; Kuntin requires a locally usable search service and may require user interaction/authentication in the browser. Search and model services are external and may have their own access rules and costs. Default DeepSeek endpoint/model values are in `agent_tools/collector/config.py`; verify service availability before use.

## Workflow

1. Browse and filter the published Fragility Database. Select **View curve** to inspect the function, parameters, original publication title, and DOI.
2. In the Literature Agent Workbench, enter country, building type, and IM. With valid service configuration, start DeepSeek-assisted literature retrieval. Inspect generated queries, deduplicated search results, download records, and the process report.
3. Upload an accessible local PDF or select a downloaded PDF. The local service locates relevant pages and calls the existing PDF-based function extraction pipeline. The result includes a 17-column parameter-candidate table, source-evidence DOCX, and validation JSON when produced.
4. Compare candidate parameters and source locations with the paper. Record human source-verification decisions. Candidate output remains separate from the published database.
5. A maintainer may prepare a new database version only through a separate, reviewed source-verification/rebuild workflow. This release intentionally offers no one-click publication endpoint.

Automated full-text retrieval primarily accesses publicly available/open-access files. Subscription publisher or institutional authentication is not integrated by this release. Some papers need document-specific parsing; scanned or image-only curves and unsupported layouts can require manual digitization or checking. A candidate page or numerical row is not proof of source verification.

## Static hosting, data files, and exports

For quick read-only use, open `out/index.html` directly in Edge or Chrome. To use the Literature Agent Workbench, start `server.py` and visit `http://127.0.0.1:8876/#agent` instead.

The `out/` directory can be served as a static site for **read-only browsing** of the verified JSON snapshot. A static host cannot run local Agent jobs, access local PDFs, or call the loopback Python endpoints. Use the local service for those operations.

The database table uses the existing field keys such as `Country`, `Building_Type`, `IM`, `Damage_State`, `Curve_ID`, and `Group_ID`. The UI exports database JSON, a function CSV, filtered Japan JSON, and SVG curves. Agent jobs can export search/download CSVs, parameter-candidate CSV, source-evidence DOCX, validation JSON, verification decisions JSON, and process reports where produced. Scientific JSON values, original publication titles, DOI strings, source-specific labels, and IDs are not translated in this release.

## Citation, license, and support

**Suggested citation:** FragilityAtlas, *A global seismic fragility database with DeepSeek-assisted literature retrieval and PDF-based function extraction*, version V1. Full authors, publication year, article DOI, and repository URL: **To be completed by the maintainer**. Cite the original publication associated with any reused fragility function.

**License:** The FragilityAtlas source code is distributed under the [MIT License](LICENSE). The license applies to the code; users should separately observe the copyright and access conditions of original publications and any third-party materials. The published database contains literature-derived records and source metadata; consult the corresponding original publications when reusing individual fragility functions.

**Support/contact:** For questions, bug reports, or technical support, please contact:

- **Zhaodong Wang**
- Email: [wzd9728@163.com](mailto:wzd9728@163.com)

Please include the FragilityAtlas version, operating system, and a concise description of the issue when reporting a problem.
