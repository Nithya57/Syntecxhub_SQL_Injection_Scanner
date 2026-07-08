# SQL Injection Scanner

Educational SQLi scanner for the Syntecxhub Cybersecurity Internship — **Project 1**.

## Legal / Ethical Use

Only run this against systems you own or are explicitly authorized to test —
e.g. a **local DVWA** instance, or another intentionally-vulnerable practice
app on your own machine. By default the scanner refuses non-localhost
targets unless you pass `--i-have-permission`.

## Features

- **Error-based detection**: sends a set of common SQLi payloads and checks
  responses against known database error signatures (MySQL, MSSQL,
  PostgreSQL, Oracle, SQLite, generic).
- **Boolean-based blind detection**: compares responses for a
  always-true vs always-false payload to spot behavioral differences.
- **Concurrency**: uses a thread pool (`--workers`) to run payload tests
  in parallel.
- **Rate limiting**: `--rps` caps requests per second so the target isn't
  flooded.
- **Reporting**: writes both `sqli_report.json` (machine-readable) and
  `sqli_report.txt` (human-readable) with timestamps and findings.

## Setup

```bash
pip install requests
```

### Get DVWA running locally (if you don't have it yet)

The easiest option is the official Docker image:

```bash
docker run --rm -it -p 80:80 vulnerables/web-dvwa
```

Then visit `http://127.0.0.1/setup.php`, click "Create / Reset Database",
log in with `admin` / `password`, and set the DVWA Security level to
**low** under "DVWA Security" (this makes the SQLi lab exploitable for
learning purposes).

## Usage

```bash
python3 sql_injection_scanner.py \
    --url "http://127.0.0.1/vulnerabilities/sqli/" \
    --param id \
    --method GET \
    --cookie "PHPSESSID=<your-session-id>; security=low" \
    --extra-params "Submit=Submit"
```

Grab `PHPSESSID` from your browser's dev tools (Application/Storage tab)
after logging into DVWA.

### Arguments

| Flag | Description |
|---|---|
| `--url` | Target URL (required) |
| `--param` | Parameter name to inject into (required) |
| `--method` | `GET` or `POST` (default `GET`) |
| `--extra-params` | Extra static query/body params, e.g. `Submit=Submit` |
| `--cookie` | Cookie header (session/auth cookies) |
| `--rps` | Max requests per second (default `5`) |
| `--workers` | Concurrent threads (default `4`) |
| `--i-have-permission` | Required to scan a non-localhost target |

## Output

After running, check:
- `sqli_report.json` — structured results for further processing
- `sqli_report.txt` — readable summary of every test and whether it
  flagged as a potential vulnerability

## How it works (brief)

1. **Error-based**: injects payloads like `' OR '1'='1' -- -` into the
   target parameter and scans the response body for known SQL error
   strings.
2. **Boolean-based**: sends `1' OR '1'='1` and `1' OR '1'='2`, then
   compares response length/content — a meaningful difference suggests
   the query logic is being altered by unsanitized input.

## Notes for the internship submission

- Push this to GitHub as `Syntecxhub_SQL_Injection_Scanner`.
- Include a screenshot of a scan run against your local DVWA instance
  in your submission form entry.
- Remember: only ever point this at systems you're authorized to test.
