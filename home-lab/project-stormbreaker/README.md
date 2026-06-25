# Project Stormbreaker

A multi-source IOC threat-intelligence and threat-hunting platform for the
terminal. Give it any indicator of compromise (IP, domain, URL, file hash,
email, or phone number) and it:

1. **auto-detects** the indicator type,
2. queries the relevant intelligence/OSINT sources **concurrently**,
3. maps findings to **MITRE ATT&CK**,
4. generates **threat-hunting hypotheses** and an **investigation workflow**,
5. engineers **Wazuh + Sigma detection rules**,
6. profiles possible **threat-actor** associations,
7. and emits a **colour-coded terminal report**, a **professional PDF**, an
   **ATT&CK Navigator layer**, and **raw JSON logs**.

## Install

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

## API keys

Keys are read from environment variables and never hard-coded. Set whichever you
have; **any source without a key is skipped** and the rest still run.

```bash
export VT_API_KEY="..."              # virustotal.com (free)
export ABUSEIPDB_API_KEY="..."       # abuseipdb.com (free)
export SHODAN_API_KEY="..."
export IPINFO_API_KEY="..."          # optional, raises rate limit
export URLSCAN_API_KEY="..."         # optional, search works keyless
export HIBP_API_KEY="..."            # PAID key required for v3
export HYBRID_ANALYSIS_API_KEY="..." # needs an approved account
export HUNTER_API_KEY="..."
export NUMVERIFY_API_KEY="..."
export CENSYS_API_ID="..."  ; export CENSYS_API_SECRET="..."
export MALWAREBAZAAR_API_KEY="..."   # abuse.ch unified Auth-Key (free at auth.abuse.ch)
export URLHAUS_API_KEY="..."         # SAME abuse.ch Auth-Key — covers URLhaus too
```

Note: MalwareBazaar and URLhaus both use the **single unified abuse.ch Auth-Key**
from auth.abuse.ch and it already setted at the same value for both. Generate/regenerate it under
Profile → Optional → Auth-Key (connecting a second login provider is recommended
so account access will not be lost).

## Usage

```bash
python3 main.py 8.8.8.8
python3 main.py example.com --analyst "*insert name*" --case-id CASE-001
python3 main.py https://example.com/login
python3 main.py 44d88612fea8a8f36de82e1278abb02f
python3 main.py user@example.com
python3 main.py +61400123456
python3 main.py <sha256> --no-pdf --no-navigator   # terminal only
```

Outputs land in `reports/` (PDF + Navigator JSON) and `logs/` (raw responses).
Open the Navigator JSON at https://mitre-attack.github.io/attack-navigator/
("Open Existing Layer" → "Upload from local").

## What runs for each indicator type

| Type   | Sources |
|--------|---------|
| IP     | VirusTotal, AbuseIPDB, Shodan, IPinfo, Censys, URLhaus, Ahmia |
| Domain | VirusTotal, URLScan, Shodan, WHOIS, DNS, Wayback, OSINT (dorks+SSL), Ahmia |
| URL    | VirusTotal, URLScan, Wayback, OSINT, URLhaus |
| Hash   | VirusTotal, MalwareBazaar, Hybrid Analysis, Ahmia |
| Email  | HaveIBeenPwned, Hunter, Ahmia |
| Phone  | NumVerify |

URLhaus (abuse.ch) tracks malware-distribution URLs and the hosts serving them and
catches malware infrastructure that general reputation engines can miss.

## Architecture

```
main.py                  orchestrator: detect → select → gather (asyncio) → intel → report
config.py                all keys (env) + endpoints + tunables
modules/
  base.py                shared async HTTP: retry, exponential backoff, rate limits
  indicator.py           auto-detect IOC type
  result.py              the one normalized SourceResult shape every module returns
  virustotal.py abuseipdb.py urlscan.py ipinfo.py shodan.py dns_enum.py
  whois_lookup.py malwarebazaar.py hybrid_analysis.py wayback.py censys.py
  ahmia.py hibp.py emailrep.py hunter.py numverify.py urlhaus.py osint_aggregator.py
intelligence/
  models.py              Technique / Hypothesis / DetectionRule / APTCandidate / Investigation
  mitre_mapper.py        findings → ATT&CK techniques (behaviour-tag + indicator-type mapping)
  hypothesis_generator.py techniques → hunting hypotheses
  detection_engineer.py  techniques → Wazuh XML + Sigma YAML + coverage/gaps
  apt_profiler.py        malware families → possible actors (low-confidence leads)
reporting/
  terminal_output.py     Rich banner + colour-coded panels/tables
  pdf_generator.py       full PDF report (reportlab)
  attack_navigator.py    ATT&CK Navigator layer JSON export
reports/  logs/          output folders
```

Three design choices worth understanding:
- **`base.py` centralises HTTP** so retry/backoff/rate-limit logic lives in one
  place, not copy-pasted into every module (DRY).
- **`result.py` normalises output** so the renderer, PDF and Navigator only ever
  handle one shape regardless of which API produced it (adapter pattern).
- **`mitre_mapper` maps by behaviour, not just type.** It reads the threat tags
  from every source (e.g. `elf`, `mozi`, `powershell`, `coinminer`, `mshta`) and
  maps the technique each behaviour implies, so a payload-hosting URL maps to
  T1105 Ingress Tool Transfer, not a blanket "spearphishing link." It ships a
  built-in ATT&CK catalog (fast, offline) with optional `mitreattack-python`
  STIX enrichment for power users.

## Important caveats

- **Detection rules are starter drafts.** They're readable and conservatively
  tuned in which validate against your own telemetry before production use.
- **Actor attribution is unreliable** by nature (shared tooling, false flags,
  commodity malware). The APT profiler only emits Low/Medium confidence and
  always records its basis. Treat it as leads, not conclusions.
- **Findings are decision support, not ground truth.** Absence of detections is
  not proof an indicator is safe.

## Changelog

**v0.2 (current)**
- Added **URLhaus** (abuse.ch) as an 18th source for IPs and URLs that catches
  malware-distribution infrastructure (validated live against a Mozi IoT
  botnet payload that other sources missed).
- Rebuilt MITRE ATT&CK mapping to be **behaviour-driven**: reads threat tags
  from any source and maps the implied technique, instead of assuming a
  technique from the indicator type. URLs are now classified as payload-host
  (T1105) vs phishing (T1566.002) based on evidence.
- **False-positive hardening:** filter Ahmia's own infrastructure onion out of
  results, and weight the overall verdict so a low-confidence source can't
  override authoritative ones (a clean VirusTotal result is no longer escalated
  to SUSPICIOUS by a lone Ahmia heuristic hit).

**v0.1**
- Initial release: 17 sources, concurrent async lookups, ATT&CK mapping,
  hunting hypotheses, Wazuh/Sigma detection rules, APT profiling, terminal +
  PDF + Navigator output.

Possible future extensions: URLScan deep-scan submission mode, passive-DNS /
reverse-IP via a dedicated source, expanded malware-family→technique coverage,
and a `--json` machine-readable output mode.

## License

Copyright (c) 2026 st0rmzking777. All Rights Reserved.
This repository is public for portfolio and evaluation purposes only; the code
may not be copied, modified, or reused without written permission. See `LICENSE`.
