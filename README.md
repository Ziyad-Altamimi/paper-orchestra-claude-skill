# PaperOrchestra Skill for Claude Code

An evidence-first academic research and paper-writing skill for [Claude Code](https://docs.claude.com/en/docs/claude-code/overview), inspired by Google Research's [PaperOrchestra](https://github.com/google-research/paper-orchestra) multi-agent framework.

The skill guides Claude Code through a phased workflow that turns raw research materials (idea notes, experiment logs, data, figures) into a grounded paper draft. It never invents citations or results, and it marks every claim as verified, assumed, or missing.

> **Not an official Google project.** This is an independent adaptation of PaperOrchestra's *methodology*. It contains no PaperOrchestra code and does not run its Python pipeline. See [Attribution](#attribution).

## What the skill does

`paper-orchestra` is a single Claude Code skill (`SKILL.md` plus reference files). When you invoke it, or when Claude Code decides it fits your request, it:

- plans the research question, contributions, evaluation design, and literature search strategy
- finds and **verifies** sources with the tools available in your session, and records them in a source-evidence matrix
- builds a paper outline linked to evidence and data files
- drafts sections using only verified sources and your own data
- reviews the draft like a critical peer reviewer, against a quality checklist
- optionally prepares a LaTeX version using a template already in your project

Everything it writes goes into your research project, not into the skill folder.

## Key features

- **Phased workflow with approval gates.** Claude stops after each major phase and asks before moving on.
- **Run one phase or all of them.** For example, review an existing draft without running the earlier phases.
- **Source-evidence matrix.** Each source gets a DOI or URL, the claims it supports, where in the source each claim appears, and a verification status (`verified`, `partially-verified`, `unverified`, `rejected`).
- **Explicit evidence labels.** Content is marked **Verified**, **Assumption**, or **Missing**. Drafts use inline markers such as `[CITATION NEEDED: ...]`, `[DATA MISSING: ...]`, and `[ASSUMPTION: ...]`.
- **No overwrites.** Existing files are never overwritten without your permission.
- **Quality checklist** covering methodology, citations, reproducibility, limitations, and academic integrity.
- **No dependencies.** It uses Claude Code's built-in tools (file reading and writing, search, web search and fetch) and any research connectors you already have.

## How the workflow works

```text
Intake -> 1. plan -> 2. lit -> 3. outline -> 4. write -> 5. review -> 6. latex (optional)
             ^ approval gate after each phase, or run any single phase directly
```

1. **Intake.** Claude scans your project for existing materials (Markdown, LaTeX, BibTeX, CSV, JSON, notebooks, PDFs, logs, figures). It confirms the research question, target venue, length limit, citation style, literature cutoff date, and which phases you want.
2. **Phases.** Each phase writes its output files and appends to a `worklog.md`. It then reports what was produced, how many items are verified, assumed, or missing, and any open questions.
3. **Gate.** You choose to approve and continue, revise the phase, or stop.

## Installation

Claude Code loads personal skills from `~/.claude/skills/<skill-name>/SKILL.md`, and project skills from `.claude/skills/<skill-name>/SKILL.md` inside a repository. The repository root of this project *is* the skill folder, so you can clone it straight into place.

The extra repository files (`README.md`, `LICENSE`, `.gitignore`) sit next to `SKILL.md` and don't affect the skill.

### Linux and macOS

Personal install (available in all your projects):

```bash
mkdir -p ~/.claude/skills
git clone https://github.com/Ziyad-Altamimi/paper-orchestra-claude-skill.git ~/.claude/skills/paper-orchestra
```

Project install (shared with collaborators through that project's repository):

```bash
mkdir -p .claude/skills
git clone https://github.com/Ziyad-Altamimi/paper-orchestra-claude-skill.git .claude/skills/paper-orchestra
```

### Windows (PowerShell)

Personal install:

```powershell
New-Item -ItemType Directory -Force "$env:USERPROFILE\.claude\skills" | Out-Null
git clone https://github.com/Ziyad-Altamimi/paper-orchestra-claude-skill.git "$env:USERPROFILE\.claude\skills\paper-orchestra"
```

Project install, run from your project root:

```powershell
New-Item -ItemType Directory -Force ".claude\skills" | Out-Null
git clone https://github.com/Ziyad-Altamimi/paper-orchestra-claude-skill.git ".claude\skills\paper-orchestra"
```

The target folder name must be `paper-orchestra` so that it matches the skill's `name`. Start a new Claude Code session after installing. To update later, run `git pull` inside the skill folder.

## Usage

Run the full workflow, phase by phase:

```text
/paper-orchestra
```

Run a single phase:

```text
/paper-orchestra plan
/paper-orchestra lit
/paper-orchestra review
```

### Choosing the research project directory

The second argument is the project path. If you leave it out, the skill uses Claude Code's current working directory, and it confirms the location with you the first time.

```text
/paper-orchestra plan ./my-thesis
/paper-orchestra review ~/research/alert-triage-study
```

By default, outputs are written to a `paper-orchestra/` folder inside the project:

```text
<project>/paper-orchestra/
  00_project_brief.md
  01_research_plan.md
  02_evidence_matrix.md
  02_literature_review.md
  references.bib
  03_outline.md
  04_sections/
  05_review_report.md
  06_latex/
  worklog.md
```

If your project already has a structure such as `paper/`, `thesis/`, or `docs/`, the skill offers to use that instead.

You don't have to use the slash command. The skill's description lets Claude Code pick it up automatically for requests like "do a literature review for my paper" or "review this draft critically".

## Workflow phases

| Phase | Purpose | Main outputs |
|---|---|---|
| `plan` | **Research planning.** Problem, research questions, hypotheses, scoped contributions, evaluation design, evidence inventory, and a literature search strategy (broad context for the introduction; methodology clusters with limitation hypotheses for related work). | `00_project_brief.md`, `01_research_plan.md` |
| `lit` | **Literature review and evidence matrix.** Discovers sources from local files, connected research tools, and the web. Checks each source's metadata, publication date against the cutoff, and the specific claim it supports. | `02_evidence_matrix.md`, `02_literature_review.md`, `references.bib` |
| `outline` | **Paper structure.** Section hierarchy (following a venue template if you provide one), content bullets tied to source materials, citations per subsection, a figure and table plan, word budget, and limitations and reproducibility sections. | `03_outline.md` |
| `write` | **Evidence-based drafting.** Writes sections one at a time. Uses exact values from your data files, cites only verified sources, and marks gaps inline. | `04_sections/*.md` |
| `review` | **Critical review.** Strengths, weaknesses, questions, a score with rationale, checklist results, and a prioritized revision plan. Revisions are applied only after your approval, for up to about three rounds. | `05_review_report.md` |
| `latex` | **LaTeX preparation (optional).** Converts approved sections into LaTeX using your venue template or a minimal `article` skeleton. Offers to compile only if a TeX toolchain is already installed. | `06_latex/` |

Use `all`, or no phase argument, for the full workflow.

## Example research use cases

- Turning experiment logs and notes into a conference or workshop paper draft
- Building a verified literature review and BibTeX file for a thesis chapter
- Critically reviewing an existing manuscript before submission
- Planning a study's evaluation design and threats to validity before running experiments
- Preparing a security-operations study. `references/example-miyar-alert-investigation.md` shows a short worked example: a study of investigating a Wazuh alert on a Windows endpoint, with placeholder values only.

## Project structure

```text
paper-orchestra-claude-skill/
├── SKILL.md                                    # Skill definition (frontmatter + workflow)
├── README.md                                   # This file
├── LICENSE                                     # MIT
├── .gitignore
└── references/
    ├── templates.md                            # Evidence matrix, outline, review, worklog templates
    ├── example-miyar-alert-investigation.md    # Short worked example
    └── provenance.md                           # Origin and relationship to PaperOrchestra
```

## Academic integrity and evidence verification

The skill's integrity rules override requests for speed or polish:

- It never fabricates references, authors, DOIs, URLs, quotations, statistics, results, figures, or citation keys.
- Every citation must match a `verified` or `partially-verified` row in the evidence matrix. Anything else is flagged as `[CITATION NEEDED]`.
- Every number must trace to a file you provided. Anything else is flagged as `[DATA MISSING]`.
- It doesn't claim to beat a cited work unless your own results compare against it directly.
- Reviewer requests for new experiments are recorded as limitations or future work, never invented.
- Limitations are always stated.
- Drafts are your work product to check and take responsibility for. Follow your institution's or venue's policy on disclosing AI assistance.

Verification depends on what Claude can actually open in your session. Paywalled or unreachable sources stay `unverified` until you supply them.

## Limitations

- This is a prompt-based workflow, not software. Output quality depends on the model, your materials, and the tools available in your Claude Code session.
- Literature discovery is limited to your local files, any research connectors you have set up, and web search and fetch. It doesn't replace a systematic database search.
- Claude can still make mistakes. Check every citation and number before you submit anything.
- LaTeX compilation works only if `latexmk`, `pdflatex`, or `bibtex` is already installed. The skill does not install dependencies.
- It doesn't reproduce PaperOrchestra's plotting agent, automated scoring (autoraters), or reported benchmark results.

## Attribution

This skill is an independent adaptation inspired by **PaperOrchestra** by Google Research:

- Repository: <https://github.com/google-research/paper-orchestra> (Apache License 2.0)
- Paper: Yiwen Song, Yale Song, Tomas Pfister, Jinsung Yoon. *PaperOrchestra: A Multi-Agent Framework for Automated AI Research Paper Writing.* arXiv:2604.05018, 2026.

Only the high-level methodology was adapted: outlining, literature review with verification, section writing, review-driven refinement, and formatting. No source code, prompts, templates, or other files from PaperOrchestra are included. This repository is **not affiliated with, endorsed by, or officially supported by Google** or the PaperOrchestra authors. See [`references/provenance.md`](references/provenance.md) for what was adapted and where this skill deliberately differs.

## License

Released under the [MIT License](LICENSE). PaperOrchestra itself is licensed separately under the Apache License 2.0 by Google LLC. Its license is not affected by, and does not apply to, the files in this repository.
