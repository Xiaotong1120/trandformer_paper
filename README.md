# trandformer_paper
This is the repo for my transformer paper presentation.


## Background: Why FineWeb?

### The Need for High-Quality Pretraining Data
- Large Language Models (LLMs) rely on **autoregressive Transformer architectures**, where performance heavily depends on **pretraining dataset quality and scale**.
- Training data must be **large enough** for effective learning while **filtered well enough** to remove low-quality content.

### Data Sources & Challenges
- One of the most common data sources for LLM pretraining is **Common Crawl**, a publicly available web archive.
- However, **raw web data contains significant noise**, including:
  - **Boilerplate text** (e.g., website templates, ads)
  - **Gibberish content** and **spam**
  - **Massive duplication**, leading to inefficient training

### Why Filtering & Deduplication Matter
- **Low-quality web text** negatively impacts model performance, as most downstream tasks **do not involve noisy, redundant content**.
- **Over-filtering**, however, risks reducing dataset diversity and **removing useful information**.
- The tradeoff between **scale** and **quality** makes pretraining dataset curation a crucial problem.

### Limitations of Existing Datasets
Several public datasets have attempted to improve LLM pretraining data, but each has drawbacks:
- **OSCAR**: Uses a **fastText language classifier** and **line-level deduplication**, but lacks deep filtering.
- **C4**: Applies heuristic filters (e.g., removing lines without punctuation) but **does not scale well**.
- **The Pile**: A mix of sources but **contains redundancy** and lacks fine-grained filtering.
- **RefinedWeb**: Uses **Trafilatura** for extraction and **MinHash deduplication**, but **could further improve filtering**.

### The FineWeb Solution
- FineWeb introduces a **principled data curation pipeline** to produce a **15-trillion-token** high-quality dataset.
- Key innovations:
  - **Advanced Filtering**: Combining **heuristic rules** and **C4-like filters** to remove low-quality text.
  - **MinHash-based Deduplication**: **Per-crawl deduplication** instead of global deduplication to **preserve dataset diversity**.
  - **FineWeb-Edu Subset**: A **1.3T token dataset** optimized for **knowledge-intensive tasks**, significantly boosting performance on benchmarks like **MMLU & ARC**.

**FineWeb is designed to narrow the gap between open-source and proprietary LLM pretraining datasets by making high-quality data available to the research community.**
