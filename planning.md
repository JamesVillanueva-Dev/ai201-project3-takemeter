# TakeMeter Classification Project: Planning Document

## 1. Community Selection

### Community Chosen: r/gaming

**Why r/gaming?**
r/gaming is a large, active subreddit with over 4 million subscribers dedicated to discussing video games, gaming culture, and gaming news. This community is ideal for a classification task because posts naturally vary across multiple dimensions: some are factual game announcements, some are personal opinions about games, some ask for help or recommendations, and some share interesting gaming facts. The discourse is naturally diverse and non-trivial to classify because the same topic (e.g., a game's quality) can be discussed from factual, opinion-based, or help-seeking angles. This variety makes the classification task genuinely challenging and interesting.

---

## 2. Labels & Definitions

I will use **4 labels** for this classification task:

### Label 1: **Opinion**
A post that primarily expresses a personal judgment, preference, or subjective view about a game, gaming trend, or gaming-related topic. The main purpose is to share what the poster believes or feels.

**Example 1:** "Red Dead Redemption 2 is the most immersive game ever made. The attention to detail is unmatched, and I think it's one of the greatest games of all time."

**Example 2:** "Honestly, microtransactions in AAA games have ruined the industry. It's getting ridiculous and I hate how greedy these companies have become."

---

### Label 2: **Explanation**
A post that primarily provides factual information, technical details, clarifications, or educational content about games, gaming mechanics, updates, or industry news. The main purpose is to inform or explain.

**Example 1:** "Just to clarify: the newest Baldur's Gate 3 patch changed the AC calculation system. Proficiency bonus now applies directly to AC instead of through attack rolls. This makes armor significantly more valuable."

**Example 2:** "Call of Duty's DLSS implementation uses Nvidia's SuperResolution technology to render at a lower resolution and upscale. This improves performance by 20-30% depending on GPU."

---

### Label 3: **Seeking Assistance**
A post that asks a question or requests help, advice, or recommendations from the community. The main purpose is to get guidance or solve a problem.

**Example 1:** "I'm stuck on the Elden Ring boss fight with Malenia. Does anyone have any tips or strategies that worked for them? What weapons/spells do you recommend?"

**Example 2:** "Looking for a good co-op game to play with my friends. We like story-driven games but also want competitive elements. Any suggestions?"

---

### Label 4: **Cool Fact**
A post that shares an interesting, surprising, or fun piece of trivia, discovery, or novelty about games. The main purpose is to inform and entertain with unexpected or lesser-known information.

**Example 1:** "Did you know? The original Half-Life was developed in just 3 years with a team of 15 people. The engine was built from scratch and this level of quality was incredible for 1998."

**Example 2:** "I just discovered that in Skyrim, if you pickpocket someone's clothes, they walk around in their underwear. I've spent 2 hours just doing this and it's hilarious."

---

## 3. Hard Edge Cases

### Primary Edge Case: Opinion vs. Explanation with Opinion

**Challenge:** Posts that explain facts but also embed personal judgment make it ambiguous whether the post is primarily explanatory or opinion-based.

**Example ambiguous post:** "The new Starfield gameplay reveal shows 120fps on high-end PCs, but honestly, I think 60fps is good enough. Why pay for a gaming PC when my console runs games fine at 60fps?"

This post contains both factual explanation (120fps capability) and personal opinion (thinking 60fps is sufficient).

**Handling Strategy:** 
- Classify by **primary intent**: What is the post's main message? Is the poster primarily trying to inform readers of a fact, or primarily advocating for their subjective view?
- If the opinion is secondary to explaining facts → classify as **Explanation**
- If the facts are just support for a subjective argument → classify as **Opinion**
- When truly ambiguous (50/50), use the **last sentence** as the tiebreaker: what does the poster emphasize at the end?

### Secondary Edge Case: Seeking Assistance vs. Opinion

Some posts ask for help while also expressing frustration or dissatisfaction. "Anyone else think this game is impossibly hard? How do people beat this?" contains both a question and an implicit opinion about difficulty.

**Handling Strategy:** Focus on the **functional purpose**. If the primary goal is to get help/advice, classify as **Seeking Assistance** even if opinion is embedded.

---

## 4. Data Collection Plan

**Source:** r/gaming subreddit via Reddit API (PRAW library)

**Collection Strategy:**
- Scrape posts from r/gaming's "Hot" and "New" sections to capture recent, diverse posts
- Target 100 total posts (25 per label) to build initial balanced dataset
- Use random sampling to avoid selection bias toward popular posts

**Handling Underrepresentation:**
- If any label has <15 examples after 100 posts, I will:
  1. Use targeted keyword searches (e.g., "how do I" for Seeking Assistance, "did you know" for Cool Fact)
  2. Manually search for underrepresented label patterns within existing r/gaming threads
  3. Increase total dataset size to 150 if necessary to achieve balanced representation

**Timeline:** Aim to collect examples over 2-3 days to capture different posting times and user activity patterns.

---

## 5. Evaluation Metrics

**Primary Metrics:**

1. **Accuracy** — Overall percentage of correct classifications. This tells me if the model is working at all, but it's not enough alone.

2. **Per-Label Precision & Recall** — For each label:
   - **Precision:** Of posts I predicted as [Label], how many were actually [Label]? (False positives matter—don't want to mislabel gaming discussions)
   - **Recall:** Of posts that are actually [Label], how many did I find? (False negatives matter—don't want to miss legitimate posts)

3. **Macro F1-Score** — Balanced measure of precision and recall across all labels. This is important because some labels may be rarer, and I want the model to perform well on all of them, not just dominant classes.

4. **Confusion Matrix** — Shows which labels are confused with each other. I expect Opinion↔Explanation to have high confusion (our identified edge case), and this will reveal if that's true.

**Why These Metrics?**
- Accuracy alone hides poor performance on minority labels
- Precision/Recall show whether the model is useful in practice (e.g., if Seeking Assistance has low recall, I miss help requests)
- Macro F1 ensures balanced performance across all 4 labels
- Confusion matrix validates our edge case assumptions and guides error analysis

---

## 6. Definition of Success

**Minimum Acceptable Performance:** Macro F1-score of **0.70 or higher** across all labels.

**Why 0.70?**
- At 0.70 F1, the model correctly identifies ~70% of all post types while maintaining reasonable precision (minimal false positives)
- This is realistic for a 4-label task with overlapping concepts (Opinion vs. Explanation edge cases)
- Below 0.70, too many misclassifications to be useful for a real community tool

**Real-World Usefulness:**
If this classifier were deployed in a gaming community tool (e.g., to automatically filter/organize r/gaming posts), users would expect:
- Seeking Assistance posts reliably flagged (low false negatives on help requests)
- Cool Facts not mislabeled as Opinions (avoid making educational content seem subjective)
- Acceptable trade-off: Some Opinion↔Explanation confusion is acceptable because both indicate substantive discussion

**Target Performance:** Ideally **0.75+ macro F1** — this would mean the model is production-ready for real-world use in community moderation or post organization tools.
