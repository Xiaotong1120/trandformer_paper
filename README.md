# Presentation on "FineWeb Datasets: Decanting the Web for Finest Text Data at Scale" by Guilherme Penedo et al., Hugging Face, NeurIPS 2024.
Xiaotong Ma

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

**FineWeb is designed to narrow the gap between open-source and proprietary LLM pretraining datasets by making high-quality data available to the research community.**

## Data Collection & Curation

### Experience Setup
To ensure fair and reliable comparisons when evaluating data curation methods, the authors controlled:

- **Model Architecture:** Fixed (Llama, 1.71B parameters, GPT-2 tokenizer).
- **Training Scale:** Equal tokens and steps for all variants.
- **Randomness:** Two different seeds per variant to minimize random variation.
- **Evaluation Benchmarks:** Selected stable and representative tasks (e.g., MMLU, ARC, HellaSwag).

This setup guarantees differences in performance are due only to **data quality**, not experimental conditions.
