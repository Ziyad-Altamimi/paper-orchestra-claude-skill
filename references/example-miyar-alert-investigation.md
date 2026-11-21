# Worked example: MIYAR security-alert investigation study

Illustrative only. The skill itself is general-purpose; this shows how the phases apply to a security-operations research project. Everything below in angle brackets is a placeholder - no sources or results are asserted here.

## Context

MIYAR (Active Counterfactual Investigation Lab) is a student graduation project: an investigation-first, evidence-sufficiency platform for SOC L2 analysts, scoped initially to one Windows endpoint scenario monitored with Wazuh.

Invocation: `/paper-orchestra plan <path-to-project>`

## Phase 1 - plan (excerpt)

- **RQ:** Does prompting analysts with counterfactual checks ("what evidence would exist if this alert were benign?") improve investigation decisions on a Wazuh alert compared with a standard triage workflow?
- **Scope:** one Windows scenario, <N> alerts, <participants or simulated analysts>. Contributions are claimed for this scenario only.
- **Evidence inventory:**
  - Verified: scenario definition file, Wazuh rule IDs used (`<path>`).
  - Assumption: replayed logs are representative of real SOC traffic - to be argued, or stated as a limitation.
  - Missing: decision-accuracy and time-to-decision results (experiment not yet run).
- **Search strategy:**
  - Intro (macro): alert fatigue and SOC workload; surveys of SOC automation.
  - Related-work clusters: (1) alert triage and prioritisation, (2) provenance-graph or causal investigation, (3) counterfactual / contrastive explanation in security.
  - Citation hints: source introducing 'Wazuh' (official documentation), source introducing 'MITRE ATT&CK', 'Sysmon' documentation, NIST SP 800-61 (incident handling guide - confirm the current revision before citing).

## Phase 2 - lit (matrix excerpt)

| ID | Cite key | DOI / URL opened | Claim supported | Status |
|---|---|---|---|---|
| S01 | `<key>` | `<official ATT&CK URL>` | Technique IDs used to label the scenario | verified (docs page opened) |
| S02 | `<key>` | `<doi>` | SOC analysts face high false-positive volume | partially-verified (abstract only) |
| S03 | `<key>` | - | "Counterfactual reasoning reduces confirmation bias in triage" | unverified - no source found -> claim goes to gap list |

S03 must not be cited. The draft either finds a verified source or reframes the claim as the study's hypothesis.

## Phase 4 - write (marker usage)

> Analysts in the counterfactual condition reached a decision in [DATA MISSING: median time per alert, results/timing.csv] compared with [DATA MISSING] for the baseline. Wazuh rule <id> mapped to ATT&CK technique <Txxxx> [S01].

## Phase 5 - review (checklist hits)

- Reproducibility: Wazuh version, ruleset version, Sysmon config, and log-replay method must be stated -> currently missing.
- Limitations: single scenario, small sample, simulated traffic, and possible learning effect between conditions.
- Integrity / ethics: anonymise any real hostnames or user data in logs; disclose AI assistance per university policy.
