"# GraphMed" 

GraphMed is a phased research pipeline for **graph-based clinical reasoning** over longitudinal patient records.  
It combines:

- structured clinical extraction from notes,
- patient-specific temporal knowledge graphs,
- vector memory retrieval across visits,
- an external medical knowledge base (RAG),
- and an interactive agent workflow for question answering and graph evolution.

The repository includes end-to-end phase scripts (`phase1_...` to `phase10_...`), evaluation artifacts, and a Streamlit demo app.

---

## Why GraphMed (Problem & Motivation)

Longitudinal clinical reasoning is difficult when information is spread across many visits, mixed with narrative text, and changes over time.  
GraphMed addresses this by:

1. converting visit narratives into structured entities and relations,
2. tracking temporal state and conflicts in patient graphs,
3. retrieving relevant historical memory semantically,
4. grounding responses with an external medical KB, and
5. evaluating system behavior against baseline approaches.

---

## Major Capabilities

- **Phase 1**: Synthetic patient timeline generation + normalization/validation.
- **Phase 2**: Rule-based + optional LLM-assisted clinical entity extraction.
- **Phase 3**: Patient knowledge graph construction and validation.
- **Phase 4**: Chroma-based vector memory store for visit retrieval.
- **Phase 5**: Medical KB ingestion (DrugBank, CDC guidelines, PubMed, MedlinePlus) with source-priority retrieval.
- **Phase 6**: ReAct-style GraphMed agent (LangGraph + tools) and interactive/demo runners.
- **Phase 7**: Conflict-classifier dataset generation and LoRA training.
- **Phase 8**: Graph evolution testing (add/update/conflict/trend behavior).
- **Phase 9**: Evaluation framework (factual QA, longitudinal consistency, conflict detection).
- **Phase 10**: Evaluation-driven graph updater from Phase 9 artifacts.
- **Demo App**: Streamlit interface for graph exploration, chat, evolution, and new-visit intake.

---

## High-Level Architecture / Workflow

### Core data and reasoning flow

1. **Patient data ingestion** (`data/patients`, then `data/patients_processed`)
2. **Extraction** (`src/extraction.py`)  
   Notes -> conditions, medications, symptoms, labs, relationships
3. **Knowledge graph build** (`src/graph.py`)  
   Temporal graph with confidence decay and conflict tracking
4. **Vector memory build** (`src/memory.py`)  
   Visit summaries embedded and stored in Chroma
5. **Medical KB build** (`src/medical_kb.py`)  
   External clinical evidence ingested and chunked for retrieval
6. **Agent runtime** (`src/agent_react.py`, `src/agent_direct.py`, `src/agent.py`)  
   Multi-tool reasoning over graph + memory + KB
7. **Evaluation** (`src/evaluation.py`)  
   Compares GraphMed vs baseline across three experiments
8. **Optional updater** (`src/knowledge_updater.py`)  
   Applies evaluation-driven graph updates and audit trail

### Agent + memory + graph + KB interactions

- Graph queries provide structured patient state/history
- Memory retrieval pulls semantically similar prior visits
- Medical KB retrieval provides external reference evidence
- Conflict/runtime modules support contradiction handling during graph evolution

---

## Repository Structure

```text
graphmed/
├── app.py                               # Streamlit demo app
├── phase1_*.py ... phase10_*.py         # Phase runners
├── config/
│   └── settings.py                      # Paths, model names, env-key loading
├── src/
│   ├── extraction.py                    # Phase 2 extraction
│   ├── graph.py                         # Patient graph + confidence logic
│   ├── memory.py                        # Chroma memory stores
│   ├── medical_kb.py                    # External KB ingestion/retrieval
│   ├── agent.py / agent_react.py        # Agent runtimes
│   ├── evaluation.py                    # Phase 9 experiments
│   └── knowledge_updater.py             # Phase 10 updater
├── data/
│   ├── patients/                        # Raw/synthetic patient JSON
│   ├── patients_processed/              # Extracted/processed patient JSON
│   ├── graphs/                          # Persisted patient graphs
│   ├── conflict_data/                   # Phase 7 train/val/test data
│   ├── medical_sources/                 # Input sources for Phase 5
│   └── medical_kb/                      # Persisted medical KB
├── evaluation/
│   └── results/phase9/                  # Phase 9 output artifacts
├── models/                              # Conflict classifier outputs
├── requirements.txt
└── requirements_langgraph.txt
```

---

## Prerequisites

- Python **3.10+** (the setup verifier checks for this)
- `pip`
- (Optional but commonly used) virtual environment
- API keys in `.env` for model-backed workflows

### `.env` keys

Create `.env` at repository root.

#### Required for common GraphMed workflows

- `GROQ_API_KEY`  
  Used by synthetic generation, extraction LLM mode, and default agent/evaluation provider paths.

#### Optional (used in specific providers/features)

- `OPEN_ROUTER_API_KEY` or `OPENROUTER_API_KEY` (OpenRouter provider paths)
- `GOOGLE_API_KEY` (Google/Gemini provider path in direct agent)
- `PUBMED_EMAIL` (enables PubMed ingestion in Phase 5)
- `HUGGINGFACE_API_KEY` (used by `test_huggingface.py`)
- `WANDB_API_KEY` (loaded in config; optional unless you add wandb-tracked workflows)

---

## Installation & Environment Setup

```bash
python -m venv venv
source venv/bin/activate          # Linux/macOS
# .\venv\Scripts\Activate.ps1     # Windows PowerShell

pip install -r requirements.txt
```

Optional (LangGraph-focused dependency stack):

```bash
pip install -r requirements_langgraph.txt
```

Optional model setup used by extraction/setup checks:

```bash
python -m spacy download en_core_web_sm
pip install https://s3-us-west-2.amazonaws.com/ai2-s3-scispacy/releases/v0.5.4/en_core_sci_md-0.5.4.tar.gz
```

You can run a setup sanity check with:

```bash
python verify_setup.py
```

---

## Usage

### Typical phase execution order

```bash
python phase1_generate_data.py
python phase1_normalize_patients.py --input-dir data/patients --report-path data/reports/patient_normalization_report.json
python phase1_validate.py

python phase2_process_patients.py
python phase3_build_graphs.py
python phase4_build_memory.py
python phase5_build_medical_kb.py
python phase6_run_agent.py

python phase7_generate_training_data.py
python phase7_train_conflict_classifier.py
python phase8_test_evolution.py
python phase9_run_evaluation.py
python phase10_run_knowledge_updater.py
```

Most phase scripts are interactive and present menu options in the terminal.

### Demo app

```bash
streamlit run app.py
```

---

## Data Handling Notes

- `data/patients/` contains synthetic or prepared patient timelines (JSON).
- `data/patients_processed/` stores extraction-enriched patient records.
- `data/graphs/` stores serialized patient knowledge graphs.
- `data/medical_sources/` is the input staging area for Phase 5 knowledge sources.
- `data/medical_kb/` stores the persisted vectorized external medical KB.
- `data/mimic_raw/` exists as a repository directory target for raw MIMIC-style data placement.

### Privacy and safety caveats

- The repository includes tooling for synthetic data generation and local JSON processing.
- If you introduce real clinical data, de-identification/privacy controls are your responsibility.
- `.env` and `data/` are gitignored in this repository, but you should still treat all patient-like data and API keys as sensitive.

---

## Phase 9 Evaluation Results (from `evaluation/results/phase9/phase9_results_table.csv`)

| Experiment | Mode | Metric | GraphMed | Baseline |
|---|---|---|---:|---:|
| E1 factual QA (50) | full_stack | FactScore | 0.5533333333333333 | 0.6533333333333333 |
| E1 multi-visit subset | full_stack | FactScore | 0.6333333333333333 | 0.5 |
| E2 longitudinal (50) | full_stack | BERTScore | -0.13392754166387022 | 0.057962282886728646 |
| E2 longitudinal (50) | full_stack | ConsistencyScore | -0.012663564878994863 | 0.18430216944928146 |
| E3 conflicts (30) | full_stack | Precision | 1.0 | 0.0 |
| E3 conflicts (30) | full_stack | Recall | 1.0 | 0.0 |
| E3 conflicts (30) | full_stack | F1 | 1.0 | 0.0 |
| E3 conflicts (30) | full_stack | F1_vs_pure_llm_prompt | 1.0 | 0.0 |

For experiment logic and artifact generation, see `src/evaluation.py` and `phase9_run_evaluation.py`.

---

## Configuration

Primary central settings are in `config/settings.py`, including:

- base data/model/evaluation directories,
- API key environment variable loading,
- model name defaults,
- graph/agent/evaluation constants.

Do **not** hardcode secrets in source code; keep keys in `.env`.

---

## Testing and Verification

Available scripts in this repository include:

- `python verify_setup.py` (environment/package/model checks)
- `python test_minimal.py`
- `python test_groq.py`
- `python test_api.py`
- `python test_huggingface.py`
- phase-specific validation/evaluation scripts (for example `phase3_validate_graphs.py`, `phase8_test_evolution.py`, `phase9_run_evaluation.py`)

---

## Contributing

Contributions are welcome through issues and pull requests.  
When contributing, keep phase outputs reproducible and avoid committing secrets or sensitive datasets.

---

## License

No license file is currently present in this repository.  
If you plan to reuse or redistribute this code, confirm licensing terms with the repository owner first.

---

## Medical Disclaimer

GraphMed is a **research/education project**.  
It is **not** a medical device and must **not** be used for clinical diagnosis, treatment decisions, or emergency care."
