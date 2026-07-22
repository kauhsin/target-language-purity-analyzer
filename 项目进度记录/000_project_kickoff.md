# 000 Project Kickoff

Date: 2026-07-17

Project: Explainable Target-Language Purity Analyzer

Repository:

```text
https://github.com/kauhsin/target-language-purity-analyzer
```

Local workspace:

```text
/Users/xing/Documents/方言纯度分析
```

## Purpose of This Kickoff

This kickoff task prepared the working environment and collaboration process for
the target-language purity analyzer project. It did not start Milestone 0
implementation. The project owner explicitly wants to move slowly and use this
project as a way to learn the machine learning workflow, not just to generate a
finished codebase quickly.

The next substantive task should be a new Codex task for:

```text
Milestone 0: Problem Definition and Output Protocol
```

## Core Project Idea

The project will analyze generated text for consistency with a requested target
language variety:

- Shanghainese
- Cantonese
- Mandarin

The intended system should answer:

1. Whether a generated sentence is consistent with the requested target variety.
2. If not, which competing language variety is strongest.
3. Which spans provide model evidence for Shanghainese, Cantonese, or Mandarin.
4. Optionally, which real training-corpus examples support the explanation.

The project is not intended to make absolute word-level language ownership
claims.

## Current Accepted Technical Direction

The current accepted high-level direction comes from the handoff document:

```text
PROJECT_HANDOFF_Target_Language_Purity_Analyzer.md
```

Important constraints from that document:

- Train only one Transformer model in version 1.
- The main training data consists primarily of pure-language corpora:
  Shanghainese, Cantonese, Mandarin.
- Do not train a second mixed-language detector.
- Do not add a `Mixed` training label to the main classifier.
- Do not require token-level language labels.
- Do not say that a word absolutely belongs to one language.
- Use the wording `evidence for Shanghainese`, `evidence for Cantonese`, and
  `evidence for Mandarin`.
- Use occlusion / leave-one-out attribution as the core explanation method.
- Training-corpus retrieval may be added as auxiliary evidence, but it must not
  replace the main classifier.
- Version 1 does not require Mandarin translations, parallel corpora, automatic
  rewriting, or semantic preservation checking.
- Mixed sentences are mainly for testing and challenge evaluation.
- Do not interpret softmax scores as percentages of words or strict language
  purity percentages.

## Collaboration Rules

The working rules are stored in:

```text
codex_working_rules.md
```

The most important working rule is:

```text
Discuss first. Design second. Build third.
```

Default mode is discussion. Unless the project owner explicitly approves
implementation, Codex should not create files, create folders, write code, or
refactor the repository.

Milestones should follow this process:

1. Discuss.
2. Design.
3. Wait for explicit approval.
4. Implement only the approved milestone.
5. Review.
6. Stop.

Codex should challenge weak assumptions, explain tradeoffs, preserve design
rationale, and avoid premature engineering.

## Important Correction During Kickoff

Codex initially moved too quickly and created draft README, docs, code, tests,
and data scaffolding. The project owner clarified that this was too fast. Those
draft files were removed, and the project returned to a minimal state.

This is now an explicit project norm:

- Do not scaffold ahead.
- Do not implement before approval.
- Do not create project documents until the project owner asks for them.
- Let the linguistic expert control language judgments and data quality.

## Files Placed in the Local Project

Two project rule files were copied into the local root folder:

```text
PROJECT_HANDOFF_Target_Language_Purity_Analyzer.md
codex_working_rules.md
```

At the time this kickoff record is written, these two files exist locally but
are intentionally not committed to GitHub yet.

## Project Progress Record System

A progress-record system was agreed on.

Historical records should live in:

```text
项目进度记录/
```

Each completed task or phase can produce a numbered record, for example:

```text
项目进度记录/000_project_kickoff.md
项目进度记录/001_milestone_0_problem_definition.md
```

The root file:

```text
CURRENT_PROJECT_STATE.md
```

should summarize the current state of the project. New Codex tasks should read
this current-state file rather than reading every historical record by default.
Historical records can be consulted when a past decision needs to be traced.

Recommended startup instruction for future tasks:

```text
Please first read PROJECT_HANDOFF_Target_Language_Purity_Analyzer.md,
codex_working_rules.md, and CURRENT_PROJECT_STATE.md. Then continue from the
current task state.
```

## GitHub Setup

The GitHub repository is:

```text
https://github.com/kauhsin/target-language-purity-analyzer
```

The local project was connected to that repository with:

```text
origin https://github.com/kauhsin/target-language-purity-analyzer.git
```

The GitHub repository was originally created with a README, so the remote was
not empty. The first local commit and the remote README commit had unrelated
histories. The project owner manually completed:

```text
git pull origin main --allow-unrelated-histories --no-rebase --no-edit
git push -u origin main
```

The first successful push placed the empty progress-record directory in the
remote repository via:

```text
项目进度记录/.gitkeep
```

During setup, HTTPS access to GitHub was unstable. Browser access worked, but
terminal/Codex `git` commands sometimes failed with:

```text
Failed to connect to github.com port 443
Error in the HTTP2 framing layer
```

After the project owner changed VPN/network conditions and retried in the
terminal, pull and push succeeded.

## Current Git Policy

Codex may help with Git operations after the project owner explicitly asks.
Before committing, Codex should state which files will be committed.

During this kickoff, the project owner specifically requested that only the
progress-record directory placeholder be pushed at first, not the handoff or
working-rules files.

At the end of this kickoff, the project owner approved creating and pushing:

```text
项目进度记录/000_project_kickoff.md
CURRENT_PROJECT_STATE.md
```

No implementation code should be added as part of this kickoff record.

## Next Task

The next recommended task is a new Codex conversation titled something like:

```text
Milestone 0 - Problem Definition and Output Protocol
```

That task should begin by reading:

```text
PROJECT_HANDOFF_Target_Language_Purity_Analyzer.md
codex_working_rules.md
CURRENT_PROJECT_STATE.md
```

Milestone 0 should be discussion-first. It should define:

- The exact problem statement.
- The label space.
- Input and output protocol.
- What `status` means.
- What the system may and may not claim.
- Success criteria for version 1.
- Which decisions belong to the linguistic project owner.
- What should eventually be written into formal project documentation.

Milestone 0 should not start data collection, write training scripts, or build a
model unless the project owner explicitly approves implementation for that
milestone.

