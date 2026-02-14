# Repository Analysis: Optimus-Solver

## What this repository is
OptiMUS is an LLM-driven optimization modeling pipeline that takes a natural-language optimization problem and produces:
1. extracted objective/constraints,
2. mathematical formulations,
3. executable Gurobi Python code,
4. debug iterations if generated code fails.

## Top-level architecture
- `main.py`: end-to-end orchestrator of the full workflow.
- Prompt-and-parser modules:
  - `parameters.py`: parameter extraction.
  - `objective.py`: objective extraction.
  - `constraint.py`: constraint extraction.
  - `constraint_model.py`: convert constraints to LaTeX + auxiliary vars.
  - `objective_model.py`: convert objective to LaTeX.
  - `target_code.py`: convert modeled constraints/objective to gurobipy snippets.
- Code generation and execution:
  - `generate_code.py`: writes full `code.py` from state.
  - `execute_code.py`: runs code and asks LLM to debug on failure.
- Utilities and runtime state:
  - `utils.py`: API clients, parsing helpers, JSON state/load/save, logger.
  - `variables.py`: variable extraction utilities (mostly subsumed by constraint modeling in current flow).
- RAG support:
  - `rag/rag_utils.py`: RAG paths + enum modes.
  - `rag/query_vector_db.py`: Chroma/OpenAI embedding retrieval functions.
- Data assets:
  - `data/rag/*`: persisted vector DBs + constraints dataset used for retrieval.

## Execution pipeline (`main.py`)
The run is sequential and state-driven (JSON checkpoints):
1. initialize run directory and state from problem folder (`desc.txt`, `params.json`, `labels.json`),
2. extract objective,
3. extract constraints,
4. formulate constraints (including NEW VARIABLES + AUXILIARY CONSTRAINTS),
5. formulate objective,
6. generate constraint/objective code snippets,
7. assemble full solver script,
8. execute + iterative debug.

## Expected input problem folder
The `--dir` target is expected to include:
- `desc.txt`: natural-language problem statement,
- `params.json`: parameter values + metadata,
- `labels.json`: categories used when RAG mode is `problem_labels`.

## Notable design choices
- Parsing strategy relies heavily on strict output delimiters (`=====`, JSON/list extraction from string tails).
- Most robustness comes from retry loops + parser fallbacks.
- RAG is optional and configurable via `--rag-mode` with three modes:
  - `problem_description`,
  - `constraint_or_objective`,
  - `problem_labels`.
- Default model in `main.py` is `gpt-4o`; utility code also supports Groq llama model naming.

## Caveats / observations
- API keys in `utils.py` and `rag/query_vector_db.py` are placeholders (`"###"`), so runtime requires environment/project-specific key setup.
- Some modules contain interactive `input(...)` flows for low-confidence checks, which may block non-interactive runs if enabled.
- `variables.py` exists but the current pipeline in `main.py` derives variables through constraint formulation (`NEW VARIABLES`).

## File-by-file quick guide
- `README.md`: project/paper overview and links.
- `main.py`: orchestration entrypoint.
- `parameters.py`: parameter extraction prompts + confidence filtering.
- `objective.py`: objective extraction prompt/parser.
- `constraint.py`: constraint extraction + optional consistency checks.
- `constraint_model.py`: convert natural-language constraints to LaTeX and collect introduced variables.
- `objective_model.py`: objective LaTeX formulation.
- `target_code.py`: produce gurobipy snippets for constraints/objective.
- `generate_code.py`: writes full optimization script from state.
- `execute_code.py`: execute generated script and debug with LLM on errors.
- `utils.py`: shared IO, parsing helpers, API clients, logging/state helpers.
- `rag/rag_utils.py`: RAG filesystem paths and mode enum.
- `rag/query_vector_db.py`: vector retrieval over stored optimization examples.
- `data/rag/...`: persisted retrieval indexes and metadata.
