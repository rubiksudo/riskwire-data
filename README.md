# RiskWire Data

**Daily archive of curated CVE-exploitation intelligence.**

Published to the public domain (CC0). Snapshots come from [riskwire.io](https://riskwire.io) — a weekly-refresh feed that filters ~70,000+ cyber-press articles per week down to the CVEs with real exploitation signal: proof-of-concept availability, in-the-wild attacks, or breaches attributed to named threat actors. Every event is cross-referenced against [CISA's Known Exploited Vulnerabilities (KEV) catalog](https://www.cisa.gov/known-exploited-vulnerabilities-catalog) with federal remediation deadlines.

**Full live feed:** [riskwire.io](https://riskwire.io)

---

## What's in this repo

A date-stamped JSON snapshot of the RiskWire pipeline output, one file per day:

    data/
      2026-08-20.json
      2026-08-21.json
      2026-08-22.json
      ...

Each snapshot contains the events tracked in a rolling window, with structured fields for CVE identifiers, severity (CVSS), vendor/product, PoC availability, victim organization, named threat actors, event date, CISA KEV status, federal remediation deadline, and source citations.

## Schema

Every event has these top-level fields:

| Field | Type | Description |
|---|---|---|
| `record_id` | string | Stable ID for the event across days |
| `record_title` | string | Human-readable event title |
| `added_on` | ISO-8601 | When RiskWire first ingested the event |
| `updated_on` | ISO-8601 | Last time the event was modified |
| `enrichment.main_cve` | string | Primary CVE identifier (`CVE-YYYY-NNNN`) |
| `enrichment.all_cve_codes` | string | Comma-separated list of every CVE mentioned |
| `enrichment.severity` | string | CVSS severity + score (e.g. `Critical (9.8)`) |
| `enrichment.vendor_or_product` | string | Vendor and/or product affected |
| `enrichment.poc_available` | enum | `Yes` / `No` / `Unknown` |
| `enrichment.victim_organization` | object | Named victim, if identified |
| `enrichment.threat_actors` | string | Named threat actors / APT groups |
| `enrichment.event_date` | ISO date | When the exploit / breach / PoC actually occurred |
| `enrichment.enrichment_confidence` | enum | LLM extraction confidence |
| `citations` | array | Source articles, each with `title`, `link`, `source`, `published_date` |
| `connected_entities` | array | Vendor/watchlist matches, if applicable |

CISA KEV cross-references are computed live at render time in the RiskWire dashboard, not stored in the snapshot — reproduce them with the [official KEV feed](https://www.cisa.gov/sites/default/files/feeds/known_exploited_vulnerabilities.json).

## Example usage

**Python:** Load the latest snapshot and list every Critical event:

    import json, urllib.request, datetime

    today = datetime.date.today().isoformat()
    url = f"https://raw.githubusercontent.com/rubiksudo/riskwire-data/main/data/{today}.json"
    data = json.loads(urllib.request.urlopen(url).read())

    for record in data["all_records"]:
        e = record.get("enrichment", {}) or {}
        if not e.get("main_cve"):
            continue
        sev = str(e.get("severity", ""))
        if sev.lower().startswith("critical"):
            print(e["main_cve"], "-", e.get("vendor_or_product", "?"), "-", record.get("record_title", ""))

**Load the CISA KEV catalog and enrich locally:**

    kev_url = "https://www.cisa.gov/sites/default/files/feeds/known_exploited_vulnerabilities.json"
    kev = {v["cveID"]: v for v in json.loads(urllib.request.urlopen(kev_url).read())["vulnerabilities"]}

    for r in data["all_records"]:
        cve = (r.get("enrichment") or {}).get("main_cve")
        if cve and cve in kev:
            due = kev[cve].get("dueDate")
            print(f"KEV ✓ {cve} — federal due date {due}")

## What this repo is NOT

- **Not a real-time feed.** Snapshots are daily. For hourly updates use the paid tier at [riskwire.io](https://riskwire.io).
- **Not a replacement for CISA KEV.** RiskWire's added value is the news-driven layer (threat actors, victims, PoC availability, context) on top of CISA's authoritative catalog. Always source federal remediation deadlines from CISA directly.
- **Not a vulnerability database.** For canonical CVE detail use [NVD](https://nvd.nist.gov/) or [MITRE CVE](https://www.cve.org/).

## License

Released under [Creative Commons Zero v1.0 Universal](./LICENSE) — public domain. No attribution required, though a link back to [riskwire.io](https://riskwire.io) is appreciated.

## Contact

- **Live product & paid tiers:** [riskwire.io](https://riskwire.io)
- **General:** hello@riskwire.io
- **Issues with the data:** open an issue in this repo

---

*Curated by [RiskWire](https://riskwire.io) — the weekly exploited-CVE feed with CISA KEV overlay.*
