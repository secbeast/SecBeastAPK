# SecBeastAPK

Advanced Automated Android Security Analysis Framework — built for bug bounty hunters, mobile pentesters, malware analysts, red team operators, and security researchers running **authorized** assessments.

SecBeastAPK orchestrates industry-standard tooling (APKTool, JADX, ProjectDiscovery stack, Nuclei, optional MobSF) behind a **single Rich-powered CLI**, producing **HTML / JSON / Markdown** reports with quantitative risk scoring.

> **Legal use only.** Run SecBeastAPK only against APKs you own or have explicit permission to test.

---

## Features

| Area | Capability |
|------|------------|
| Static analysis | APK decode (APKTool), Java/Kotlin decompilation (JADX), manifest inspection |
| Secrets | Regex + entropy scanning for keys/tokens/JWT-like strings |
| Endpoints | URLs, domains, IPs, buckets, WebSockets, GraphQL hints |
| Cloud | Opportunistic bucket/listing checks against extracted URLs |
| Firebase | Async probes for common REST exposure paths |
| Network recon | httpx, subfinder, katana, nuclei (full pipeline) |
| APIs | GraphQL introspection probe; ffuf-based fuzzing with bundled wordlist |
| Risk engine | Severity-weighted scoring + attack surface heuristics |
| Plugins | Drop Python modules under `plugins/` implementing `analyze(ctx)` |
| Scale | `--massive` directory scanning with bounded concurrency |

---

## Screenshots

Terminal output uses Rich tables, progress indicators, and severity-colored summaries. Add your own screenshots under `docs/screenshots/` after first runs (`README` intentionally ships without binary images).

Suggested captures:

1. `secbeastapk ./samples/app.apk --full` — summary table  
2. Generated `output/<slug>/report.html` — executive dashboard  

---

## Requirements

- **Python 3.12+**
- **Java** (for JADX / some tooling)
- External binaries on `PATH` (installed automatically by `install.sh` where possible):

  | Binary | Purpose |
  |--------|---------|
  | `apktool` | Resource/manifest decode |
  | `jadx` | Java/Kotlin sources |
  | `httpx` | Alive hosts / fingerprint |
  | `nuclei` | Template scanning |
  | `subfinder` | Passive subdomains |
  | `katana` | Crawling |
  | `ffuf` | Content discovery |

Optional: MobSF (REST API), Frida, objection, mitmproxy — referenced when present.

---

## Quick start (Linux / macOS)

```bash
git clone https://github.com/secbeast/SecBeastAPK.git
unzip SecBeastAPK.zip -d SecBeastAPK
cd SecBeastAPK
chmod +x install.sh
./install.sh
source .venv/bin/activate

python secbeastapk.py /path/to/app.apk --full
```

Outputs land in `./output/<apk-slug>/` including `report.html`, `report.json`, `report.md`.

---

## CLI usage

```bash
# Baseline static scan + lightweight network probes
python secbeastapk.py app.apk

# Full offensive pipeline (httpx → katana → nuclei → ffuf)
python secbeastapk.py app.apk --full

# Dynamic toolchain checks + mitm guidance stub
python secbeastapk.py app.apk --dynamic

# Concurrency tuning
python secbeastapk.py app.apk --threads 50

# Report formats only
python secbeastapk.py app.apk --report html,json

# Severity reporting filter (post-scan filter)
python secbeastapk.py app.apk --severity critical,high

# Mass scanning
python secbeastapk.py --massive ./apk-folder/ --full --threads 32

# Air-gapped static-only style analysis (no outbound HTTP)
python secbeastapk.py app.apk --offline
```

---

## Configuration

Edit `config/settings.yaml` or export:

| Env var | Purpose |
|---------|---------|
| `SEC_BEAST_CONFIG` | Alternate YAML path |
| `MOBSF_URL` | MobSF base URL |
| `MOBSF_API_KEY` | MobSF REST token |

---

## Docker

```bash
docker compose build
docker compose run --rm secbeastapk /apk/sample.apk --full
```

Mount your APKs read-only at `./apks` → `/apk` per `docker-compose.yml`.

---

## Architecture overview

```
secbeastapk.py (Typer CLI)
    └── core/runner.py — orchestrates asynchronous stages
            ├── modules/apktool.py / jadx.py
            ├── modules/extractor.py / secrets.py
            ├── modules/firebase.py / cloud.py
            ├── modules/httpxscan.py / subfinder.py / katana_scan.py
            ├── modules/nuclei.py / graphql.py / api_fuzzer.py
            ├── modules/reporter.py — Jinja2 templates
            └── core/risk_engine.py — scoring
```

Plugins are discovered from `plugins/*.py` (classes exposing async `analyze`).

---

## Roadmap

- [ ] YAML rule packs under `rules/` with inline matchers  
- [ ] Differential scans between APK versions  
- [ ] SARIF export for CI ingestion  
- [ ] Optional SQLite corpus for massive campaigns  

---

## Contributing

1. Fork / branch from `main`.  
2. Run `python -m compileall .` before submitting.  
3. Keep findings factual — avoid noisy duplicate signatures.  
4. Document new modules in this README (Features + Architecture).  

---

## License

MIT — see `LICENSE`.
