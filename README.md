# Challenge 3: Multi-Agent vs. Single-Agent Topology in Forensic Legal Audit and Contractual Procurement

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/EstebanGZam/multi-agent/blob/main/reto3_topologia_multiagente_forense_legal.ipynb)
![Python Version](https://img.shields.io/badge/python-3.10%2B-blue)
![Conda Env](https://img.shields.io/badge/conda%20env-apo__iii-green)
![LangSmith](https://img.shields.io/badge/observability-LangSmith-orange)

Formal submission for **Challenge 3 (Workshop 03: Multi-Agent and Orchestration)** of the Artificial Intelligence course (2026-2) at Universidad Icesi.

This project presents an empirical, quantitative comparison between a **Single-Agent Baseline (Monolithic ReAct)** and various **Multi-Agent Topologies** (Multi-Agent Uniform, Multi-Agent Mixed with Model Tiering, and Intelligent Dynamic Router with Adaptive Triage) when solving a high-stakes, analytical task: **Legal Forensics and Contractual Procurement Auditing** under the Colombian public procurement regulatory framework (Law 80/1993, Law 1150/2007, Decree 1082/2015, and Council of State jurisprudence).

---

## Authors

- Esteban Gaviria Zambrano (A00396019)
- Julián Andrés Mendoza Castro (A00395237)

---

## 1. Problem Formulation and Use Case

In corporate governance and public procurement, determining the legal, financial, and technical validity of high-value proposals requires rigorous, concurrent evaluation across three distinct disciplines:
1. **Legal Domain**: Detecting statutory disqualifications, conflicts of interest, debarment records from the Comptroller General (Boletin de Responsables Fiscales) and Inspector General (Procuraduria), and mandatory rejection causes.
2. **Financial Domain**: Peritus auditing of balance sheets, certified calculation of qualifying financial ratios (Liquidity Ratio, Debt Ratio, Interest Coverage Ratio, Working Capital), and equity gap identification.
3. **Technical Domain**: Standard classification using UNSPSC codes, verified experience registered in the RUP (in minimum wage multiples - SMMLV), and mandatory accreditation of security standards (e.g., ISO/IEC 27001).

### Monolithic Single-Agent Failure Modes vs. Multi-Agent Solution
- **Monolithic Single-Agent Failure Modes**: A single agent equipped with all tools simultaneously suffers from context saturation (*lost in the middle*), conflates mandatory rejection grounds with weighted scoring criteria, makes arithmetic errors in balance sheet percentages, and lacks an adversarial peer-review mechanism.
- **Multi-Agent Solution**: Role specialization with **clean context windows**, concurrent parallel exploration via `asyncio.gather`, structured synthesis governed by Pydantic contracts, and adversarial quality control through an independent **Forensic Critic (Evaluator-Optimizer pattern)**.

---

## 2. Architectural Topologies

### Topology A: Single-Agent Baseline (Monolithic ReAct)
A single model receives the entire procurement file, holds all tools within its single prompt context, and executes a sequential reasoning-action loop.

```mermaid
flowchart LR
    User([Procurement File / Query]) --> Agent[Monolithic ReAct Agent]
    Agent <--> Tools[(Forensic Tools:\nSearch, Ratios, Disqualifications)]
    Agent --> Verdict([Final Forensic Verdict])
```

### Topology B: Multi-Agent Uniform (Orchestrator + Parallelism + Rapporteur + Critic)
- **Orchestrator**: Decomposes the procurement audit into independent subtasks (`LEGAL`, `FINANCIAL`, `TECHNICAL`).
- **Specialist Analysts**: Run concurrently using `asyncio.gather` with specialized prompts and domain-specific tools.
- **Legal Rapporteur**: Synthesizes cross-domain findings into a reasoned legal opinion.
- **Forensic Critic (Evaluator-Optimizer)**: Evaluates the draft verdict against a strict rubric (score 1 to 5). If unsupported assertions or contradictions are detected, it issues structured feedback for refinement.
- **Model Assigned**: Uniform deployment of `gemini-3.5-flash-lite` (Google AI Studio Free-Tier) across all agents ($0.0375 / $0.150 USD per million tokens).

```mermaid
flowchart TD
    User([Procurement File / Query]) --> Orq[Forensic Orchestrator]
    Orq -->|Legal Subtask| Jur[Legal Specialist]
    Orq -->|Financial Subtask| Fin[Financial Specialist]
    Orq -->|Technical Subtask| Tec[Technical Specialist]
    Jur <--> ToolsJur[(Legal DBs & Records)]
    Fin <--> ToolsFin[(Financial Calculator)]
    Tec <--> ToolsTec[(UNSPSC & RUP Tools)]
    Jur --> Rel[Legal Rapporteur / Synthesizer]
    Fin --> Rel
    Tec --> Rel
    Rel --> Draft[Draft Opinion]
    Draft --> Crit[Forensic Critic / Evaluator-Optimizer]
    Crit -->|Rejected: Score < 4 / Hallucination| Rel
    Crit -->|Approved: Score >= 4| FinalVerdict([Approved Forensic Verdict])
```

### Topology C: Multi-Agent Mixed (Model Tiering)
Identical architecture to Topology B, but implementing a tiered model distribution strategy:
- Specialist Analysts and Rapporteur: Economical model `gemini-3.5-flash-lite` ($0.0375 / $0.150 USD per million tokens).
- Orchestrator and Critic: Higher-reasoning model `gemini-3.5-flash` ($0.075 / $0.300 USD per million tokens).

### Topology D: Dynamic Router (Adaptive Triage)
A lightweight classifier examines the input query:
- If monodisciplinary (e.g., verifying a specific financial formula or checking an individual debarment record), it routes directly to the designated specialist, eliminating orchestration overhead.
- If multidisciplinary or complex, it delegates the query to the complete multi-agent committee.

```mermaid
flowchart TD
    User([Query / Procurement Case]) --> Router{Triage Router}
    Router -->|Narrow Legal| Jur[Legal Specialist]
    Router -->|Narrow Financial| Fin[Financial Specialist]
    Router -->|Narrow Technical| Tec[Technical Specialist]
    Router -->|Multidisciplinary / Complex| MultiTeam[Full Multi-Agent Committee]
    Jur --> Resp([Fast Direct Response])
    Fin --> Resp
    Tec --> Resp
    MultiTeam --> Resp
```

---

## 3. Quantitative Results Matrix

Empirical evaluation executed across the calibrated 10-case forensic benchmark (`BENCHMARK_DATASET`) utilizing live Google Gemini Free-Tier endpoints (`gemini-3.5-flash-lite` and `gemini-3.5-flash`):

| Configuration | Mean Quality (0-1) | Mean Cost (USD) | Total Cost (USD) | Mean Latency (s) | Latency p50 (s) | Latency p95 (s) | Success Rate (%) | Calls / Task | Coordination Cost (%) | Duplicate Queries (%) |
|---|---|---|---|---|---|---|---|---|---|---|
| **A · Single-Agent (Baseline)** | 0.730 | $0.000057 | $0.00057 | 1.868 | 1.779 | 2.596 | 100.0% | 1.9 | 0.0% | 30.3% |
| **B · Multi-Agent Uniform** | 0.954 | $0.000197 | $0.00197 | 5.648 | 5.860 | 7.794 | 100.0% | 7.0 | 35.8% | 36.4% |
| **C · Multi-Agent Mixed (Tiered)** | **0.965** | $0.000237 | $0.00237 | 11.485 | 11.012 | 17.837 | 100.0% | 6.6 | 37.6% | 36.3% |
| **D · Dynamic Router** | 0.794 | $0.000133 | $0.00133 | 5.410 | 5.005 | 9.171 | 100.0% | 5.6 | 34.9% | 24.0% |

---

## 4. Evaluation of the Five Core Dimensions (PPT_Multiagente.pdf Slide 4)

1. **Quality**:
   - The Single-Agent Baseline scored 0.730, struggling on multi-party consortia where fiscal disqualifications are embedded in minority partners.
   - Multi-Agent Uniform reached 0.954, validating the role-specialized context isolation.
   - Multi-Agent Mixed reached **0.965**, demonstrating that dedicating a stronger reasoning model (`gemini-3.5-flash`) to the adversarial Forensic Critic reliably identifies subtle non-compliance clauses.
2. **Cost**:
   - Calculated with precision using token tracking: `Cost = (tokens_in * price_in + tokens_out * price_out) / 1e6`.
   - The Single-Agent baseline averaged $0.057 mUSD per case.
   - Topology B increased mean cost to $0.197 mUSD per case, an accessible investment under Google AI Studio free tier for high-reliability forensic validation.
   - Topology C cost was $0.237 mUSD per case, only 20% higher than Topology B while capturing the peak quality score.
3. **Latency**:
   - Single-Agent latency remained the lowest (1.868 s mean, 1.779 s p50).
   - Multi-Agent Uniform achieved a fast 5.648 s mean (5.860 s p50), aided by `asyncio.gather` parallel specialist execution.
   - Multi-Agent Mixed recorded 11.485 s mean due to sequential reflection and critique rounds.
   - Dynamic Router achieved 5.410 s mean with 24.0% duplicate queries.
4. **Robustness**:
   - 100% operational success rate across all evaluated configurations.
   - Active protection against cost runaway via deterministic budget enforcement (`BudgetExceededException`).
   - In negative controls (e.g., nonexistent tender LP-999-2026), agents abstained cleanly with "Information unavailable", completely avoiding hallucinations.
5. **Coordination**:
   - In Topology B, 35.8% of total cost represents orchestration and critique overhead.
   - In Topology C, coordination overhead was 37.6%, reflecting balanced tiering without runaway expenditure.
   - Duplicate queries ranged between 24.0% (Router) and 36.4% (Uniform multi-agent).

---

## 5. Pareto Frontier Analysis: Cost vs. Quality

Mathematical dominance analysis across the 4 topologies reveals that all configurations lie on the Pareto optimal curve:

$$\text{Pareto Frontier} = \{ \text{A · Single-Agent}, \text{D · Dynamic Router}, \text{B · Multi-Agent Uniform}, \text{C · Multi-Agent Mixed} \}$$

- **Configuration A (Single-Agent)**: The Pareto choice when latency and token cost are absolute constraints, and a preliminary screening suffices.
- **Configuration D (Dynamic Router)**: The triage sweet spot for interactive systems processing mixed simple and complex workloads.
- **Configuration B (Multi-Agent Uniform)**: The high-throughput multi-agent choice balancing rapid turnaround (5.65 s) with strong quality (0.954).
- **Configuration C (Multi-Agent Mixed)**: The non-dominated peak accuracy configuration (0.965) for critical legal forensics where false negatives carry severe statutory liability.

---

## 6. Observability and Cost Tracing with LangSmith

Cost monitoring and execution tracing are instrumented through **LangSmith** and a dual local cost tracker:
- **Project**: `reto3-multiagent-evaluation`.
- **Run Traces**: Every tool invocation, agent step, critic iteration, and router branch is tagged with metadata (`topology`, `case_id`, `role`, `model`).
- **Dual Tracking**: Local pricing mirrors actual token rates on OpenRouter and Google Gemini APIs, allowing cost verification even when running offline or behind proxy boundaries.

---

## 7. Execution Instructions

### Option A: Google Colab (Recommended One-Click Setup)
1. Click the badge: [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/EstebanGZam/multi-agent/blob/main/reto3_topologia_multiagente_forense_legal.ipynb).
2. To run with live API endpoints, store your keys in **Colab Secrets** (key icon in the sidebar):
   - `OPENROUTER_API_KEY`: For open model routing (e.g., DeepSeek, Llama, Qwen).
   - `GEMINI_API_KEY`: For Google Gemini models.
   - `LANGSMITH_API_KEY`: For live LangSmith cloud tracing.
3. Run all cells sequentially (`Runtime -> Run all`). If no keys are provided or if credit limits are reached, the notebook executes autonomously and deterministically using the built-in `ForensicSimulationEngine`.

### Option B: Local Conda Environment (`apo_iii`)
```bash
# Activate the existing apo_iii Conda environment
conda activate apo_iii

# Set up environment variables
cp .env.example .env
# Edit .env with your respective API keys

# Launch JupyterLab or Jupyter Notebook
jupyter lab reto3_topologia_multiagente_forense_legal.ipynb
```

---

## 8. Repository Layout

```text
multi-agent/
├── README.md                                       # Complete project documentation and report
├── reto3_topologia_multiagente_forense_legal.ipynb    # Fully executed Jupyter notebook with English outputs and figures
├── PPT_Multiagente.pdf                             # Theoretical reference slides (Unit III)
├── arquitectura_multiagente.ipynb                  # AsyncOpenAI multi-agent classroom reference
├── arquitectura_multiagente_langgraph.ipynb        # LangGraph multi-agent classroom reference
├── pyproject.toml                                  # Python project definition and dependencies
├── uv.lock                                         # UV dependency lockfile
├── .env.example                                    # Environment variable template
└── .gitignore                                      # Git versioning exclusion rules
```

---

## 9. License and Academic Credits

Developed for the **Artificial Intelligence (2026-2)** course at **Universidad Icesi**, instructed by Professor Jose Armando Ordonez.
