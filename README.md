# LLM for Red/Blue Teaming: Cybersecurity Evaluation via CTFKnow

[![Python](https://img.shields.io/badge/Python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Paper](https://img.shields.io/badge/arXiv-2506.17644-red.svg)](https://arxiv.org/abs/2506.17644)

## 🗂️ Project Info

| Field | Details |
|---|---|
| **Student** | TAM Kar Nam (SID: 21191632) |
| **Project Title** | LLM for Red/Blue Teaming |
| **Supervisor** | Prof. Wang Shuai |
| **Industry Collaborator** | CMHK |
| **Course** | CSIT 6910 – Independent Project |

---

## 📖 Overview

This project comprehensively evaluates the technical cybersecurity knowledge of mainstream Large Language Models (LLMs) using the **CTFKnow** benchmark — a dataset derived from Capture-the-Flag (CTF) challenge write-ups. By measuring LLM performance on both single-choice and open-ended questions, the project investigates *why* certain model architectures outperform others, specifically comparing reasoning vs. non-reasoning models and newer vs. older model generations.

The project reproduces and extends the evaluation methodology from:
> Ji, Z., Wu, D., Jiang, W., Ma, P., Li, Z., & Wang, S. (2025). *Measuring and Augmenting Large Language Models for Solving Capture-the-Flag Challenges.* [arXiv:2506.17644](https://arxiv.org/abs/2506.17644)

### 🎯 Key Objectives

- **Reproduce** the CTFKnow measurement results from Ji et al., 2025 (Section 3: Measuring LLM Capability for CTF)
- **Evaluate** GPT-5-mini, GPT-4o, and o4-mini on CTF technical knowledge tasks
- **Analyze** why reasoning models outperform non-reasoning models
- **Investigate** why newer models (GPT-5-mini) outperform older models (GPT-4o)
- **Visualize** performance by difficulty, topic, and question type

---

## 🏗️ Architecture & Pipeline

```
Cyber-Security-Evaluation/
├── dataset/                 # CTFKnow dataset (205 samples used)
│   └── list.json            # Challenge metadata
├── run.py                   # Main evaluation pipeline (Azure OpenAI)
├── prompts.py               # LLM prompt templates
├── visualize.py             # Data visualization (pandas/matplotlib)
├── results/
│   ├── gpt-4o_results.json
│   ├── gpt-5-mini_results.json
│   ├── o4-mini_results.json
│   └── combined_results.json  # Merged with difficulty + type fields
└── README.md
```

---

## 🚀 Quick Start

### Prerequisites

- Python 3.8+
- Azure OpenAI API access (configured to run without VPN)
- Required Python packages

### Installation

1. **Clone the repository**
   ```bash
   git clone https://github.com/goodtam8/Cyber-Security-Evaluation-.git
   cd Cyber-Security-Evaluation-
   ```

2. **Install dependencies**
   ```bash
   pip install -r requirements.txt
   ```

3. **Set up Azure OpenAI credentials**
   ```bash
   export AZURE_OPENAI_API_KEY="your-azure-api-key"
   export AZURE_OPENAI_ENDPOINT="your-azure-endpoint"
   ```

### Basic Usage

#### 1. Knowledge Extraction (Pre-collected data available from [Ji et al., 2025](https://github.com/tszdanger/CTFKnow/tree/main/dataset))
```bash
python run.py K -i dataset/list.json -o dataset/knowledge.json
```

#### 2. Question Generation
```bash
python run.py Q -i dataset/knowledge.json -o dataset/questions.json -q dataset/question_list.json
```

#### 3. Model Evaluation
```bash
# Multiple choice evaluation
python run.py E -M single -l gpt-4o -o results/gpt-4o_results.json

# Open-ended question evaluation
python run.py E -M open -l gpt-5-mini -o results/gpt-5-mini_results.json
```

#### 4. Visualization
```bash
python visualize.py --input results/combined_results.json
```

---

## 📊 Evaluation Results

Models were evaluated on the first **205 samples** of the CTFKnow dataset due to token budget constraints (≈HKD 300). An additional validator model (o3-mini) was used to cross-check short-answer responses to avoid bias.

### Decoding Hyperparameters

| Parameter | Value |
|---|---|
| Temperature | 0 |
| top_p | 0.9 |
| top_k | 50 |
| presence_penalty | 1.15 |
| frequency_penalty | 0.2 |
| max_tokens | 2048 |

### Single-Choice Performance

All three models achieved **>90% accuracy** on single-choice questions:

| Model | Single-Choice Accuracy |
|---|---|
| **GPT-5-mini** | **96.5%** (highest) |
| o4-mini | >90% |
| GPT-4o | >90% |

### Open-Ended Performance

Performance drops significantly (20–30%) on open-ended questions:

| Model | Open-Ended Accuracy |
|---|---|
| **GPT-5-mini** | 75–80% |
| o4-mini | 75–80% |
| GPT-4o | ~60.8% (lowest) |

### Key Insights

1. **Reasoning models > Non-reasoning models**: o4-mini almost always respects key problem constraints and applies micro-decomposition (breaking problems into sequential local steps), whereas GPT-4o frequently proposes solutions that violate constraints.

2. **Newer models > Older models**: GPT-5-mini outperforms GPT-4o because it benefits from better-curated training data containing CTF-specific idioms (e.g., tcache safe-linking, ROT47, PNG structure manipulation). Rather than using generic high-frequency heuristics like GPT-4o, GPT-5-mini aligns with task-specific implicit rubrics.

3. **Web challenges are hardest**: Web tasks involve dynamic, tool-driven, multimodal knowledge (SQLi, XSS, file upload exploitation) that is harder to consolidate during LLM pre-training.

---

## 🔧 Core Prompts

### Knowledge Extraction
> Instructs an LLM to extract up to 2 distinct universal CTF knowledge points from each write-up, including applicable conditions and payload examples.

### Question Generation
> Converts extracted knowledge into single-choice questions with 4 options (1 correct + 3 distractors with matching format), and open-ended short-answer questions.

### Answer Evaluation
> Uses o3-mini as a judge to classify model answers as `correct` or `incorrect` based on strict equivalence to a reference answer.

Full prompts are available in [`prompts.py`](prompts.py) and the project report appendix.

---

## 📈 Future Work

- Evaluate additional recently-launched models to track improvement trends
- Develop effective CTF task transformations (t → t₁, t₂, …) to better probe model knowledge
- Enrich model knowledge through:
  - Targeted **model fine-tuning** on CTF-specific data
  - **Prompt chaining** with automated feedback loops
  - **Retrieval-Augmented Generation (RAG)** to embed missing procedural knowledge
- Investigate causal reasoning gaps in LLMs for dynamic exploit scenarios

---

## 📚 References

- Ji, Z., Wu, D., Jiang, W., Ma, P., Li, Z., & Wang, S. (2025). Measuring and augmenting large language models for solving capture-the-flag challenges. *arXiv:2506.17644*. https://arxiv.org/abs/2506.17644
- Shao, M., et al. (2024). NYU CTF Dataset: A Scalable Open-Source Benchmark Dataset for Evaluating LLMs in Offensive Security. *arXiv:2406.05590*.
- Yang, J., et al. (2023). Language agents as hackers: Evaluating cybersecurity skills with capture the flag. *Multi-Agent Security Workshop @ NeurIPS'23*.

---

## 📄 Citation

```bibtex
@article{ji2025measuring,
  title={Measuring and Augmenting Large Language Models for Solving Capture-the-Flag Challenges},
  author={Ji, Zimo and Wu, Daoyuan and Jiang, Wenyuan and Ma, Pingchuan and Li, Zongjie and Wang, Shuai},
  journal={arXiv preprint arXiv:2506.17644},
  year={2025}
}
```

---

## 📄 License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.
