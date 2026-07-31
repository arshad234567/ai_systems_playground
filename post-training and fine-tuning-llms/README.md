# Understanding LLM Training: Pre-training, Mid-training, and Post-training

## Overview

Large Language Model training happens in **three broad stages**: pre-training → mid-training → post-training. This document explains what each stage does, why it exists, and how post-training (fine-tuning + reinforcement learning) builds on top of the earlier stages.

---

## 1. Pre-training — "Reading the entire library"

- The model starts with **completely randomized weights** and knows nothing.
- Task: **next-token prediction** — given some text, predict the next word/token.
- Example: given "Once upon", predict "a", then "midnight", etc. (echoing *The Raven*).
- Trained on a **huge, largely uncurated slice of internet text** — mixed quality (low and high quality books/pages alike).
- **Extremely expensive**: can take *months of compute* on massive infrastructure.
- What emerges is surprisingly powerful: by repeatedly predicting the next token across huge datasets, the model implicitly learns **concepts and world knowledge**.
  - Example: "The sky is ___" → high probability on "blue" (learned association).
  - Context-sensitive: "The sun is setting, the sky is ___" → "orange" becomes more likely — showing the model has learned *contextual* concepts, not just fixed associations.
- **Limitation**: the resulting *base model* only knows how to continue text statistically. It has raw knowledge but no notion of being helpful, following instructions, or behaving appropriately.

---

## 2. Mid-training — "Reading a curated set of advanced books"

- Essentially **continued pre-training**, but on a smaller, more **curated** dataset.
- Often handled by a different team than pre-training within frontier labs.
- Still uses the same next-token prediction objective.
- Common goals:
  - Teaching **new languages** (e.g., adding strong Chinese capability).
  - Adding **new modalities** (audio, images).
  - **Extending context length** (teaching the model to handle much longer sequences than it saw in pre-training).
- Think of it as a bridge stage: the model already has raw intelligence from pre-training; mid-training sharpens/extends specific capabilities before the final shaping stage.

---

## 3. Post-training — "Becoming an effective tutor/assistant"

Post-training is where a raw, knowledgeable-but-unruly base model is turned into something **useful, safe, and interactive** — able to answer questions clearly and interact politely enough to actually function as a helpful assistant.

Two major techniques:

### a) Fine-tuning (a.k.a. Supervised Fine-Tuning / SFT)
- You provide **explicit input → target output pairs**.
- The model learns from **gradients** based on how close its output is to the target, and improves over time.
- Efficiency techniques: **LoRA (Low-Rank Adaptation) adapters** — small additional weight matrices that let you fine-tune effectively without updating the full model, making it feasible to fine-tune on a single GPU or even locally.

### b) Reinforcement Learning (RL)
- Instead of exact target outputs, the model gets a **reward/score** signal indicating whether a response was good or bad.
- Requires a way to *generate* that reward — often another model (a **reward model**).
- **RLHF (Reinforcement Learning with Human Feedback)**: the reward model itself is trained on human preference data, and that reward model is then used to train the main LLM.
- This pipeline can involve **up to four different models** working together, making RL **computationally expensive** relative to fine-tuning.

---

## Summary Comparison

| Stage | Data | Objective | Curation Level | Cost | Outcome |
|---|---|---|---|---|---|
| **Pre-training** | Massive raw internet-scale text | Next-token prediction | Low (broad, mixed quality) | Very high (months of compute) | Raw knowledge / base model |
| **Mid-training** | Curated, targeted datasets | Next-token prediction | Medium-high | Moderate | New languages, modalities, longer context |
| **Post-training** | Labeled examples / preference & reward signals | Match target output (SFT) or maximize reward (RL) | High (curated, human-informed) | Varies (SFT cheaper, RL expensive) | Helpful, aligned, usable assistant |

**Helpful analogy:**
- Pre-training = reading an entire library indiscriminately.
- Mid-training = reading a curated set of advanced books on specific topics.
- Post-training = learning to be an effective tutor — how to answer clearly and interact politely.

---

## Key Takeaways

1. Post-training is the **last** stage of LLM training, following pre-training and mid-training.
2. Pre-training builds raw knowledge; mid-training sharpens/extends specific capabilities; post-training shapes **behavior and usability**.
3. **Fine-tuning (SFT)** teaches the model from explicit target outputs.
4. **Reinforcement learning** teaches the model from reward/score signals, often via a separately trained reward model (RLHF).
5. LoRA adapters make fine-tuning accessible on limited hardware (even a single GPU or local machine).
6. RL pipelines are more complex and computationally expensive than SFT, often involving multiple models.
