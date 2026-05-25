# Optimization of Retrieval in RAG Systems via Active Context Injection and Semi-Structured Data Flattening

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python 3.10+](https://img.shields.io/badge/Python-3.10+-blue.svg)](https://www.python.org/downloads/)
[![Docker](https://img.shields.io/badge/Docker-Supported-blue.svg)](https://www.docker.com/)
[![Hội thảo: ICT 2026](https://img.shields.io/badge/Presented%20at-ICT%202026-orange.svg)](#)

Source code and dataset for the paper: *"Tối ưu hóa truy xuất trong hệ thống RAG thông qua kỹ thuật bổ sung ngữ cảnh chủ động và làm phẳng dữ liệu bán cấu trúc"* presented at the 11th National Scientific Conference on Information and Communication Technology (ICT), Dong Thap, Vietnam (May 2026).

---

## 📋 Overview

This study addresses the context loss and structural fragmentation challenges encountered when processing semi-structured data (e.g., JSON, long lists) in Retrieval-Augmented Generation (RAG) pipelines. 

Traditional chunking strategies often break hierarchical logical relationships, stripping lower-level tokens of their parent metadata. We propose a data preprocessing framework combining **Natural Language Flattening** and **Active Context Injection** to ensure semantic independence for each chunk before embedding vectorization.

### Key Contributions
* **Data Representation Layer Optimization:** Converts key-value structures into coherent natural language propositions.
* **Active Context Injection:** Periodically injects high-level identification metadata into localized logical blocks.
* **Controlled Chunking Strategy:** Eliminates arbitrary text splitting across logical units without requiring complex parent-child retrieval architectures.

---

## 🏗️ Data Preprocessing Pipeline


```

┌──────────────────────┐     ┌──────────────────────┐     ┌──────────────────────┐
│   Raw Data (JSON)    │     │   Data Flattening    │     │  Context Injection   │
│  Hierarchical Tree   │───▶ │  Natural Language   │───▶ │  Injected Metadata   │
│  (Fragmented Chunks) │     │     Propositions     │     │ (Semantic Autonomy)  │
└──────────────────────┘     └──────────────────────┘     └──────────────────────┘
│
▼
┌──────────────────────┐     ┌──────────────────────┐     ┌──────────────────────┐
│  Evaluation (RAGAS)  │     │   Inference Model    │     │  Vector DB Storing   │
│  GPT-4o-mini Judge   │◀─── │  Llama-3.1-8b (API)  │◀─── │  Gemini Embedding    │
└──────────────────────┘     └──────────────────────┘     └──────────────────────┘

```

---

## 📊 Experimental Evaluation

The methodology was validated using a real-world admissions dataset from Nha Trang University and evaluated via the **RAGAS framework** (LLM-as-a-Judge with `GPT-4o-mini`).

### Metrics Summary (Average over 50 Ground Truth Queries)

| Evaluation Metric | Baseline 1 (Raw JSON) | Baseline 2 (Raw Markdown) | Proposed Approach |
| :--- | :---: | :---: | :---: |
| **Context Precision** | 0.7933 | 0.7817 | **0.8556 (+6.23%)** |
| **Faithfulness** | 0.5484 | 0.6371 | **0.7990 (+16.19%)** |
| **Answer Correctness** | 0.5524 | 0.6181 | **0.7017 (+8.36%)** |

*Note: The metadata repetition introduces an ~18% increase in total vector storage tokens, with a marginal latency trade-off (~120ms), while significantly improving precision and minimizing hallucinations.*

---

## 🛠️ System Configuration & Environment

* **Orchestration Platform:** AnythingLLM (Dockerized Deployment)
* **Host OS:** Linux (Ubuntu 24.04)
* **Embedding Core:** Google Gemini Embedding 001 API
* **Inference Engine:** Llama-3.1-8b-instant (Cloud API Gateway / Local Deployment Compatible)
* **Chunk Parameters:** Size = 1000 tokens, Overlap = 200 tokens

---

## 📝 Citation

If you utilize this approach or dataset in your research, please cite the conference paper:

```bibtex
@inproceedings{ngo2026rag,
  title={Tối ưu hóa truy xuất trong hệ thống RAG thông qua kỹ thuật bổ sung ngữ cảnh chủ động và làm phẳng dữ liệu bán cấu trúc},
  author={Ngo, Tuong-Nghi and Lê, Thị Bích Hằng and Nguyễn, Đình Hưng},
  booktitle={Hội thảo khoa học Quốc gia về Công nghệ thông tin và Truyền thông (ICT)},
  year={2026},
  address={Đồng Tháp, Việt Nam}
}

```

---

## 📜 License

This project is licensed under the MIT License - see the LICENSE file for details.
