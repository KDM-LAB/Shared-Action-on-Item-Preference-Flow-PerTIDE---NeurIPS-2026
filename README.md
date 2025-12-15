# MPChat Personalization via User Interaction Graph (UIG)  
### Hybrid Sentiment Analysis with SBERT-Expanded Lexical Cues

---

## 📌 Project Overview

This project builds a **personalized interaction modeling pipeline** over the **MPChat (ACL 2023)** dataset by constructing a **User Interaction Graph (UIG)** centered on a target speaker (referred to as *Alice*).

The core goal is to infer **how Alice reacts to other users over time**, using a **hybrid sentiment inference mechanism** that combines:

- Rule-based lexical signals  
- Pretrained sentiment polarity  
- Semantic similarity via SBERT embeddings  

The resulting sentiment-labeled interactions form the basis for downstream personalization and graph-based modeling.

---

## 📂 Dataset

**Dataset:** MPChat (Multimodal Persona Chat, ACL 2023)  
**File Used:** `mpchat_nrp.json`

Each split (`train`, `val`, `test`) contains multi-party dialogues with:
- `messages`
- `authors`
- `created_utcs`
- `main_author` (target speaker)
- `candidate_personas`

This project **excludes images** and focuses entirely on text-based interactions.

---

## 🧠 Core Design: User Interaction Graph (UIG)

For each dialogue:

- **S-node** → Alice’s utterance (response)
- **D-nodes** → Other users’ utterances that Alice responds to
- **Edges** → Labeled with Alice’s inferred sentiment toward the D-node

### Recency Bias
Only the **most recent D-node** before an Alice response is linked, reflecting realistic conversational dynamics.

---

## 🔁 Dialogue Processing Pipeline

### 1. Dialogue Extraction
For each dialogue:
- Identify all turns where Alice speaks
- For consecutive Alice turns:
  - Collect all intervening utterances from other users as context (D-nodes)
  - The second Alice turn becomes the S-node

### 2. UIG Instance Format

Each processed instance is stored as:

```json
{
  "dialog_id": "uuid",
  "target_speaker": "alice",
  "context": [
    {
      "turn_index": 5,
      "speaker_id": "u3",
      "utterance": "text",
      "sentiment_towards_target": "positive | negative"
    }
  ],
  "s_node": {
    "turn_index": 6,
    "speaker_id": "alice",
    "utterance": "text"
  }
}
