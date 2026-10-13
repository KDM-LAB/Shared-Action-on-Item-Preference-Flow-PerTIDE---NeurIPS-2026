PerTIDE
Action-On-Item Preference Flow: A Shared Event Schema for Predictive and Generative Personalization  
NeurIPS 2026 · arXiv:2610.01375
PerTIDE (Personalization via multi-Timescale Interaction Drift Encoder) is a history encoder for personalization over heterogeneous user interactions. Instead of building a separate history representation for each domain or task, PerTIDE rewrites native interactions into a shared action-on-item event stream and learns how that evidence should enter, persist in, and be read from a user-specific state.
The model separates three parts of the update process:
Action-conditioned evidence entry: positive, negative, and command-like roles modulate the content written to state.
Multi-timescale persistence: long-term, short-term, and episodic traces retain evidence with different temporal biases.
Command-conditioned consumption: the retained user state is adapted for a predictive or generative consumer.
In compact form:
```text
native interaction
      ↓
action role + target-locus embedding
      ↓
action-conditioned evidence entry
      ↓
long-term / short-term / episodic traces
      ↓
adaptive fusion
      ↓
command-conditioned task-ready state
      ↓
prediction or generation
```
This repository contains the notebook-based research implementation used for the PerTIDE experiments, together with data-preparation utilities, embedding generation, evaluation code, and LLM baselines.
Repository structure
File	Purpose
`pertide_full.ipynb`	Main joint recommendation + personalized generation pipeline. Includes schema normalization, leakage-safe splits, PerTIDE encoder, recommendation head, UGI-conditioned DistilGPT2 decoder, three-stage training, checkpointing, stored-prediction evaluation, and PerSEval computation.
`pertide_movierec.ipynb`	Recommendation-only PerTIDE implementation for MovieLens-style trajectories. Includes action-conditioned updates, long/short/episodic traces, sampled training, checkpointing, and ranking evaluation.
`pertide-final.ipynb`	Shared multi-task research implementation for mixed predictive/generative tasks, including next-user prediction, next-item prediction, personalized summarization, and dialogue generation.
`embeddings.ipynb`	Builds BGE text embeddings for dialogue, document, response, and summary nodes.
`Data_prep.ipynb`	MPChat preprocessing and construction of dialogue interaction instances with derived action/sentiment roles.
`Recursive_Summarisation.ipynb`	Incremental persona/memory construction over MPChat-derived interactions.
`Recursive_recreated.ipynb`	Revised recursive memory/persona pipeline with stabilized fact, interest, style, and trajectory updates.
`recursive_persona.md`	Design notes for the incremental persona-memory pipeline.
`deepseek_personalization_baselines.ipynb`	DeepSeek personalization baseline for generation.
`llm_baselines (1).ipynb`	Additional LLM baselines using DeepSeek, Qwen, and Phi-family models.
`mpchat_hybrid_sentiment_updated.json`	MPChat-derived intermediate interaction data used by the auxiliary dialogue pipeline.
`mpchat_recursive_memory_stub.jsonl`	Intermediate recursive-memory output for the MPChat pipeline.
`uig_visual_sample.csv`	Small UIG-style interaction sample for inspection/visualization.
`requirements.txt`	Snapshot of the development environment.
Setup
The release is notebook-first. Create an isolated Python environment, install PyTorch for your platform, then install the remaining dependencies.
```bash
python -m venv .venv
```
Activate it on Linux/macOS:
```bash
source .venv/bin/activate
```
or on Windows PowerShell:
```powershell
.\.venv\Scripts\Activate.ps1
```
Install a PyTorch build appropriate for your CPU/CUDA setup from pytorch.org, then install the core packages used by the notebooks:
```bash
python -m pip install --upgrade pip
python -m pip install jupyterlab numpy pandas scipy scikit-learn matplotlib seaborn tqdm nltk
python -m pip install transformers sentence-transformers accelerate sentencepiece safetensors
```
`requirements.txt` records the original development environment, including a CUDA 12.1 PyTorch build. It can be useful for exact dependency inspection, but platform-specific packages should be adjusted when reproducing on a different operating system or CUDA version.
Launch the notebooks with:
```bash
jupyter lab
```
Data
Raw benchmark datasets are not redistributed in this repository. Download them from their original sources and follow the corresponding terms of use:
MovieLens 1M
MIND
PENS
OpenAI Reddit TL;DR / summarize-from-feedback
MPChat
The training notebooks consume preprocessed trajectories and embeddings, not the raw benchmark files directly. Before running a notebook, edit the path/configuration block near its beginning to point to your local preprocessing outputs.
MovieLens input
`pertide_movierec.ipynb` expects:
```text
interactions_ordered.csv
emb_movies.pkl
```
The interaction table is expected to contain an ordered user history and target movie, including the fields used by the notebook such as `userid`, `history`, and `target_mid`. `emb_movies.pkl` is a dictionary of movie embeddings keyed by the normalized movie identifiers used in the trajectory table.
The notebook defaults to a maximum history length of 50, uses seed 42, saves resumable checkpoints under `pertide_movierec_checkpoints/`, and evaluates the best checkpoint with multiple sampled-negative settings.
PENS recommendation + personalized generation
`pertide_full.ipynb` is the main joint recommendation/generation notebook. Configure the following variables in Cell 1 before execution:
```python
INTERACTIONS_CSV
NEWS_CSV
SUMMARIES_CSV
NEWS_BODY_EMB_PKL
SUM_T5_EMB_PKL
EXT_MAIN_TEST_CSV
EXT_TESTSUM_CSV
EXT_TESTSUM_PKL
ROOT_OUT
```
The notebook performs the following sequence:
load and normalize the interaction, news, summary, and embedding files;
build schema-normalized histories and leakage-safe train/validation/test splits;
train the recommendation stage;
continue with joint recommendation + generation training;
fine-tune the generation side;
restore the best checkpoints and generate stored predictions;
compute recommendation metrics, lexical/semantic generation metrics, and PerSEval outputs.
The default checkpoint/output directories are created under `ROOT_OUT`:
```text
ckpt_pertide_rec_sum_fixed/
eval_outputs_pertide_rec_sum_fixed/
```
Important generated files include stage-wise checkpoints and histories, stored recommendation/generation predictions, metric summaries, and PerSEval result files.
MPChat and mixed-task pipeline
The MPChat notebooks are auxiliary experiments for broader applicability under dialogue eventization.
Start by downloading MPChat and placing its speaker-identification data at the location configured in `Data_prep.ipynb` (the notebook currently expects `./mpchat/mpchat_si.json`). Run `Data_prep.ipynb` to build dialogue interaction instances and the derived positive/negative/neutral role labels.
For the recursive persona-memory experiments, use either `Recursive_Summarisation.ipynb` or the revised `Recursive_recreated.ipynb`. The design decisions behind this pipeline are documented in `recursive_persona.md`.
For the mixed multi-task encoder in `pertide-final.ipynb`, prepare the CSV and embedding files referenced in its configuration cells. The notebook expects tables for dialogue, documents, responses, summaries, and a mixed task table, together with the corresponding embedding dictionaries.
`embeddings.ipynb` can generate those text embeddings with `BAAI/bge-base-en-v1.5`. Edit `DATA_DIR` and `OUT_DIR` before running it; the checked-in notebook retains a local development path in its final configuration block.
LLM baselines
`deepseek_personalization_baselines.ipynb` and `llm_baselines (1).ipynb` contain the prompted LLM baselines used in the research workflow. They load models through Hugging Face Transformers with `device_map="auto"` and half-precision weights where configured. The 32B-model experiments are intended for appropriately provisioned GPU environments.
The baseline notebooks also use preprocessed CSVs. Update their dataset paths before execution.

Reproducibility notes

The main training notebooks set the random seed to 42 unless explicitly overridden.
CUDA is used automatically when available; several notebooks include CPU fallbacks, although large-model baselines are not practical CPU workloads.
Checkpoint/resume logic is included in the main PerTIDE notebooks.
No trained PerTIDE checkpoint is bundled in this repository; checkpoints are produced by the training notebooks.

Paper

The accompanying paper is:

> Parthiv Chatterjee, Kashish Kanjaria, Vashisth Purani, Sourish Dasgupta, and Tanmoy Chakraborty. **Action-On-Item Preference Flow: A Shared Event Schema for Predictive and Generative Personalization.** NeurIPS 2026.
Paper: arXiv:2610.01375
If you use this code or build on PerTIDE, please cite:
```bibtex
@inproceedings{chatterjee2026pertide,
  title     = {Action-On-Item Preference Flow: A Shared Event Schema for Predictive and Generative Personalization},
  author    = {Chatterjee, Parthiv and Kanjaria, Kashish and Purani, Vashisth and Dasgupta, Sourish and Chakraborty, Tanmoy},
  booktitle = {Advances in Neural Information Processing Systems},
  year      = {2026}
}
```
Third-party assets
Dataset and pretrained-model licenses are not superseded by this repository. MovieLens, MIND, PENS, OpenAI Reddit TL;DR, MPChat, Hugging Face models, and other third-party assets remain subject to the licenses and usage terms published by their respective maintainers. In particular, MPChat usage is subject to the terms documented by the official MPChat release.
Questions and issues
For implementation questions, reproducibility issues, or corrections to the release, please use the repository issue tracker and include the notebook name, the configuration used, and the relevant error/output trace.
