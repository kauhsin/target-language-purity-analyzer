# AI Target Language Purity System

An explainable AI system for analyzing whether a text matches its intended language.

In this project, **purity** describes that match. It does not suggest that languages themselves are pure or should be free from outside influence.

在这个项目中，“纯度”是指一段文本是否符合它原本想使用的语言。它并不意味着语言本身是纯粹的，也不认为语言不应该受到其他语言的影响。

The project examines both human-written and AI-generated text. Its goal is not to assign absolute language labels to individual words, but to identify evidence that influences the system's language-variety assessment.

Shanghainese, Cantonese, and Mandarin are the initial case study. The underlying approach is intended to remain applicable to other target language varieties.

## What the System Analyzes

Given a sentence and its intended target language variety, the system aims to:

- evaluate how consistently the sentence reflects the target variety;
- identify relevant competing-language evidence;
- highlight words or phrases that influence the assessment;
- communicate uncertainty without treating language boundaries as absolute.

This can support language-quality analysis for learner-produced, human-written, and AI-generated text.

## Current Status

The project is currently in **Phase 1 — AI Target Language Purity Engine**.

**Milestone 0: Problem Definition and Output Protocol is complete.** The project has defined its Version 1 scope, data principles, input/output protocol, evidence contract, and high-level success criteria.

No production model or complete analysis pipeline has been released yet. The next milestone is **Milestone 1: Dataset Engineering**.

## Documentation

- [Current Project State](CURRENT_PROJECT_STATE.md) — current progress and next milestone
- [Milestone 0 Specification](milestones/milestone_0/MILESTONE_0_PROBLEM_DEFINITION_AND_OUTPUT_PROTOCOL.md) — approved problem definition, data principles, and output protocol
- [Phase 1 Technical Handoff](PROJECT_HANDOFF_Target_Language_Purity_Analyzer.md) — detailed Version 1 decisions, constraints, and implementation plan

## Repository Status

This project is under active development. Documentation, experiments, evaluation results, and usage instructions will be added as each milestone is completed.

Milestone 0 documentation was completed on 2026-07-20.
