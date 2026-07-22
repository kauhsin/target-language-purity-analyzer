# Milestone 0 Working Decision Log

Date: 2026-07-20

Status: Completed decision record. This file preserves the detailed reasoning
and accepted decisions from the Milestone 0 design discussion. The concise,
authoritative specification is
[`MILESTONE_0_PROBLEM_DEFINITION_AND_OUTPUT_PROTOCOL.md`](../milestones/milestone_0/MILESTONE_0_PROBLEM_DEFINITION_AND_OUTPUT_PROTOCOL.md).

## Evaluation Data Categories

Future development and evaluation data must distinguish at least three types
of sentences:

1. **Target-specific sentences** — sentences with relatively clear evidence
   for the intended language.
2. **Cross-variety compatible sentences** — sentences that can be fully
   accepted in two or more of the project languages. These are not mixed
   sentences merely because their written form is shared.
3. **Mixed or mismatched sentences** — sentences containing competing-language
   evidence that may justify a user-facing warning.
4. **Orthographic-variation cases** — equivalent or compatible target-language
   expressions written with different attested, community, or individual
   character choices. These cases test whether spelling conventions create
   false warnings or unstable explanations.

For cross-variety compatible sentences, evaluation must check:

- whether the system produces false mismatch warnings;
- whether the model systematically suppresses the score of another acceptable
  target language;
- whether large score gaps reflect corpus imbalance, source bias, or model
  overconfidence rather than a genuine target-language mismatch.

The false-warning rate is the primary product concern. Score gaps across
acceptable languages are an important diagnostic, but acceptable languages do
not need to receive identical scores.

## Data Strategy Requirements

Data balance must be assessed across more than sentence counts. The project
must compare and document:

- the amount of data in each class;
- sources and the number of independent sources;
- genre, register, and writing style;
- topics;
- sentence length and formatting;
- punctuation, scripts, character variants, and transcription conventions;
- internal diversity across authors, speakers, documents, and settings;
- time period, region, and generation where known;
- label quality and annotation confidence;
- duplicates, near-duplicates, and repeated templates;
- the proportion of synthetic, translated, or automatically transformed text.

The data strategy should reduce the risk that a model classifies language by
source, topic, formatting, or another shortcut. Equalizing class counts alone
is not sufficient. The project should not discard most higher-resource data by
default merely to match the smallest class. Sampling, class weighting,
learning curves, and balanced evaluation sets remain options to compare after
the real data has been audited.

The three language corpora do not need identical content or parallel
translations. They should be made reasonably comparable in naturalness,
register, source type, sentence length, and time period. Shanghainese is the
resource constraint and therefore defines the feasible corpus profile;
Cantonese and Mandarin sources should be selected to match that profile where
possible. Unavoidable differences must be documented and evaluated separately
by source or relevant data slice.

Each language should use multiple genuinely independent sources where
possible, but Milestone 0 does not set an arbitrary minimum such as three
sources per class. Independence, diversity, and source concentration matter
more than the raw number of download locations. The project should record each
source's contribution, detect copied or derived sources, report per-source
performance, and use a held-out-source evaluation when the available data can
support it. If a language has only one usable source, development may continue
with that limitation stated clearly and with narrower claims.

Shanghainese will anchor both the feasible corpus profile and the initial data
budget. The project will first determine how many qualified, diverse
Shanghainese sentences are available, then select Cantonese and Mandarin data
that match its source types, register, and quantity as closely as practical.
This is a sourcing order, not separate language-by-language model training; the
three-way classifier will still train on all three classes together.

Initial training experiments will use equal class sizes and nested data scales
to build learning curves. The exact scales will depend on the usable
Shanghainese training split. Validation and test data must be reserved before
creating these training subsets, and the subsets should be nested so that a
larger experiment adds data rather than replacing all examples. Higher-resource
Cantonese and Mandarin data will remain in the audited data pool and may later
support a controlled rotating balanced-sampling comparison.

Every candidate source must receive a usage classification, even for personal
or academic research. The source registry should distinguish at least:

- commercially usable;
- research-only;
- permission required;
- unclear.

Public accessibility is not treated as permission for model training,
redistribution, or commercial use. The registry should preserve the owner,
source URL, access date, license or terms, permitted research and commercial
uses, model-training status, redistribution and modification rights, privacy
risks, notes, and review status.

Sources with a clear commercial-use path are preferred for the core corpus.
Exceptionally valuable research-only data may be used for a clearly identified
research stage when its use is lawful and its restrictions are documented. A
model trained on such data must not be presented as commercially reusable by
default. Sources with unclear rights remain quarantined until reviewed.

Cross-class identical or near-identical text requires special handling. A
shared expression must not automatically be treated as dirty data, forced into
one exclusive language, or mistaken for ordinary leakage.

## Sentence-Level Reference Data

A reference sentence may be short or depend on some conversational context if
it is still a natural example of the target language. Short and low-information
sentences must not dominate a corpus. Their frequency and length distribution
should be measured and kept reasonably comparable across languages. A 5%–10%
range may be used as an initial audit prompt, not as an approved hard limit.

The target register is **natural conversational writing**: text that a person
could naturally say in everyday interaction and that also works as a useful
written input. The core corpus should avoid both extremes:

- formal, stiff, or written-only language that people would not normally say;
- transcription-heavy speech dominated by backchannels, false starts,
  repetitions, fillers, fragments, or other features unlikely to represent the
  project's written user inputs.

This does not justify removing genuine colloquial grammar, sentence-final
particles, dialect vocabulary, or other real target-language features. The
same register standard must be applied across Shanghainese, Cantonese, and
Mandarin so that the classifier does not learn formality or transcription style
as a shortcut for language.

Dialect and conversational register are separate dimensions: a dialect is not
synonymous with casual speech, and conversational language is not synonymous
with dialect. Data review must not remove genuine variety features merely to
make the text look more like standardized writing.

## Practical Human Review

The core training corpus does not require formal sentence-by-sentence
`accept`, `reject`, and `uncertain` annotation. For this small project, the
project owner will inspect every candidate source at a high level, scan its
contents, and exclude clearly unsuitable material during cleaning. Sources
with systematic problems may be rejected as a whole or handled with a
documented cleaning rule.

Raw source files must remain immutable. Excluding a sentence means omitting or
marking it in the processed dataset, not destroying the original record. Where
practical, the pipeline should preserve exclusion counts and coarse reason
categories without requiring a detailed manual annotation for every rejected
sentence.

Data will be maintained in three logical layers:

1. immutable raw source material;
2. cleaned data with documented technical processing;
3. selected data used for training, development, or evaluation.

Punctuation must be preserved when it carries sentence structure, meaning,
tone, or interactional information. The model input should not remove commas,
question marks, exclamation marks, sentence-final punctuation, or other useful
marks by default. Purely typographic variation should be normalized
consistently across all three languages, including full-width versus half-width
forms and equivalent Chinese versus Western punctuation forms where the
mapping is safe.

Original punctuation remains available in the raw text. The normalized form
must use a documented policy. Potentially meaningful distinctions—such as
repeated punctuation, ellipsis length, quotation structure, decimal points,
hyphens, and punctuation inside identifiers—require explicit rules rather than
blind replacement. Punctuation distributions should be audited by language and
source so that the classifier does not use source formatting as a shortcut.

Metadata requirements will be finalized during the data audit rather than
exhaustively specified in Milestone 0. Source-level metadata and inherited
defaults should be used wherever possible. Sentence-level fields such as
original script or conversion status need explicit values only when they differ
from the source default or when the distinction is required for audit and
reproducibility.

Detailed sentence-level human judgments are reserved for smaller development,
challenge, and final evaluation sets, where label quality directly determines
the validity of the reported results.

The project owner has the linguistic expertise to conduct routine review and
the main judgments for Shanghainese, Mandarin, and Cantonese. A Cantonese
native speaker may proofread a small final evaluation set when practical, but
external annotation is not a dependency for routine development. Any use of a
single principal reviewer and any optional external review should be reported
transparently in academic or public results.

## Solo-Project Feasibility

Version 1 is a small, useful, individually led project. Data, evaluation, and
model design should not assume an annotation team or a large operational
pipeline. When a proposed dataset or evaluation task cannot reasonably be
completed by one person in a short focused work period, the project should
first reduce its scope, use a smaller high-value sample, or redesign the task.
Exceptionally valuable work may still be considered after an explicit
importance-versus-cost discussion; it is not included by default.

A reference sentence does not need a distinctive or exclusive feature of its
target language. It may also be acceptable in another supported language. The
full training corpus will not require manual cross-variety compatibility
labels; a smaller curated evaluation set will cover that phenomenon.

## Orthographic Variation

The project must not treat a difference in writing convention as sufficient
evidence of a target-language mismatch. The reference data may include:

- variants documented in dictionaries, research, or reliable corpora;
- community spellings that are common in real use;
- individual or creative spellings observed in real data.

The project aims to expand coverage when new variants are found, but Version 1
cannot guarantee recognition of every individual or creative spelling.

Original text must always be preserved. In addition, confirmed orthographic
variants should be mapped to a shared internal analysis form before model
training and inference. This internal normalization prevents the classifier
from treating a source-specific spelling convention as language evidence. It
does not claim that only one public spelling is linguistically correct.

Normalization must be lexical and context-sensitive. For example, a confirmed
community spelling in a phrase may map to the same lexical form as another
orthography, but the same character must not be replaced globally when it has
an unrelated ordinary meaning elsewhere. Every mapping should preserve its
source, evidence, review status, and link to the original span.

The same normalization component must be used for training, development,
testing, and user inference. Reports should quote the user's original form,
while model analysis uses the normalized form. Span offsets or an equivalent
mapping must connect explanations back to the original input.

The working Version 1 approach is a reviewed **orthographic normalization
lexicon** combined with limited context-sensitive rules. Confirmed phrases may
be mapped one by one; recurring patterns may become rules only when their
contexts and exceptions can be stated safely. Unknown cases remain unchanged
or are queued for review. Training a separate normalization or segmentation
model is outside the current scope. The exact matching algorithm, rule order,
exception handling, and lexicon schema remain implementation questions.

An unseen form must not enter the review queue merely because it is out of
vocabulary or unfamiliar. It may be an orthographic variant, a new or rare
word, a name, place, brand, ordinary tokenizer decomposition, typo, or input
error. A candidate variant requires concrete evidence, such as repeated
correspondence across sources or expert identification. Version 1 prioritizes
precision: it is safer to miss a variant temporarily than to normalize a valid
new word or proper name incorrectly.

> **Unknown does not mean orthographic variant.**

Three different segmentation layers must not be conflated:

1. phrase and context matching for orthographic normalization;
2. the pretrained Transformer's character or subword tokenizer;
3. human-readable word spans used for occlusion explanations.

They may use different boundaries. Version 1 should investigate existing
Chinese and Cantonese segmentation resources, tokenizer coverage, and offset
mapping, but it does not commit to building a new dialect tokenizer.

For Version 1 occlusion, the explanation spans begin with the complete,
non-overlapping output of a selected Chinese word segmenter. Every segmented
word is eligible for testing, including a one-character item when the segmenter
treats it as a word. Version 1 does not add a separate character scan or
arbitrary overlapping multiword n-grams.

Reviewed Shanghainese, Cantonese, and, where useful, Mandarin lexical resources
may add a small number of alternative spans when the base segmenter splits a
confirmed lexical item incorrectly. Overlap should remain exceptional and
evidence-based; the user-facing result should suppress redundant spans. These
segmentation lexicons remain separate from the orthographic normalization
lexicon.

Dictionary PDFs, existing word lists, or other lexical sources may support the
segmentation lexicons after extraction, license review, cleaning, and
linguistic review. Extracted headwords must retain source and usage-rights
metadata. Coverage must be measured rather than assumed from the dictionary's
size.

A small, manually reviewed segmentation audit set is required. It should cover
important Shanghainese and Cantonese words, competing-language items,
ambiguous boundaries, common function words, and normalized orthographic
variants. Version 1 will compare available segmentation tools and dictionary
extensions on this set; it will not train a new dialect segmenter.

Orthographic normalization occurs before the pretrained model tokenizer. The
normalizer must retain an alignment between original and normalized character
spans so that a user can see the form they entered, the confirmed internal
mapping when relevant, and the resulting evidence expressed against their
original text.

Confirmed variant records should preserve provenance and, when available,
distinguish different relationships, such as alternative character choices for
the same lexical item and community phonetic spellings that borrow a character
for its sound. The exact terminology and schema remain to be designed with
linguistic review.

Writer or producer metadata may be recorded when it is provided by the source,
legally usable, relevant, and ethically appropriate. Linguistic background
must not be guessed from the text. Source-level descriptions are preferable to
unnecessary personal profiling.

Orthographic variants of the same or nearly identical sentence must be grouped
during data splitting. A variant must not be placed in training while its
near-duplicate counterpart is placed in testing and then presented as evidence
of generalization.

The preprocessing order must place deterministic, versioned orthographic
normalization and duplicate grouping before the final train/validation/test
split. Any transformation learned from corpus statistics must be fitted on the
training split only. During development, the normalization lexicon and rules
may evolve through review of training and development data. Every change must
create a new resource version and trigger reprocessing, duplicate regrouping,
and split validation as needed. Normalization resources and split assignments
must be frozen before final test evaluation.

Group-aware splitting must use the smallest meaningful unit that prevents
obvious dependency leakage; a group is not mechanically an entire book,
author, or corpus. A phrasebook or textbook may be divided by lesson, dialogue,
scene, exercise, template family, or another internal unit when those parts are
reasonably independent. Truly independent entries may be split individually
after duplicate and template checks. Speaker- or author-level isolation is
desirable only when metadata and data volume make it practical; it is not a
hard requirement for low-resource varieties.

The same conceptual grouping rules and comparable strictness must be applied
to Shanghainese, Cantonese, and Mandarin. A richer Mandarin data pool must not
receive a stricter core split policy that would make cross-language results
incomparable. Additional stricter analyses may be run when a language has more
metadata or sources, but they must be reported as separate diagnostics rather
than mixed into the main comparison.

The main in-distribution split may contain independent units from the same
sources in training, validation, and test. This measures performance on new
texts from familiar source types. A held-out-source evaluation answers the
harder question of generalization to a new source and is included only when the
available source diversity can support it. Results must state which setting was
used.

Only literal or algorithmically similar overlap can be detected automatically.
Exact normalized intersections across language corpora can be found with text
matching, and near-duplicate methods can flag candidates. These procedures
cannot determine every sentence that speakers would accept in multiple
languages. The full training corpus will not receive exhaustive cross-variety
compatibility labels; that judgment remains limited to a small curated
development and evaluation set. A focused shared-expression evaluation set is
a required Version 1 evaluation slice.

The selected main training data will keep one normalized copy of each exact
sentence per language while preserving its occurrence count and independent
source list as metadata. This favors pattern diversity over corpus frequency.
Frequency-aware experiments are deferred. When the same normalized text occurs
in more than one language corpus, one labeled copy may be retained for each
language; cross-language copies are not deduplicated against one another. They
must share a cross-language overlap group and remain in the same split. If they
appear in validation or testing, they must be analyzed as shared or ambiguous
cases rather than treated as ordinary exclusive-label accuracy examples.

Version 1 user input is limited to Simplified Chinese text. Traditional Chinese,
IPA, romanization, pinyin, and English code-switching are outside the initial
input scope. Their support is a future extension.

The core training corpus will use human-produced text or text that has received
credible human review. When authorship is uncertain, the project owner will
review the source and the text before deciding whether it is eligible. A
dictionary or similarly curated source may count as human-reviewed even when
the individual writer is not identified.

High-quality Traditional Chinese source material may be converted to Simplified
Chinese for Version 1. The original Traditional Chinese text must be preserved,
the converted form must remain linked to it, and the conversion must receive
linguistic review. Original and converted counterparts are one data item for
splitting and leakage control, not independent examples.

Numbers require an explicit normalization policy across training, development,
evaluation, and user input. Original forms must remain available. The policy
must distinguish contexts such as quantities, dates, decimals, identifiers,
and phone numbers instead of assuming that every Arabic numeral has one safe
Chinese-character conversion.

Before implementation, the project should review established practices and
available libraries for number normalization. A library should be adopted only
after its behavior has been validated for the project's Chinese varieties and
tokenizer; no single universal normalization method is assumed in advance.

## Warning and Abstention Principle

The highest softmax score does not by itself determine whether a text should
receive a target-language mismatch warning. A competing language may have the
highest score even when the text is acceptable in the intended language.

The working user-facing status for cases without sufficient warning evidence
is:

> **No clear mismatch found.**

This means that the system did not find a sufficiently specific and stable
reason to warn the user. It does not prove that the text is correct, natural,
or exclusively part of the target language.

If the model produces unusual sentence-level scores but cannot support a clear
warning, the system should abstain or return an uncertain result rather than
overstate its conclusion.

> **When the system cannot explain the warning clearly, it should usually
> abstain.**

The final product wording will be refined later in plain, natural English and
Chinese.

## Role of Explainability

Explainability is part of the Version 1 Engine and may become one condition in
the final warning protocol. A warning may eventually consider:

- unusually low support for the target language;
- strong support for a competing language;
- specific competing-language evidence that can be localized to a word or
  phrase;
- attribution that is reasonably stable across deletion and mask occlusion;
- thresholds validated on target-specific, cross-variety compatible, and
  mixed or mismatched development data.

These are principles, not approved numerical thresholds or a finalized rule.

For a language `L` and candidate span `s`, Version 1 uses the following sign
convention:

```text
deletion_evidence(L, s)
    = original_score(L) - score_after_deleting_s(L)

mask_evidence(L, s)
    = original_score(L) - score_after_masking_s(L)
```

A positive value means that the span increased the model's support for that
language: removing or masking it lowered the language score. A negative value
means that removing or masking the span raised the language score. Zero or a
near-zero value means that the perturbation had little measured effect.

Substantial, stable negative deltas for the intended language are relevant to
the target-language assessment and must not be hidden by default. In
user-facing language, describe the effect directly—for example, “this span
lowered the model's support for Shanghainese”—rather than presenting an
unexplained label such as `negative evidence`. The same span may also show
positive evidence for a competing language, and one concise evidence item may
report both effects.

For the Version 1 use case, stable target-language score suppression is the
primary user-facing span signal. Evidence ranking and presentation should first
help the user find words that lowered the model's support for the intended
language. Positive target-language evidence remains available, while positive
competing-language evidence helps identify the direction of the competing
pattern. The exact ranking formula remains a development decision.

A negative target-language delta without clear positive evidence for another
supported language indicates that the span suppressed the target score, but it
does not identify the source of the mismatch. Such cases require cautious or
uncertain interpretation.

Deletion and mask evidence remain separate Version 1 results. They must not be
silently combined into one score before a validated combination rule exists.
These values describe changes in the classifier's output; they are not
probabilities that a span belongs to a language.

The machine-readable result must make the delta nature explicit. Original
language scores are stored once at the sentence level. Each span's deletion and
mask result should distinguish the scores after occlusion from the calculated
evidence deltas, rather than presenting a negative delta where a reader might
mistake it for a softmax probability. Exact field names remain part of the
final schema design.

The canonical model-run record stores original and post-occlusion scores.
Evidence deltas are calculated from those scores rather than stored as a third
independent source of truth. User-facing results primarily show the calculated
delta; detailed post-occlusion scores may remain available for debugging and
research reports.

Occlusion is not independent linguistic evidence. It is an explanation derived
from the same classifier and inherits the classifier's data and modeling
biases. It can support a warning by showing which spans affected the model, but
it cannot prove that a span belongs exclusively to another language or that
the intended language does not accept it.

The absence of a strong occlusion result means that the system lacks a clear,
localized reason for a warning. It is not positive proof that the sentence is
valid in the target language.

## Valid Input and Available Analysis

The Version 1 runtime request requires both `text` and `target_language`.
`target_language` must be one of Shanghainese, Cantonese, or Mandarin. Version
1 is a target-language analysis task, not an open language-identification mode.
Comparing the three languages without an intended target is a possible future
Engine extension and is outside the approved Version 1 scope.

Abstaining from a warning must not mean withholding the analysis. For every
valid input, the Engine should still make the available model results visible,
even when the final assessment is uncertain. These results may include:

- the scores for all three supported languages;
- the score for the intended language;
- the language with the highest model score;
- evidence found for relevant words or phrases;
- score changes produced by deletion and mask occlusion.

Public-facing text may call the softmax outputs **language scores** instead of
requiring users to understand the term *softmax*. The interface must still
explain that these are relative model scores, not percentages of language
composition.

All three language scores should be returned without a redundant standalone
`highest_other_score`. In the user-facing presentation, show the intended
language first and order the two other languages from higher to lower score.
The highest other language can still be derived internally when warning logic
needs it, but it is not a separate required display field.

The target-language assessment is sentence-level. Evidence spans do not receive
categorical `match`, `mismatch`, or absolute language-membership labels. Span
outputs describe and rank how masking or deleting a span changes each language
score. The exact sentence-status names and any user-facing evidence-strength
levels remain to be validated rather than fixed prematurely.

User-facing `weak`, `medium`, or `strong` evidence bands are deferred to a
later product stage. Version 1 will preserve continuous score changes, rank
evidence, and report attribution agreement or instability without inventing
unvalidated categorical strength thresholds.

Version 1 does not require the user to identify a suspect word or provide a
`focus_span`. The explanation module automatically tests every non-punctuation
word produced by the selected segmenter, then reports the strongest relevant
evidence. Transformer subword tokens are not assumed to be the same as these
human-readable words. The exact segmenter selection and ranking method remains
to be designed and evaluated.

Empty, punctuation-only, corrupted, unsupported, or otherwise invalid inputs
may return an input error instead of meaningless language scores. The exact
validation rules remain deferred.

## Deferred Decisions

The following remain open until suitable development data exists:

- numerical target-score, competitor-score, and margin thresholds;
- the minimum evidence strength needed for a warning;
- how deletion and mask evidence should be combined;
- calibration methods;
- the final status set and final user-facing wording;
- the exact metrics and pass criteria for each evaluation category.

## Version 1 Success Principles

Milestone 0 does not set unsupported numerical performance thresholds. Version
1 should meet the following high-level conditions before it is presented as a
successful first system:

1. **Usable data** — the three reference corpora pass the approved provenance,
   licensing, naturalness, comparability, normalization, deduplication, and
   split checks.
2. **Reproducible pipeline** — the audited data, preprocessing, model training,
   evaluation, and outputs can be rebuilt from recorded versions and
   configurations.
3. **Meaningful sentence classifier** — on target-specific held-out reference
   sentences, the classifier performs materially better than chance and
   generally gives the intended language stronger support than the other two.
   Shared or cross-variety-compatible sentences are evaluated separately and
   are not required to have one dominant language score.
4. **Controlled sensitivity** — when a valid target-language sentence receives
   a controlled, linguistically reviewed replacement from another supported
   language, the target-language score should move downward and the relevant
   competing-language score should move upward in aggregate.
5. **Traceable explanation direction** — the explanation module should be able
   to test whether the controlled replacement produces stable target-score
   suppression and competing-language support. Detailed localization metrics
   and pass thresholds are deferred to explanation development.
