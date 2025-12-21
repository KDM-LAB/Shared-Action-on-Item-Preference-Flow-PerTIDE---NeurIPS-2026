# Incremental Persona Memory: Design & Rationale

This document explains the final approach we used to extract user personas,
why earlier attempts failed, and why this design scales to the full dataset.

---

## Problem We Were Solving

We wanted to extract **long-term user personas** from a Reddit-style dialog dataset.

Key constraints:
- Dialogs are **not sequential**
- Users appear **sporadically** across many threads
- Most users have **very few utterances**
- We cannot afford to generate empty personas for thousands of users
- We need personas that feel **natural and defensible**, not hallucinated

---

## Why Naive Approaches Failed

### 1. Per-Utterance Memory Updates (Too Strict)

Initially, we updated persona memory after every single utterance with rules like:

- “Only update after many interactions”
- “Never infer from one message”

**Result:**  
Persona memory never updated at all.

This created a deadlock:
- Memory starts empty
- Model is forbidden from creating the first signal
- Persona stays empty forever

---

### 2. Batch / Aggregate Per-User (Too Coarse)

We considered aggregating all utterances for a user and summarizing them at once.

**Why we rejected this:**
- Loses temporal grounding
- Encourages hallucinated coherence
- Breaks alignment with the original dialog structure
- Harder to debug and validate

---

## The Chosen Approach: Incremental + Stabilized Persona Updates

We kept the original incremental pipeline but fixed **where it was failing**.

### Core Idea

> Persona should grow **gradually**, but must be allowed to **start existing**.

So instead of “only update after many interactions”, we used:

> “Update only when a trait is **clearly expressed and high-level**, even if memory is empty.”

---

## How the Final Pipeline Works

Each dialog instance is processed independently.

At every step we do **two separate things**:

### 1. Persona Memory Update (Long-Term)

We maintain a rolling persona object with three fields:

- **Facts**  
  Stable self-identification or roles  
  (e.g. “creates art to fund more art”)

- **Style**  
  Tone and expressiveness  
  (e.g. “casual, friendly”)

- **Interests**  
  High-level recurring topics  
  (e.g. “art”, NOT brands or locations)

The model:
- Sees the **current persona memory**
- Sees **one new user utterance**
- Uses sentiment/alignment scores as *hints only*
- Updates memory **only if a trait is likely to persist**

Crucially:
- If nothing is strong enough → memory stays unchanged
- If a trait is clearly expressed → it can be added tentatively

---

### 2. Trajectory Summary (Short-Term)

Separately, we maintain a **trajectory summary** that answers:

> “What just happened in this interaction?”

This captures:
- Topic
- Intent
- Factual context

It **never** updates persona traits.

This separation prevents:
- Style leakage
- Interest hallucination
- Feedback loops

---

## Why This Works for Sparse Users

Most users only have 1–2 utterances.

Instead of forcing a persona, we allow:
- **No persona** when nothing persistent exists
- **Minimal persona** when a strong signal appears

This avoids:
- Fabricated interests
- Overfitting one-off comments
- Noisy personalization

---

## Why This Scales to 7,000+ Dialogs

- One pass over the dataset
- No cross-dialog dependency
- No per-user batching
- Memory grows only when justified

Persona memory becomes:
- **Sparse but meaningful**
- **Auditable**
- **Aligned with raw data**

---

## Key Design Principles We Followed

1. **Incremental, not batch**
2. **High-level, not specific**
3. **Allow first signal**
4. **Never overwrite with emptiness**
5. **Separate memory from trajectory**
6. **Use metadata as hints, not truth**

---

## Final Outcome

For users with real signal, we get personas like:

- Facts: *creates art to fund more art*
- Style: *casual, expressive*
- Interests: *art*

For users without signal:
- Persona remains empty (by design)

This is acceptable and **honest**.

---

## Takeaway

> The pipeline wasn’t broken — the prompt logic was over-constrained.

By allowing **carefully-gated first updates**, we unlocked stable persona learning
without sacrificing correctness or scale.
