# D76 CyberOps

D76 CyberOps is a local-first toolkit for authorized security research, analysis and administration. It runs on your own machine as a small Flask app with a SQLite database. No tool in it produces simulated results: when something cannot run (missing dependency, no permission, no network), the interface says so.

All 38 tools in the original specification are installed. This package is **phase 4**, the last, and includes everything from phases 1 to 3.

| Phase | Tools |
|---|---|
| 1 | Intelligence Hub, DevKit, Password Auditor, FileHash, Settings, Activity log, Global search |
| 2 | CaseVault, OSINT Explorer, ReconNote, CodeVault, Knowledge Base, Operations Board |
| 3 | HeaderLab, JWT Inspector, API Workbench, OpenAPI Auditor, Subnet Calculator, CipherBox, OTP Lab, Certificate Inspector, Redactor, Metadata Inspector, File Inspector, Mail Header Analyzer, IOC Extractor, Timestamp Decoder, Host Diagnostics, Admin Toolbox, Text Pipelines, DataLens, Docs Studio |
| 4 (this build) | Domain Inspector, Threat Intel, Security Checklist, ConfigAudit, LogAnalyzer, Terminal, Nova CLI, BackupGuard, NetWatch, ReportGen |

Press **Ctrl K** (or use "Jump to tool") anywhere to open a tool by name. **All tools** lists every tool with a filter. Sidebar groups remember whether you left them open.

## Requirements

- Python 3.9 or newer
- Flask (the only dependency; see `requirements.txt`)

## Install and run

```bash
python3 -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt
python app.py
```

Open http://127.0.0.1:5076.

## Configuration

All optional, set as environment variables.

| Variable | Default | Purpose |
|---|---|---|
| `D76_HOST` | `127.0.0.1` | Interface to bind. Keep it on loopback unless you know why you are changing it. |
| `D76_PORT` | `5076` | Port. |
| `D76_DATA` | `./data` | Folder holding the SQLite database. |
| `D76_SECRET` | random per start | Session signing key. Set it if you want sessions to survive restarts. |
| `D76_ALLOWED_HOSTS` | none | Extra hostnames to accept (comma separated, ports ignored, `*.example.com` wildcards allowed), for example your domain or tunnel hostname. localhost, 127.0.0.1, 0.0.0.0, any IP address (LAN or server IP) and this machine's own hostname are always accepted. Any other hostname is rejected to block DNS-rebinding. |
| `D76_TRUST_PROXY` | off | Set to `1` behind a reverse proxy that sets `X-Forwarded-Host/Proto/For`, so HTTPS and the public hostname are detected correctly. |

## Architecture

```
d76cyberops/
  app.py          routes, JSON API, CSRF and host checks, tool registry
  db.py           SQLite schema and helpers
  templates/      Jinja pages (base layout, hub, one template per tool)
  static/         app.css and one small script per page
  data/           created on first run (d76.db)
```

- Server-rendered pages with plain JavaScript. No frontend framework, no CDN scripts or fonts.
- All state-changing requests need a per-session CSRF token sent in the `X-CSRF-Token` header.
- A Content-Security-Policy limits scripts and styles to the app itself. The only external resource is the D76 logo image.
- The schema for later phases (cases, notes, snippets, tasks, articles, reports) is already created so global search and export cover them as tools arrive.

## Database

`data/d76.db` (SQLite). Tables: `settings`, `activity`, `cases`, `notes`, `evidence`, `entities`, `relations`, `snippets`, `projects`, `tasks`, `task_history`, `kb_articles`, `recon`, `reports`. Databases created by phase 1 are upgraded automatically on start. Use Settings > Export to get a JSON copy, and Import to merge one back. Passwords are never stored. The activity log keeps the latest 2,000 events and records only tool names and event types.

## What is installed in phase 2

- **Intelligence Hub**: tool index, saved case count, recent activity, system status, quick actions, reference links.
- **DevKit**: JSON format/minify/validate, Base64, URL and HTML entity encoding, SHA-1/256/384/512 text hashes, UUID v4 and secure random IDs (rejection sampling, no modulo bias), timestamp converter, regex tester, line diff, color converter. All client-side.
- **Password Auditor**: length, character types, repetition, sequences and keyboard walks, common-password and leetspeak checks, entropy estimate. Runs in the browser; the password is never sent anywhere, saved or logged.
- **FileHash**: SHA-256, SHA-512, SHA-1 and MD5 for one or many files, comparison against a published checksum, JSON and CSV export. Files are streamed to your local server, hashed in 1 MB chunks and discarded. Nothing is written to disk or sent elsewhere. Upload limit is 1 GB per request.
- **CaseVault**: cases with status, investigator and tags; timestamped notes; evidence records with SHA-256. Choosing a file hashes it on your local server and discards it (nothing is stored or modified); you can also record a known hash. Verify later by selecting the file again: the result (match or mismatch) and the time are recorded. Export a case as Markdown, HTML or JSON.
- **OSINT Explorer**: research cases holding entities (person, organization, domain, website, username, URL, note) with source URL, evidence reference, tags, bookmarks, and relationships between entities, shown as a map. Filter by type, tag, text or bookmark; export CSV, Markdown, HTML or JSON. It never looks anything up for you; every entry is yours.
- **ReconNote**: structured notes (target, scope, date, authorization reference, observations, URLs, DNS findings, technology observations, file references). Scope and authorization are required. Markdown and JSON export.
- **CodeVault**: snippets with language (auto-detected or chosen), category, tags, favorites, search, copy, syntax highlighting for HTML, CSS, JavaScript, Python, JSON, SQL and Bash (Markdown and text are shown plain), JSON import and JSON/Markdown export. Snippets are never executed.
- **Knowledge Base**: 11 starter articles across 10 categories, Markdown rendering, search, bookmarks, read status, and your own articles. Starter articles are added once; deleting them does not bring them back (a full reset does).
- **Operations Board**: projects with a four-stage board (Backlog, In progress, Review, Completed), priorities, deadlines with overdue marking, tags, notes, progress, and a history of every task change.
- **Settings**: theme, developer mode, storage info, export/import, reset (requires typing RESET).
- **Activity** and **Search**: local history and search across cases, notes, snippets, articles, tasks and reports.

## What is installed in phase 3

Web security
- **HeaderLab**: paste response headers, a Content-Security-Policy or Set-Cookie lines. Gives a graded review with plain explanations. Nothing is fetched.
- **JWT Inspector**: decodes tokens, flags alg none, missing expiry, risky header fields, and verifies HS256/384/512 signatures against a secret you supply. It does not guess secrets.
- **API Workbench**: send requests to APIs you are authorized to test. Variables (`{{name}}`) live only in the page. Saved requests replace Authorization, Cookie, API-key headers and key-like URL parameters with placeholders; check bodies yourself. Outbound requests block private and local addresses unless you tick the local or lab option, connect to the address that was validated, and cap size and time.
- **OpenAPI Auditor**: static review of an OpenAPI 3 or Swagger 2 document (JSON only).

Cryptography
- **CipherBox**: AES-256-GCM with a PBKDF2-SHA256 key (600,000 iterations by default), in the browser. The header is authenticated; a wrong passphrase or any change is detected. There is no recovery if the passphrase is lost.
- **OTP Lab**: TOTP and HOTP, checked against the RFC 4226 and RFC 6238 test vectors. Verify codes with a drift window; read otpauth links.
- **Certificate Inspector**: decodes PEM certificates, CSRs and public keys with a built-in DER reader; private keys are detected but never decoded.

Privacy, forensics and network
- **Redactor**: masks emails, Malaysian IC numbers (date validated), payment cards (Luhn checked), phone numbers, IPv4 addresses and secrets; strips tracking parameters from links. Pattern based, so review the result.
- **Metadata Inspector**: EXIF (including GPS), XMP, PNG text chunks, PDF properties and Office or OpenDocument properties. JPEG and PNG can be saved as cleaned copies.
- **File Inspector**: real file type from signatures, entropy chart, ASCII and UTF-16 strings, hex view, PE section table and ELF headers. Files are read as bytes and never executed.
- **Mail Header Analyzer**: route and delays, SPF/DKIM/DMARC results, From, Reply-To and Return-Path mismatches. Nothing is looked up online.
- **IOC Extractor**: IPs, domains, URLs, emails, hashes, CVE and ATT&CK IDs, with refanging and defanged output.
- **Timestamp Decoder**: FILETIME, WebKit, Mac, HFS+, Excel, .NET and Unix formats side by side.
- **Subnet Calculator**: details, split, summarize, range to CIDR and membership for IPv4 and IPv6.

System, data and documents
- **Host Diagnostics**: CPU, memory, disks, largest processes and listening TCP ports on the machine running D76. CPU, memory, process and port views need Linux `/proc`; other systems show what is available and say what is not.
- **Admin Toolbox**: cron explainer with next run times (standard day-of-month/day-of-week rule), and chmod, symbolic and umask conversion.
- **Text Pipelines**: chained line transformations, saved to the database.
- **DataLens**: profiles CSV, TSV and JSON arrays (types, gaps, duplicates, possible personal data) and converts between CSV and JSON.
- **Docs Studio**: Markdown editor with templates, export to .md or .html, save to the Knowledge Base, and an RFC 9116 security.txt builder.

## What is installed in phase 4

- **Domain Inspector**: DNS (A, AAAA, MX, NS, TXT, CNAME, SOA, CAA) through a built-in resolver client, TLS handshake and certificate details, the redirect chain with timings, security headers, and optional RDAP registration data. Needs an explicit authorization tick; private and local targets need a second tick. A failed lookup is reported as failed. Results can be exported or saved for reports.
- **Threat Intel**: CISA Known Exploited Vulnerabilities, CISA advisories, NVD CVEs, Microsoft and Ubuntu advisories out of the box, plus feeds you add (https only). Items are cached in the database with a 15 minute refresh window, filtered by severity, exploited status and bookmark, and exported as JSON or CSV. Severity appears only where a source supplies it. An NVD key, if you have one, goes in `D76_NVD_API_KEY`; it is never stored or shown.
- **Security Checklist**: 50 items across 12 areas. Automatic checks fetch the home page of a site you are authorized to test (HTTP to HTTPS redirect, certificate, security headers, cookie flags, version banners) and fill in what they can; you review them. Reports export as Markdown, HTML, PDF or JSON.
- **ConfigAudit**: finds known credential formats, credentials inside URLs, plaintext secrets, weak defaults, debug mode, disabled certificate checks, wildcard origins and similar. Secret values are never returned, stored or logged, including in error messages.
- **LogAnalyzer**: web access logs, syslog, JSON lines and timestamped text. Reports counts, time span, busiest addresses, status codes, a timeline, and potential anomalies with the lines behind them.
- **Terminal**: runs allowlisted commands directly (no shell), streams output, shows exit codes, enforces a time and output limit, and runs with a scrubbed environment. Pipes between allowed commands work; redirects, chaining and background jobs do not. It is off until you turn it on, works only when D76 is opened at localhost (not through a tunnel or proxy), and can be disabled for good with `D76_TERMINAL=0`. The policy is editable in the page, and it cannot be changed by importing data.
- **Nova CLI**: explain code, write documentation and tests, analyze errors, summarize logs, review code for security problems, explain concepts and format code, with Anthropic or any OpenAI-compatible server (including local ones such as Ollama). Secrets are masked before sending. Keys come from `D76_NOVA_API_KEY`, `ANTHROPIC_API_KEY`, `OPENAI_API_KEY` or a file named by `D76_NOVA_KEY_FILE`, never from the database or the browser. Also usable from the command line: `python nova.py status`, `python nova.py explain file.py`, `python nova.py scaffold "a flask api" --out ./new-project`. Scaffolding validates every path and will not overwrite files without `--force`.
- **BackupGuard**: incremental backups with a SHA-256 for every file, change preview, history, verify, restore, and delete. Originals are only read. Restoring never deletes anything; overwriting files needs you to type OVERWRITE, deleting a backup needs DELETE. A backup that later backups depend on cannot be deleted. Links and special files are skipped. Works only at localhost, like Terminal.
- **NetWatch**: TCP, HTTP(S), DNS and ping checks against targets you add, with latency, loss, history, and alerts for repeated failure or slowness. A background monitor runs on each target's schedule while D76 is running. Also shows interface counters, gateway, resolvers and TCP states on this machine (Linux).
- **ReportGen**: builds a report from cases, research cases, checklists, saved inspections, log analyses and file hashes, and the NetWatch summary, with scope, methodology, findings, evidence references, notes, conclusion and a disclaimer. Downloads as PDF, HTML, Markdown or JSON. Everything in the HTML report is escaped. The PDF is written by D76 itself and uses standard fonts, so characters outside Western European text appear as question marks.

Environment variables added in phase 4: `D76_DNS_SERVERS` (resolvers to use if the system has none, for example on Termux), `D76_TERMINAL` (set to 0 to disable the terminal), `D76_NVD_API_KEY`, `D76_NOVA_PROVIDER`, `D76_NOVA_MODEL`, `D76_NOVA_BASE_URL`, `D76_NOVA_MAX_TOKENS`, `D76_NOVA_API_KEY`, `D76_NOVA_KEY_FILE`.

## External APIs and permissions

No tool needs an API key except Nova (and an NVD key is optional). Outbound requests happen only when you press a button: API Workbench, Domain Inspector, the checklist's automatic checks, feed refresh, NetWatch checks and Nova (apart from your browser loading the logo and any reference link you click). Threat Intel, Domain Inspector and the Security Checklist's automatic checks need internet access. Nova CLI needs a provider key (or a local model server). Terminal, BackupGuard and NetWatch act on your own machine with your own user permissions.

## Troubleshooting

- **"Host not allowed"**: you opened the app through a hostname other than localhost. Add it to `D76_ALLOWED_HOSTS`.
- **"Missing or invalid request token"**: the page is stale (for example after a server restart). Reload it.
- **Copy button does nothing**: browsers only allow clipboard access on localhost or HTTPS.
- **DevKit hashes say Web Crypto is unavailable**: same cause, open the app over localhost or HTTPS.
- **Port in use**: set `D76_PORT`.
- Turn on developer mode in Settings to see error details in the interface; the full traceback is always printed in the terminal.

## Security notes

- Use these tools only on systems you own or are authorized to assess.
- Bind to loopback. If you expose the app through a tunnel, put authentication in front of it first; the app itself has no login because it is designed for a single local user.
- Terminal and BackupGuard refuse requests that arrive through a proxy, tunnel or non-local hostname, even if you add that hostname to `D76_ALLOWED_HOSTS`. Do not work around this.
