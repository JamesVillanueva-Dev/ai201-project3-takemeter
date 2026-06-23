# TakeMeter Classification Project: Planning Document

## Community and Task

I chose the public r/gaming community. It is a good fit for a classification task because gaming conversations are not all doing the same thing: people share subjective takes, explain mechanics or news, ask for help, and post discoveries or trivia. The same game can appear in several different discourse modes, which makes the task more interesting than keyword matching. For example, an Elden Ring post could be a strategy question, a complaint about difficulty, an explanation of weapon scaling, or a surprising discovery.

This classifier will label public r/gaming posts or comments by the writer's main communicative purpose. I will classify the text itself, not the popularity of the post, the game being discussed, or whether I personally agree with it.

## Labels

I will use four labels: `opinion`, `explanation`, `seeking_assistance`, and `cool_fact`.

### `opinion`

A post is `opinion` when its main purpose is to express a personal judgment, preference, complaint, praise, prediction, or argument about a game, company, genre, mechanic, or gaming culture.

Examples:

- "Red Dead Redemption 2 is still the most immersive game ever made. The slower pace makes the world feel real instead of empty."
- "Battle passes have made multiplayer games feel like chores. I miss when unlocking cosmetics was actually fun."

### `explanation`

A post is `explanation` when its main purpose is to provide factual information, clarify how something works, summarize news, or explain a mechanic, update, technical feature, or industry event.

Examples:

- "In Baldur's Gate 3, advantage means the game rolls two d20s and uses the higher result, which is why positioning matters so much."
- "DLSS renders the game at a lower internal resolution and then upscales it with Nvidia's model, which can improve frame rate without changing every graphics setting."

### `seeking_assistance`

A post is `seeking_assistance` when its main purpose is to ask the community for help, recommendations, troubleshooting, advice, or a decision.

Examples:

- "I'm stuck on Malenia in Elden Ring. What build or strategy helped you finally beat her?"
- "What are some good couch co-op games for two people who liked It Takes Two but want something less platform-heavy?"

### `cool_fact`

A post is `cool_fact` when its main purpose is to share an interesting discovery, surprising detail, bit of trivia, unusual screenshot context, easter egg, or novelty meant to inform and entertain.

Examples:

- "The Game Boy that survived a Gulf War bombing still works and has been displayed at the Nintendo store."
- "In Skyrim, NPCs can react differently if you drop valuable items near them, and some will actually fight over who gets to keep them."

## Hard Edge Cases

The hardest boundary will be `opinion` vs. `explanation`. Many posts explain a fact in order to support a take. I will label by primary intent: if the post mainly teaches or clarifies something, it is `explanation`; if the facts are mainly evidence for a personal judgment, it is `opinion`.

Ambiguous example:

> "Starfield runs at 30fps on console because Bethesda prioritized simulation systems, but honestly that still feels unacceptable for a big release."

Decision rule: label this `opinion` because the factual setup exists to support the judgment that the frame rate is unacceptable.

Another hard boundary is `seeking_assistance` vs. `opinion`. A frustrated question can sound like a complaint, but if the writer is functionally asking for advice, I will label it `seeking_assistance`.

Ambiguous example:

> "This boss feels completely unfair. Is there any reliable way to dodge the second phase attack?"

Decision rule: label this `seeking_assistance` because the concrete request for a strategy is the main purpose.

A third hard boundary is `cool_fact` vs. `explanation`. Both can share information, but `cool_fact` is centered on novelty or surprise, while `explanation` is centered on understanding how something works.

Ambiguous example:

> "You can skip a section of Portal 2 by placing a portal behind the panel before the dialogue finishes because the trigger volume only checks your position."

Decision rule: label this `explanation` if the post focuses on the mechanism, but `cool_fact` if it is presented mainly as a surprising discovery. If still tied after reading the whole post, I will choose the label that best matches the final sentence because writers often end with their main point.

During annotation, I will keep a running note for difficult rows in the `notes` column. Each note will include the competing labels and the reason for the final decision, such as "opinion vs explanation; facts support complaint, labeled opinion."

### Documented Annotation Decisions

- "Bethesda announces yet another Skyrim port" was ambiguous between `opinion` and `explanation`. I labeled it `opinion` because "yet another" makes the post's main point a judgment about repeated Skyrim ports, not just a neutral announcement.
- "This is such a weird duo, but should I buy a SNES classic or a PS3 slim? [Nintendo] [Playstation] [help]" was ambiguous between `opinion` and `seeking_assistance`. I labeled it `seeking_assistance` because the writer is asking for buying advice.
- "Found a hidden shield worth 10 stars on the blue mark in the prison, anyone know what it's about? (Also free 10 stars)" was ambiguous between `cool_fact` and `seeking_assistance`. I labeled it `cool_fact` because the main value is sharing a hidden discovery, even though it includes a question.
- "When GameStop sells a used game, do the developers of that game get a fraction of money made from it?" was ambiguous between `explanation` and `seeking_assistance`. I labeled it `explanation` because it asks for factual clarification about an industry mechanism rather than personal advice.

## Data Collection Plan

I will collect public examples from r/gaming posts and comments. I can use the existing `cleaned_subreddit_gaming.csv` in this repo as a source pool, then manually review and label selected rows into a smaller final CSV with at least these columns:

- `text`: the post or comment text being classified
- `label`: one of `opinion`, `explanation`, `seeking_assistance`, or `cool_fact`
- `notes`: optional annotation notes, especially for difficult cases

My target is 200 labeled examples total, balanced as closely as possible:

- 50 `opinion`
- 50 `explanation`
- 50 `seeking_assistance`
- 50 `cool_fact`

I will annotate manually rather than relying on automatic labels. I will read each row in full before assigning a label. If the CSV row has both a title and a comment body, I will use the body as the primary text when the body stands alone; if the body depends on the post title for context, I will combine them as `Title: ... Comment: ...` in the `text` field.

If a label is underrepresented after reviewing 200 candidate examples, I will not accept a heavily imbalanced dataset. I will collect additional public r/gaming examples using targeted searches:

- `seeking_assistance`: "how do I", "any tips", "recommend", "what should I play", "help", "stuck"
- `cool_fact`: "did you know", "I just found", "easter egg", "detail", "turns out", "hidden"
- `explanation`: "because", "works by", "patch", "update", "mechanic", "for anyone confused"
- `opinion`: "I think", "overrated", "underrated", "best", "worst", "I hate", "I love"

If a label is still under 35 examples after 200 reviewed candidates, I will continue collecting until every label has at least 35 examples and no single label is more than 70% of the dataset. The preferred final distribution is still 50 examples per label.

## Evaluation Metrics

I will evaluate both the zero-shot baseline and the fine-tuned DistilBERT model with multiple metrics.

Overall accuracy will show the percentage of test examples classified correctly, which is useful as a quick summary but insufficient by itself. A model could look accurate by overpredicting a common label while performing poorly on rarer labels.

Macro F1 will be my primary single-number metric because all four labels matter. Macro F1 weights each label equally, so weak performance on `seeking_assistance` or `cool_fact` cannot be hidden by strong performance on `opinion`.

Per-class precision, recall, and F1 will show how each label behaves:

- Precision matters because a community tool should not dump unrelated posts into a category.
- Recall matters because missed help requests or missed informational posts would make the classifier less useful.
- Per-class F1 gives a compact view of whether each label is learnable.

The confusion matrix will be especially important because I expect the main failures to be specific label-pair confusions, especially `opinion` vs. `explanation` and `explanation` vs. `cool_fact`. The matrix will let me see whether errors are random or concentrated along those boundaries.

## Definition of Success

This classifier will be genuinely useful if it can organize r/gaming discussion modes better than a general zero-shot prompt and with reliable performance across all labels.

Minimum acceptable success:

- Fine-tuned model macro F1 is at least 0.70.
- Fine-tuned model accuracy is at least 0.70.
- No individual label has F1 below 0.60.
- Fine-tuned macro F1 is at least 0.05 higher than the zero-shot baseline.

Good enough for deployment in a real community tool:

- Fine-tuned model macro F1 is at least 0.75.
- `seeking_assistance` recall is at least 0.75, because missing help requests would be a visible failure for users.
- No single confusion pair accounts for more than half of all errors. If one pair dominates, the label boundary needs more work before deployment.

If the model misses these thresholds, I will treat the result as a useful prototype but not a deployable classifier. The next fix would be either collecting more examples for the weakest label pair or tightening the label definitions if error analysis shows annotation inconsistency.

## AI Tool Plan

### Label Stress-Testing

Before final annotation, I will give an AI tool the four label definitions and the edge-case rules above. I will ask it to generate 5-10 r/gaming-style posts that sit between two labels, especially `opinion`/`explanation`, `seeking_assistance`/`opinion`, and `cool_fact`/`explanation`.

I will manually classify those generated boundary posts using my definitions. If I cannot classify them cleanly, I will revise the relevant label definition or decision rule before annotating the 200-example dataset. I will not add generated posts to the training data; they are only for testing the clarity of the spec.

### Annotation Assistance

For the initial 200-example dataset, I plan to label manually without LLM pre-labeling. This keeps me close to the data and reduces the risk that an AI tool's assumptions shape the dataset.

If I later use an LLM to pre-label an additional batch, I will add an `ai_suggested_label` column and still manually review every example before finalizing the `label` column. I will disclose that workflow in the README AI usage section, including which tool was used and what kinds of labels I corrected.

### Failure Analysis

After evaluating the fine-tuned model, I will give an AI tool a list of misclassified test examples with true label, predicted label, and text. I will ask it to identify possible patterns, such as sarcasm, short comments, missing context from image posts, or repeated confusion between two labels.

I will verify any suggested pattern myself by rereading the wrong predictions and checking the confusion matrix. In the README, I will report only patterns that I can confirm from the actual examples, and I will mention any AI-suggested explanations that I rejected.
