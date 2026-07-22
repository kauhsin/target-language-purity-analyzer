# Milestone 0 Completion Record

Date: 2026-07-20

## Outcome

**Milestone 0: Problem Definition and Output Protocol is complete.**

The project now has an approved Version 1 problem definition, scope boundary,
runtime input contract, output protocol, evidence contract, data strategy, and
high-level success criteria. No implementation code or data pipeline was
created during this milestone.

## Deliverables

- `milestones/milestone_0/MILESTONE_0_PROBLEM_DEFINITION_AND_OUTPUT_PROTOCOL.md` — concise approved
  specification;
- `项目进度记录/004_milestone_0_working_decisions.md` — detailed decision record;
- updated `README.md`, technical handoff, working rules, and current state.

## Main Decisions

- “Purity” remains the public project term, with an immediate definition that
  rejects prescriptive or contamination-based interpretations.
- Version 1 requires an intended target language and Simplified Chinese text.
- One three-way classifier returns relative language scores; it does not
  estimate language composition.
- Automatic deletion and mask occlusion show how word spans affect all three
  scores. Negative target-language evidence is a primary mismatch signal.
- Shared expressions may be valid in more than one language and must not be
  treated as errors solely because another language scores higher.
- Shanghainese anchors the initial data profile and size. Initial training uses
  balanced class sizes and nested learning curves.
- Data governance, orthographic normalization, duplicate grouping, and
  leakage-resistant splitting are required parts of Dataset Engineering.

## Deferred by Design

Numerical thresholds, calibration, final assessment names, exact segmentation
and normalization implementations, and metric pass values require development
data. Their deferral is explicit and does not block Milestone 0 completion.

## Next Milestone

The next milestone is **Milestone 1: Dataset Engineering**. It begins with a new
design discussion and requires explicit approval before implementation.
