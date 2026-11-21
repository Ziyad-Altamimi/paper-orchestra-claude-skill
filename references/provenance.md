# Provenance

This skill is an **independent adaptation** inspired by PaperOrchestra:

- Repository: https://github.com/google-research/paper-orchestra (Apache-2.0)
- Paper: Song, Song, Pfister, Yoon. "PaperOrchestra: A Multi-Agent Framework for Automated AI Research Paper Writing." arXiv:2604.05018 (2026).

It is **not** an official Google product or Google-authored skill, is not affiliated with or endorsed by Google or the PaperOrchestra authors, and contains none of the project's code. It does not run PaperOrchestra's Python agents, its Gemini/OpenAI backends, Semantic Scholar pipeline, plotting agent, or autoraters, and it makes no claim to reproduce their outputs or reported results.

## What was adapted (methodology only)

| PaperOrchestra stage | This skill |
|---|---|
| Outline agent: plotting plan, intro vs related-work search strategy (macro vs micro), methodology clusters with limitation hypotheses and "bridge", exhaustive citation hints without guessed authors | Phase 1 planning and Phase 3 outline |
| Literature review agent: candidate discovery, metadata verification, cutoff-date filtering, dedup by normalised title, cite only collected papers | Phase 2 with a human-readable source-evidence matrix and explicit verification statuses |
| Section writing agent: exact numbers from the experimental log, booktabs tables, faithful figure descriptions, preserve existing sections | Phase 4 |
| Content refinement agent and review: reviewer strengths/weaknesses/questions, up to about 3 revise rounds, ignore requests for new experiments, keep a worklog | Phase 5 |
| Venue LaTeX templates and format checks | Phase 6 (optional, uses only locally available tools) |

## Deliberate differences

- Human approval gates between phases by default; single-phase runs. An optional `auto` mode runs end to end like the original pipeline, but keeps every integrity rule, produces a proposal instead of results when data is missing, and reports an honest final status.
- Verified / Assumption / Missing labelling everywhere.
- Limitations must be stated (the original refinement prompt instructed the opposite).
- General-purpose (any field), not just ML conference papers.
- Outputs stay in the user's project; no files are overwritten without permission.
