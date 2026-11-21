---
name: paper-orchestra
description: Phased, evidence-first research-paper workflow - research planning, verified literature review with a source-evidence matrix, paper outline, section drafting, critical peer-style review, and optional LaTeX preparation. Use when the user wants to plan, write, review, or restructure an academic/technical paper, thesis chapter, or research report from ideas, notes, logs, or experimental results; when they ask for a literature review or citation verification; when they want a full research workflow run autonomously from a single topic; or when they invoke /paper-orchestra (optionally with a mode such as auto, plan, lit, outline, write, review, latex).
argument-hint: "[auto \"topic\" | plan|lit|outline|write|review|latex|all] [project path or topic]"
---

# PaperOrchestra (Claude Code adaptation)

A phased research-writing workflow that turns raw materials (idea notes, experiment logs, data, figures) into a grounded paper. It adapts the *methodology* of Google Research's PaperOrchestra (outline -> literature review -> section writing -> review-driven refinement -> formatting) to Claude Code's own tools. It does **not** run, wrap, or reproduce the PaperOrchestra Python software, its agents, its LLM backends, or its autoraters. See `references/provenance.md`.

## Non-negotiable integrity rules

These override any user request for speed, length, or polish.

1. **Never fabricate** references, authors, titles, venues, years, DOIs, URLs, quotations, statistics, experimental results, figures, or citation keys.
2. Every citation in a draft must map to a row in the source-evidence matrix with status `verified` (or be visibly flagged `[CITATION NEEDED: ...]`).
3. Every number in a draft must trace to a user-provided file (log, CSV, notebook output) by path. If it cannot, write `[DATA MISSING: ...]` instead.
4. Label content explicitly as one of: **Verified** (checked against a source you opened this session), **Assumption** (reasonable but unconfirmed - state why), **Missing** (needed but not available).
5. Quotations: only verbatim text you actually read in the source, kept short, with location (page/section). Otherwise paraphrase and cite.
6. Do not claim superiority over a cited work unless the user's own results compare against it directly. Otherwise frame it as related, concurrent, or orthogonal work.
7. Do not invent experiments a reviewer asks for. Record them as limitations or future work.
8. **Limitations are always stated honestly** (this deliberately departs from the original refinement prompt, which told the model to avoid stating limitations).
9. Keep proposed methods, hypotheses, and planned experiments distinct from actual results. Never describe a proposed experiment as completed, never claim an empirical contribution without supporting data, and never turn an untested hypothesis into a conclusion.
10. No ghost-authorship deception: drafts are the user's work product to review, edit, and take responsibility for. Remind the user of their institution's AI-use and disclosure policy when a draft is meant for submission.

## Where files go

- All outputs live in the **active research project**, never in this skill folder.
- Project root = the path given in the arguments, else the current working directory. Confirm it with the user on first use.
- Default layout (create only what the chosen phase needs):

```
<project>/paper-orchestra/
  00_project_brief.md        # phase 1
  01_research_plan.md        # phase 1
  02_evidence_matrix.md      # phase 2 (+ optional .csv twin)
  02_literature_review.md    # phase 2
  references.bib             # phase 2 (verified entries only)
  03_outline.md              # phase 3
  04_sections/<nn>_<name>.md # phase 4
  05_review_report.md        # phase 5
  06_latex/                  # phase 6 (optional)
  worklog.md                 # append-only log of decisions per phase
```

- If the user already has a structure (e.g. `paper/`, `thesis/`, `docs/`), propose using it instead.
- **Never overwrite** an existing file without explicit permission. Before writing, check whether the target exists (Glob/Read). If it does, ask: overwrite, write a versioned copy (`_v2`), or append. Default to a versioned copy if the user is unavailable.
- Never delete user files.

## Phase selection

Parse the first argument (`$ARGUMENTS`) or the user's request:

| Arg | Phase | Needs as input |
|---|---|---|
| `plan` | 1. Research planning | topic/idea, any notes |
| `lit` | 2. Literature review + evidence matrix | research questions or outline |
| `outline` | 3. Paper outline | brief/plan, results, target venue |
| `write` | 4. Section writing | outline, evidence matrix, data |
| `review` | 5. Critical review | an existing draft |
| `latex` | 6. LaTeX preparation (optional) | approved sections, verified .bib |
| `all` / none | Full workflow, phase by phase, with approval gates | - |
| `auto ["topic"]` | Full workflow, all six phases, **no approval gates** - see *Autonomous mode* | a topic (or inferable context) |

Anything other than `auto` uses the approval-based workflow below. Running a single phase is fully supported. If a phase's inputs are missing, say what is missing, offer to run the earlier phase, or proceed with clearly marked assumptions - the user chooses.

## Phase gates

(Manual modes only. `auto` skips these gates - see *Autonomous mode*.)

Work one phase at a time. At the end of each major phase:

1. Write the phase's output file(s) and a short entry in `worklog.md`.
2. Report: what was produced (paths), Verified / Assumption / Missing counts, open questions.
3. **Ask for approval before starting the next phase** (use AskUserQuestion when available: Approve & continue / Revise this phase / Stop here). Do not chain phases silently.

## Phase 0 - Intake (always, brief)

- Glob the project for existing materials: `*.md`, `*.tex`, `*.bib`, `*.csv`, `*.json`, `*.ipynb`, `*.pdf`, logs, figure folders. Read what is relevant.
- Establish: research question, contribution claim, audience/venue, page or word limit, citation style, deadline, **literature cutoff date** (default: today), and which phases the user wants.
- Map inputs to PaperOrchestra-style roles: *idea* (method, contributions), *experimental log* (results, ablations, metrics), *template/guidelines* (venue rules), *figures*.

## Phase 1 - Research planning

Output `00_project_brief.md` and `01_research_plan.md` containing:
- Problem statement, research question(s), hypotheses, and scope boundaries.
- Claimed contributions, each scoped strictly to the evidence the user has (no over-generalisation beyond the datasets/scenarios actually evaluated).
- Methodology summary and evaluation design (data, baselines, metrics, threats to validity).
- Evidence inventory: what results exist, what is missing, what is assumed.
- Literature search strategy, split as in PaperOrchestra:
  - **Introduction (macro context):** impact/urgency of the problem, surveys, foundational works. 3-5 queries.
  - **Related work (micro, technical):** 2-4 methodology clusters; per cluster: what to investigate, a *limitation hypothesis*, narrow queries to test it, and the *bridge* to the user's approach.
  - **Citation hints:** every dataset, tool, framework, metric, standard, and baseline mentioned needs a canonical source. If you do not know the exact source, write `source introducing '<name>'` - never guess authors.

## Phase 2 - Evidence-based literature review

Source discovery, in priority order:
1. User's local files (PDFs, .bib, notes) - Read them.
2. Connected research tools if present (e.g. a scholarly-search MCP such as Consensus, Exa, or a Google Drive/Zotero connector).
3. WebSearch / WebFetch: publisher pages, DOI resolver (`https://doi.org/<doi>`), arXiv, ACM DL, IEEE Xplore, Semantic Scholar, Crossref, official standards bodies (NIST, ISO, IETF RFCs, MITRE ATT&CK), vendor documentation.

Verification procedure for each candidate source:
- Open the landing page or document. Confirm title, authors, year, venue, and DOI/URL match.
- Confirm the source is published before the cutoff date; later work may only be called concurrent.
- Confirm the specific claim it will support by reading the abstract or the relevant section. Record where (abstract, section, page).
- Deduplicate by normalised title.
- Assign status: `verified`, `partially-verified` (metadata confirmed, claim only from abstract/secondary), `unverified` (found but not opened / paywalled / mismatch), `rejected` (does not support the claim or does not exist as described).

Outputs:
- `02_evidence_matrix.md` - template in `references/templates.md`. Columns: ID, citation key, full reference, DOI/URL, source type, claim(s) supported, where in source, used in section(s), verification status, verified on (date), notes.
- `02_literature_review.md` - synthesis organised by theme/cluster (not a paper-by-paper list): consensus, disagreements, gaps, and how the gap motivates the user's work. Cite matrix IDs/keys only.
- `references.bib` - only `verified` and `partially-verified` entries; BibTeX built from the metadata you confirmed, not from memory.
- A gap list: claims that still need a source.

## Phase 3 - Paper outline

Output `03_outline.md` (template in `references/templates.md`):
- Section hierarchy following the venue template if provided (no orphan subsections: if X.1 exists, X.2 must too).
- Per subsection: content bullets that point to specific source materials (e.g. "Formalise the scoring rule from `idea.md` section 2"), not generic instructions; the evidence-matrix IDs to cite; and the data files that feed each table or figure.
- Figure/table plan: id (semantic, e.g. `fig_pipeline_overview`), type (plot/diagram/table), data source path, purpose.
- Sizing: approximate words per section against the page/word limit.
- An explicit Limitations subsection and a reproducibility/artifact plan.

## Phase 4 - Section writing

- Write one section (or a small group) at a time into `04_sections/`, in the user's chosen language and style. Ask which section to start with if unclear; default order: Method -> Experiments/Results -> Related Work -> Introduction -> Conclusion -> Abstract.
- Tables use exact values from the user's data files; cite the file path in a source comment (`<!-- source: results/run3.csv -->`).
- Describe figures only as they actually appear; do not interpret beyond what the data shows.
- Include ablations/analysis of what worked, what did not, and why - only if supported.
- Mark inline: `[CITATION NEEDED: claim]`, `[DATA MISSING: what]`, `[ASSUMPTION: what, why]`.
- Preserve already-approved sections; edit them only for consistency and say what changed. Prefer returning changed passages rather than regenerating whole documents.

## Phase 5 - Critical review

Act as a rigorous, fair reviewer (adapted from PaperOrchestra's review-driven refinement loop). Output `05_review_report.md` containing:
- Summary; Strengths; Weaknesses; Questions to authors; an overall assessment with a 1-10 score and confidence, and the reasoning behind both.
- The **quality checklist** results (below), each item pass / issue / n/a with location.
- A prioritised revision plan. For each item: addressable by rewriting with existing evidence, or requires new evidence (cannot be fabricated; becomes a limitation or future work).
- Only after approval, apply revisions (at most ~3 review->revise rounds, as in the original loop), logging each change in `worklog.md` (addressed weaknesses, integrated answers, actions taken). Stop early when no material issue remains.

### Quality checklist

**Methodology**
- Research question, hypotheses, and contributions are explicit and consistent across abstract, intro, and conclusion.
- Method is described in enough detail to re-implement; design choices are justified.
- Baselines, datasets/scenarios, and metrics are appropriate and cited; evaluation matches the claims' scope.
- Threats to validity (internal, external, construct) are discussed.

**Citations**
- Every `\cite`/reference resolves to a `verified` or `partially-verified` matrix row; no orphan or unused keys.
- Each cited source actually supports the sentence it is attached to.
- No post-cutoff work presented as a baseline that was beaten; no superiority claims without direct comparison.
- Quotations are verbatim, short, and located.

**Reproducibility**
- Data sources, versions, configurations, environment, random seeds, and hardware are stated or marked missing.
- Every number traces to a file; tables match the data exactly (spot-check by re-reading the source files or recomputing with code).
- Artifact availability (code, data, scenario definitions) is stated, including restrictions.

**Limitations**
- A dedicated limitations discussion exists and covers scope, data, generalisability, and threats to validity.
- Unanswered reviewer questions needing new experiments are listed honestly.

**Academic integrity**
- No fabricated content; all `[CITATION NEEDED]`, `[DATA MISSING]`, `[ASSUMPTION]` markers are resolved or consciously kept.
- Paraphrases are genuinely re-expressed, not close copies; prior work is credited.
- Ethics: human subjects, sensitive data, or dual-use/security concerns addressed (e.g. responsible disclosure, anonymisation).
- AI-assistance disclosure handled per the venue's or institution's policy.

## Phase 6 - LaTeX preparation (optional)

- Use the user's venue template if present in the project; otherwise ask, or offer a minimal `article` skeleton. Do not download templates without asking.
- Convert approved sections into `06_latex/main.tex` (+ `sections/*.tex`), using `references.bib`, `booktabs` tables, `\label`/`\ref` for all figures and tables, and figure paths that exist.
- Keep the template preamble intact; check `\begin`/`\end` pairing; keep unresolved markers as visible `\textcolor{red}{...}` notes.
- If `latexmk`/`pdflatex`/`bibtex` already exist, offer to compile and report errors and warnings (undefined citations or references, overfull boxes). Do not install a TeX distribution or any other dependency.

## Autonomous mode (`auto`)

Invocation: `/paper-orchestra auto "Research Topic"` or `/paper-orchestra auto`. Runs Phases 0-6 end to end **without approval gates**. Every integrity rule above still applies in full; autonomy changes who decides routine choices, never what may be claimed.

### Topic resolution
1. Use the quoted topic if given (any further argument is the project path).
2. Otherwise infer it from the current conversation, then from project context (README, idea/notes files, existing drafts). State the inferred topic in the first progress update and in `research-worklog.md`.
3. Only if neither yields a clear topic, ask **once**. If still unanswered, stop with status **Blocked**.

### Defaults for non-critical decisions
Do not stop to ask about these; pick a default, record it under "Defaults applied" in `research-worklog.md`, and continue:
- Venue: generic academic manuscript (Abstract, Introduction, Related Work, Method/Approach, Evaluation or Evaluation Plan, Discussion, Limitations, Conclusion, References).
- Length: about 6,000-8,000 words. Citation style: numeric (`plainnat`/`unsrtnat`). Language: the language of the request.
- Literature cutoff: today. Project root: argument path, else current working directory.
- Section order, figure plan, and wording choices: your judgment, documented.

### When to stop anyway
Pause only for: a fundamentally ambiguous topic (see above); an action needing authentication, permission, or private resources you cannot reach; or an irreversible or destructive action. Missing data is **not** a reason to stop - it changes the output type (below).

### Evidence-dependent output type
- **Results present** (logs, CSVs, notebooks with outputs): write a full empirical manuscript from that data.
- **No results**: write a rigorous research proposal / draft. The Evaluation section becomes an *Evaluation Plan* (design, datasets or scenarios, baselines, metrics, analysis) under a visible `PENDING EVALUATION - no results reported` banner; contributions are phrased as proposed; the abstract and conclusion make no empirical claims.
- **Source verification unavailable** (no web access or connectors): continue with local sources only, mark every unchecked reference `unverified`, and keep them out of factual claims.

### Output layout (auto mode)
```
<project>/paper-orchestra/
  research-plan.md            # Phase 1
  source-evidence-matrix.md   # Phase 2 (matrix template in references/templates.md)
  literature-review.md        # Phase 2
  paper-outline.md            # Phase 3
  manuscript.md               # Phase 4, revised in Phase 5
  critical-review.md          # Phase 5, one section per round
  limitations.md              # limitations, threats to validity, reproducibility requirements
  research-worklog.md         # progress, defaults, decisions, per-phase counts
  latex/main.tex              # Phase 6
  latex/references.bib        # Phase 6, verified/partially-verified entries only
```
Create other files only when genuinely needed. Keep all intermediate artifacts so the run is auditable.

**No overwrites:** before writing, check each target. If it exists, never replace it. If `paper-orchestra/` already holds a previous run, write this run to `paper-orchestra/run-YYYYMMDD-HHMM/` with the same layout; for single existing files, write `<name>.rN.md`. Record which choice was made in the worklog. Within the run, revisions of `manuscript.md` are preserved as `manuscript.r0.md`, `manuscript.r1.md`, ... before each review round edits the current file.

### Execution
1. **Phase 0-1:** intake and planning as described above -> `research-plan.md`.
2. **Phase 2:** search and verify -> `source-evidence-matrix.md`, `literature-review.md`. Aim for enough verified sources to support every claim; a smaller verified set beats a larger unverified one.
3. **Phase 3:** outline -> `paper-outline.md`.
4. **Phase 4:** draft the whole manuscript -> `manuscript.md`, with inline `[CITATION NEEDED]`, `[DATA MISSING]`, `[ASSUMPTION]`, and `[PROPOSED]` markers where applicable.
5. **Phase 5:** up to **three** review-and-revision rounds. Each round: write a review (checklist + scored assessment) to `critical-review.md`, snapshot the manuscript, apply fixes that existing evidence supports, log what was resolved and what was not. Stop early when a round finds no material issue that can be fixed with available evidence. Unresolvable items go to `limitations.md`.
6. **Phase 6:** generate `latex/main.tex` and `latex/references.bib` from the final manuscript; check `\cite` keys against the .bib and `\begin`/`\end` pairing. If `latexmk`/`pdflatex` already exist, compile once and record the result; never install anything. Then run the final validation: the full quality checklist, plus a search of the manuscript for unresolved markers.

### Progress updates
After each phase, send one or two lines (phase done, key counts, any default applied). Do not ask for confirmation.

### Final report
End with (template in `references/templates.md`): research title; research questions; number of `verified` sources (and partially-verified/unverified counts); phases completed; main findings **supported by evidence** (each with its source or data file; none if no results); outstanding evidence and experimental requirements; review issues identified vs. resolved; paths to all generated files; and a final status:
- **Complete** - all claims supported, empirical results present where claimed, no unresolved critical markers.
- **Draft Pending Evidence** - workflow finished but results, sources, or data are missing (e.g. evaluation pending). Never call this Complete.
- **Blocked** - could not proceed (unclear topic, required access unavailable); state what is needed.

## Tool usage summary

- Glob / Grep / Read for local materials; Write / Edit for outputs (after existence checks).
- WebSearch / WebFetch (or connected scholarly MCPs) for discovery and verification; record the URL actually opened.
- Bash only for local checks, e.g. recomputing a table value, grepping cite keys against the .bib, or compiling LaTeX with tools already installed.
- AskUserQuestion at phase gates and for overwrite decisions (manual modes only); a task list for multi-phase runs, including `auto`.

## Reference files

- `references/templates.md` - evidence matrix, outline, review report, worklog, limitations, and auto-mode final report templates.
- `references/example-miyar-alert-investigation.md` - short worked example (security alert investigation study).
- `references/provenance.md` - origin and relationship to PaperOrchestra.
