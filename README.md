# Ultimate Technology Workflow

`ultimate-workflow.yaml` is a tiered Nuclei workflow that maximizes technology coverage while keeping HTTP requests under control. It chains fingerprint detection into full template runs only when a technology is identified.

## How it works

Nuclei evaluates five layers in order. Each layer uses a different detection strategy, from cheapest to most specific:

| Tier | Template | Source catalog | What it does |
|------|----------|----------------|--------------|
| **1** | `favicon-detect.yaml` | `catalogs/favicon` | Matches favicon hashes (1 request per host) |
| **2** | `tech-detect.yaml` | `catalogs/tech-detect` | Wappalyzer-style HTTP fingerprinting |
| **3** | `fingerprinthub-web-fingerprints.yaml` | `catalogs/fingerprinthub-detech` | FingerprintHub body/header signatures |
| **4** | `http/technologies/*-detect.yaml` | `catalogs/tect-all` | Dedicated technology detect templates |
| **5** | `http/exposed-panels/*` | `catalogs/exposed-panel` | Login panels and exposed admin UIs |

When a matcher hits in tiers 1–3, Nuclei runs all templates tagged with that technology (default-logins, CVEs, misconfigs, etc.).

```
Host → favicon? → tech-detect? → FingerprintHub? → tech detect? → panel detect?
              ↓           ↓              ↓                ↓              ↓
         tags: X     tags: X        tags: X          tags: X        tags: X
              └─────────── subtemplates (all nuclei templates for that tag) ──┘
```

### Deduplication strategy

- **Tiers 1–3**: full catalogs. Overlap is intentional — an early match stops further work on that path.
- **Tiers 4–5**: only technologies **not already covered** by upper tiers. This avoids redundant detect templates and panel probes on hosts already identified.

Each technology tag appears at most once in tiers 4 and 5.

## Quick start

```bash
# Run locally
cat webservers.txt | nuclei -w workflows/ultimate-workflow.yaml -stats -vv
```

## Current stats

- Tier 1: 537 favicon matchers
- Tier 2: 565 Wappalyzer matchers
- Tier 3: 1,874 FingerprintHub matchers
- Tier 4: 188 technology detect fallbacks
- Tier 5: 389 exposed panel fallbacks
