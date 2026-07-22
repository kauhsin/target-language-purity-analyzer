# 002 V1 Design Decisions

Date: 2026-07-18

## Summary

This record captures several accepted Version 1 design decisions:

- Use softmax three-way classification rather than sigmoid multi-label
  classification.
- Keep Version 1 as a pure-language classifier whose behavior is analyzed on
  mixed or uncertain inputs.
- Use deletion occlusion and mask occlusion for explanation.
- Do not include automatic replacement or counterfactual generation in Version
  1.
- Prefer training-corpus retrieval as auxiliary evidence over automatic
  replacement.
- Treat broader applications as future research windows, not current scope.

This record is public-safe and intentionally excludes personal plans, private
institutional details, internal data-access plans, and sensitive stakeholder
information.

## Project Philosophy

Version 1 does not train a new mixed-language detector.

Instead, it trains one classifier on pure-language corpora:

- Shanghainese
- Cantonese
- Mandarin

The classifier learns each language variety's distribution. During analysis,
the system intentionally examines inputs that may be mixed, unstable, or
influenced by a competing variety.

The central idea is:

```text
Train a pure-language classifier, then analyze how it behaves when an input
creates competition among learned language distributions.
```

The project analyzes:

- Which learned language distribution the full sentence is closest to.
- Which competing distribution is strongest.
- Which spans provide evidence for each language prediction.

It does not claim to:

- Automatically identify all language mixing.
- Compute the true percentage of each language inside a sentence.
- Assign absolute language labels to every word.

## Softmax vs Sigmoid

Two modeling options were considered.

## Option A: Softmax Three-Way Classification

Training labels:

```text
Pure Shanghainese → Shanghainese
Pure Cantonese    → Cantonese
Pure Mandarin     → Mandarin
```

The model learns:

```text
If forced to choose among the three pure-language distributions, which one is
this sentence relatively closest to?
```

Example output:

```text
Shanghainese: 0.63
Cantonese:    0.27
Mandarin:     0.10
```

These scores sum to 1. They should be interpreted as:

```text
relative affinity to each learned language distribution
```

not:

```text
percentage of language composition
```

Softmax is consistent with the Version 1 training data because each pure
training sentence has exactly one label.

## Option B: Sigmoid Multi-Label Classification

Sigmoid would ask independent questions:

- Does this sentence show Shanghainese features?
- Does this sentence show Cantonese features?
- Does this sentence show Mandarin features?

This is intuitively attractive because a sentence can contain evidence for more
than one variety.

However, Version 1 does not currently have reliable multi-label training data
such as:

- Shanghainese + Cantonese
- Shanghainese + Mandarin
- Cantonese + Mandarin

Therefore sigmoid is not aligned with the Version 1 training setup.

## Decision

Version 1 uses:

```text
softmax three-way classification
```

Rationale:

- It matches the pure-language single-label training data.
- It is simpler to train and evaluate.
- It supports the project philosophy: analyze mixed or uncertain inputs through
  competition among pure-language distributions.
- It is easier to explain carefully, as long as scores are not described as
  composition percentages.

Softmax vs sigmoid may be revisited as a future research question after Version
1 is working.

## Version 1 Explainability

Version 1 explainability is not intended to explain every internal mechanism of
the Transformer.

It is intended to explain:

```text
Which spans influenced the model toward Shanghainese, Cantonese, or Mandarin?
```

All explanations should rely on the same classifier.

## Deletion Occlusion

Deletion occlusion removes a candidate span and reruns the classifier.

If removing a span substantially lowers the score for a language, the span is
evidence for that language in the model's decision.

Example wording:

```text
Removing this span substantially lowers the Cantonese score, so this span is
evidence for the model's Cantonese judgment.
```

## Mask Occlusion

Mask occlusion replaces a span with `[MASK]` and reruns the classifier.

This keeps the span position visible while hiding its content. It may reduce
some deletion artifacts, but it can also introduce mask artifacts.

## Using Both Occlusion Methods

Deletion and mask occlusion should be compared for stability:

- If both lower the same language score, the evidence is more stable.
- If deletion lowers the score but masking does not, deletion may have changed
  the sentence structure too much.
- If masking lowers the score but deletion does not, the model may be sensitive
  to `[MASK]` artifacts.

Version 1 does not try to prove which perturbation is always correct. It uses
their agreement or disagreement as an attribution stability signal.

## Why Version 1 Does Not Do Replacement

Several replacement ideas were considered:

- Random token replacement.
- Neutral replacement.
- Counterfactual replacement.
- Semantic replacement.

These introduce difficult questions:

- How should the replacement's part of speech be controlled?
- How can syntactic naturalness be preserved?
- How can semantic equivalence be preserved?
- How can the replacement avoid introducing a new language bias?
- How can this be done reliably for low-resource varieties?

Automatic replacement would turn Version 1 into a counterfactual-generation
project. That is outside scope.

If reliable semantic resources, aligned phrase pairs, or parallel corpora become
available later, replacement can be revisited as a controlled future experiment.
It should not be part of the Version 1 automatic pipeline.

## Why Retrieval Is More Important Than Replacement in Version 1

Training-corpus retrieval fits the explanation goal better than automatic
replacement.

If the model finds that a span provides evidence for Cantonese, retrieval can
show similar examples from the training corpus. This makes the explanation more
inspectable without claiming that the span absolutely belongs to Cantonese.

Retrieval should be framed as:

```text
similar training examples supporting the model evidence
```

not:

```text
proof of absolute word-level language membership
```

Retrieval remains auxiliary. It does not replace occlusion attribution and does
not become a second trained explanation model.

## Version 1 Scope

Version 1 includes:

- One Transformer classifier.
- Softmax three-way classification.
- Sentence-level language-distribution scores.
- Deletion occlusion.
- Mask occlusion.
- Stability comparison between deletion and mask occlusion.
- Evidence grouped by language variety.
- Optional training-corpus retrieval.

Version 1 excludes:

- Automatic replacement.
- Automatic counterfactual generation.
- POS analysis as a required component.
- Dependency parsing as a required component.
- A second explanation model.
- Learner-facing correction or tutoring.
- Semantic equivalence checking.

## Later-Version Direction

A later version may explore learner-facing or writer-facing suggestions, but
only after Version 1 establishes reliable diagnosis and evidence attribution.

A possible later workflow:

```text
Detect competing-variety evidence
→ Use semantic or parallel resources to find candidate target-variety alternatives
→ Substitute cautiously
→ Re-check with the Version 1 analyzer
→ Present as a candidate suggestion, not a guaranteed correction
```

This is not a Version 1 goal.

## Broader Research Window

The same framework may later generalize beyond Sinitic varieties:

- Second-language influence.
- Regional variety consistency.
- Register or style consistency.
- Diachronic style shift.
- Authorship or stylometric studies.

These are future research windows only. They should not expand Version 1.

High-stakes applications require separate validation, domain expertise, and
ethical constraints. The system should not make legal, authorship, or
high-stakes claims from Version 1 outputs.

## Current Milestone Implication

Milestone 0 should explicitly record these Version 1 design commitments:

- Softmax, not sigmoid.
- Pure-language classifier, not mixed-language detector.
- Occlusion-based evidence, not token-level language labels.
- Deletion and mask occlusion as complementary perturbations.
- No automatic replacement in Version 1.
- Retrieval as auxiliary evidence.
- Broader applications as future work, not current implementation scope.

