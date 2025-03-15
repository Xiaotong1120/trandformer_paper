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

### Step-by-Step Construction of FineWeb Dataset

The authors built **FineWeb** incrementally, systematically validating each step's impact on dataset quality and model performance. Here’s the detailed breakdown of their process:

resources: https://huggingface.co/spaces/HuggingFaceFW/blogpost-fineweb-v1
---

## ① Text Extraction (Trafilatura from WARC vs. WET)

- **Problem**: Commonly used WET files retain boilerplate text (menus, ads, templates).
- **Solution**: Extracted text directly from raw WARC files using the **trafilatura** library, producing cleaner text data.
- **Effect**: Significant improvement in model performance compared to WET extraction (Fig. 1).

---

## ② Base Filtering (Filtered vs. Unfiltered WARC)

- **Problem**: Raw WARC-extracted text still contained noise (non-English text, adult content, short/repeated content).
- **Solution**: Applied basic filters:
  - URL blocklists (remove adult content)
  - fastText English-language classification (threshold ≥0.65)
  - Basic quality/repetition filters from MassiveText
- **Effect**: Provided clear performance boost compared to unfiltered WARC data (Fig. 2).

---

## ③ Deduplication (Individual Snapshot vs. Global MinHash)

- **Problem**: Web data includes massive duplication, which hurts efficiency and performance.
- **Solution tested**: Applied MinHash deduplication globally (across all crawls) vs. individually (per snapshot).
- **Findings**:
  - **Global deduplication** removed too much valuable (older, high-quality) content, keeping lower-quality newer data.
  - **Individual snapshot deduplication** improved performance by maintaining better data diversity and quality.
- **Conclusion**: Chose individual snapshot deduplication approach for optimal balance (Fig. 3 and Fig. 5).

---

## ④ Incorporating C4's Filters (Selective Application)

- **Problem**: Despite previous improvements, **C4 dataset** still outperformed FineWeb on certain benchmarks (e.g., HellaSwag).
- **Approach**: Analyzed and selectively adopted heuristic filters from C4:
  - Short lines removal
  - Curly brackets removal
  - JavaScript and policy text removal
  - Decided **not** to apply Terminal Punctuation filter (removes ~30% data, too aggressive)
- **Effect**: Applying selective C4 filters significantly improved benchmark performance while retaining more data (Fig. 6).

---

## ⑤ Developing Custom Heuristic Filters (Systematic Approach)

- **Problem**: Further improvement required beyond existing filters.
- **Solution**: Systematically identified new heuristic filters by comparing statistics of known high-quality vs. low-quality data:
  - Fraction of lines ending with punctuation (threshold ≤ 0.12)
  - Fraction of duplicated line characters (threshold ≥ 0.1)
  - Fraction of very short lines (<30 chars, threshold ≥ 0.67)
- **Effect**: These filters removed ~22% of tokens and further improved model performance, surpassing C4 performance (Fig. 7).

---

## 🎯 **Final Result: The FineWeb Dataset (15T tokens)**

Combining all the steps above, the authors constructed the final FineWeb dataset, achieving superior performance relative to existing open datasets (RefinedWeb, C4, Dolma, The Pile, SlimPajama, etc.). Each step contributed incrementally to improved quality and benchmark results (Fig. 9 and Fig. 10).

- **Final Steps Applied**:
  - WARC text extraction (trafilatura)
  - Base filtering
  - Individual snapshot deduplication (MinHash)
  - Selected C4 filters
  - Custom heuristic filters
  - Removal of personal identifiable information (emails, IP addresses)

---

This structured, step-by-step method demonstrates the authors' systematic approach to creating a highly refined, high-performing, and openly available dataset for pretraining large language models.
