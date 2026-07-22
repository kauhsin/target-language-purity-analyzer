# Milestone 0: Problem Definition and Output Protocol

Status: Approved design specification
Date: 2026-07-20

## Plain-Language Definition

In this project, **purity** describes how well a text matches its intended
language. It does not suggest that languages themselves are pure or should be
free from outside influence.

在这个项目中，“纯度”是指一段文本是否符合它原本想使用的语言。它并不意味着语言本身是纯粹的，也不认为语言不应该受到其他语言的影响。

The formal technical concept is **target-language consistency**: the degree to
which a text provides evidence consistent with its intended language variety,
relative to the project's reference data, and whether it also provides
evidence associated with competing varieties.

The system does not enforce a prescriptive standard, treat contact between
languages as contamination, or claim that every expression belongs to only one
language. A written sentence may be acceptable in more than one variety.

## Version 1 Scope

Version 1 is an explainable Engine for short written text in Simplified Chinese.
It supports three intended target languages:

- Shanghainese;
- Cantonese;
- Mandarin.

The Engine uses one three-way Transformer classifier. It returns relative
language scores and automatically examines human-readable word spans through
deletion and mask occlusion. It does not train a `Mixed` class, require
token-level language labels, rewrite the input, or decide who wrote it.

Open language comparison without an intended target is technically possible
but requires a separate interpretation protocol. It is a future Engine
extension, not part of Version 1. Traditional Chinese input, IPA, pinyin,
romanization, and English code-switching are also outside Version 1.

## Runtime Input

The minimum valid request is:

```json
{
  "target_language": "Shanghainese",
  "text": "我想吃饭。"
}
```

Both fields are required. `target_language` is one of `Shanghainese`,
`Cantonese`, or `Mandarin`. The user does not need to identify a suspect word;
span analysis is automatic.

Empty, punctuation-only, corrupted, unsupported, or otherwise invalid input
may return an input error. Exact validation and length limits will be set during
development.

## Interpretation Rules

Language scores are relative model support, not percentages of linguistic
composition and not proof of language ownership. The intended language does
not always need the highest score for a sentence to be acceptable in that
language.

For example, a written expression may be fully acceptable in both Shanghainese
and Mandarin. A higher Mandarin score alone does not justify a Shanghainese
mismatch warning. A warning requires sufficiently clear and stable evidence.
When that evidence is absent, the working user-facing result is:

> **No clear mismatch found.**

This means that the system found no clear reason to warn the user. It does not
prove that the text is correct, natural, or exclusive to the target language.
When the system cannot explain a warning clearly, it should usually abstain
from the warning while still returning the available analysis.

## Output Protocol

Every valid request returns:

1. the original and normalized text;
2. all three language scores, with the intended language identified;
3. a sentence-level target-language assessment;
4. automatically generated span evidence;
5. enough provenance and version information to reproduce the result.

The intended language is shown first; the other two scores may be ordered from
higher to lower. The interface calls them **language scores**. It does not add
a redundant standalone “highest competing score” field.

The exact sentence-status names and final user-facing wording will be validated
with development data. They are deliberately not fixed in Milestone 0.

The following is an illustrative machine-readable contract, not a frozen
implementation schema:

```json
{
  "request": {
    "target_language": "Shanghainese",
    "original_text": "今天去饮茶。",
    "normalized_text": "今天去饮茶。"
  },
  "language_scores": {
    "Shanghainese": 0.28,
    "Cantonese": 0.61,
    "Mandarin": 0.11
  },
  "assessment": {
    "status": "provisional_status_name"
  },
  "evidence": [
    {
      "original_span": "饮茶",
      "normalized_span": "饮茶",
      "original_offsets": [3, 5],
      "deletion_scores": {
        "Shanghainese": 0.42,
        "Cantonese": 0.39,
        "Mandarin": 0.19
      },
      "mask_scores": {
        "Shanghainese": 0.39,
        "Cantonese": 0.43,
        "Mandarin": 0.18
      }
    }
  ],
  "versions": {
    "model": "model_version",
    "normalization": "normalization_version",
    "segmentation": "segmentation_version"
  }
}
```

Canonical records store the original scores and scores after occlusion.
Evidence deltas are derived rather than maintained as a third independent
source of truth.

## Evidence Contract

For language `L` and span `s`:

```text
deletion_evidence(L, s)
    = original_score(L) - score_after_deleting_s(L)

mask_evidence(L, s)
    = original_score(L) - score_after_masking_s(L)
```

A positive delta means that the span increased model support for that language.
A negative delta means that the span lowered model support for that language.
Negative target-language evidence is especially relevant because it helps the
user locate what reduced support for the intended language. Positive evidence
for a competing language can show the direction of the competing pattern.

Deletion and mask results remain separate until a combination rule has been
validated. Evidence is reported as an effect on the classifier, never as proof
that a word belongs exclusively to a language. Occlusion is derived from the
same classifier and is not independent linguistic evidence.

Evidence spans begin with non-overlapping words from a selected mature Chinese
segmenter. Reviewed dialect lexical resources may add a small number of
evidence-based alternatives when a confirmed Shanghainese or Cantonese word is
split incorrectly. Arbitrary character scans and arbitrary n-grams are outside
Version 1. Punctuation remains in model input but is not normally occluded as a
standalone evidence span.

The Transformer tokenizer, orthographic matching, and user-facing word
segmentation are separate layers. Explanations must map normalized spans back
to the user's original text.

## Reference Data Standard

Formal technical documents use **target-language reference corpus**, not
“pure-language corpus.” A reference sentence:

- comes from a traceable, legally reviewable target-language source;
- was produced for that language or credibly reviewed as that language;
- is natural conversational writing: sayable in everyday speech while still
  useful as written input;
- does not need an exclusive or highly distinctive language feature;
- may also be acceptable in another supported language.

Dialect is not the same as casual speech, and casual speech is not the same as
dialect. Data cleaning must preserve genuine dialect grammar, vocabulary,
particles, ellipsis, and natural sentence patterns. The corpus should avoid
both written-only formality and transcript-heavy fragments or fillers.

Core training data is human-produced or credibly human-reviewed. AI-generated
text may be used in evaluation but not as core training reference data.

## Data Governance and Balance

Shanghainese is sourced first and anchors the feasible register, source profile,
and initial class size. Cantonese and Mandarin should match it as closely as
practical. The three classes train together.

Initial experiments use equal class sizes and nested learning curves. All
qualified higher-resource data remains in an audited pool for later controlled
experiments. Balance is assessed not only by count, but also by source, genre,
register, topic, length, formatting, time period, internal diversity, label
quality, and duplication.

Multiple independent sources are preferred, but there is no arbitrary minimum
number. The same cleaning and split principles apply to all three languages.
Source differences and concentration must be reported; held-out-source testing
is added only when source diversity makes it meaningful.

Every source receives a usage classification: commercially usable,
research-only, permission required, or unclear. Public accessibility is not a
license. Source rights, privacy risks, provenance, transformations, and review
status must be recorded. Valuable lawful research-only data may support a
clearly identified research model, but that model cannot be presented as
commercially reusable by default.

Data has three logical layers:

1. immutable raw material;
2. cleaned, traceable data;
3. selected training and evaluation data.

Meaningful punctuation is preserved. Safe typographic differences such as
equivalent full-width and half-width punctuation are normalized consistently.
Original text is never overwritten.

## Orthography, Deduplication, and Splitting

Confirmed orthographic variants map to a shared internal form through a
reviewed lexicon and limited context-sensitive rules. Rules must not replace a
character globally when it has unrelated uses. Unknown forms remain unchanged:

> **Unknown does not mean orthographic variant.**

The lexicon may evolve during training and development, with versioning and
reprocessing. It is frozen before final testing.

The processing order is:

```text
preserve raw data
→ deterministic normalization
→ duplicate and dependency grouping
→ final train / validation / test split
→ model tokenization
```

Within one language, the selected training data keeps one normalized copy of
an exact sentence and records occurrence and source counts. If the same
normalized sentence occurs in different languages, one labeled copy remains in
each language. Those copies share an overlap group and must enter the same
split. In evaluation they are treated as shared or ambiguous cases, not
ordinary exclusive-label accuracy examples.

Group-aware splitting uses the smallest meaningful dependency unit, such as a
conversation, lesson, scene, template family, duplicate group, or conversion
group. It does not automatically place an entire book, author, or source in one
split. Validation supports iteration; the final test set remains sealed.

## Evaluation Plan

Evaluation separates at least:

- target-specific sentences;
- cross-variety compatible sentences;
- mixed or mismatched sentences;
- orthographic-variation cases;
- short or unclear cases;
- a small segmentation audit set;
- held-out-source cases when feasible.

The full training corpus does not require manual cross-variety labels. Exact
normalized overlaps can be detected automatically; broader shared acceptability
is covered by a small expert-reviewed challenge set.

## Version 1 Success Principles

Version 1 succeeds as a first Engine when:

1. its three reference corpora are usable, traceable, legally reviewed, and
   reasonably comparable;
2. its preprocessing, training, evaluation, and outputs are reproducible;
3. the classifier is meaningfully better than chance on held-out
   target-specific data, while shared cases are evaluated separately;
4. controlled, linguistically reviewed competing-language replacements lower
   target support and raise the relevant competing score in aggregate;
5. explanations can trace stable target-score suppression and competing
   support in those controlled cases.

Numerical thresholds, calibration, final status names, exact pass criteria,
and evidence-strength bands require development evidence and are deferred.

## Milestone 0 Exit

Milestone 0 is complete when this specification, its detailed decision record,
and the current project state are approved and synchronized. It authorizes
planning for Milestone 1, but it does not itself implement the data pipeline or
model.
