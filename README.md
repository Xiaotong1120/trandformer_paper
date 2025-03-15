# Presentation on "FineWeb Datasets: Decanting the Web for Finest Text Data at Scale" by Guilherme Penedo et al., Hugging Face, NeurIPS 2024.
Xiaotong Ma  
resources: https://huggingface.co/spaces/HuggingFaceFW/blogpost-fineweb-v1

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

### Step-by-Step Construction of FineWeb

The authors developed FineWeb by incrementally refining web-data quality through careful experiments. Each step of the pipeline improved model performance on benchmarks, showing the importance of rigorous data curation.

---

#### ① Text Extraction (Trafilatura from WARC vs. WET)

#### Problem (Why extraction matters):
- **Common Crawl** data is typically available in two formats:
  - **WET files** (text-only), extracted with standard tools like `htmlparser`. However, WET files often contain substantial **boilerplate text**, such as navigation menus, headers, footers, and advertisements.
  - **WARC files**, which retain the original, raw HTML data including metadata. These allow customized and potentially cleaner text extraction.

- Boilerplate text from WET files negatively impacts model training, causing the model to memorize repetitive, non-informative patterns rather than learning meaningful language.

#### Authors' Solution:
- The authors experimented by extracting clean, high-quality text directly from **WARC files** using the open-source library **trafilatura**.
- **Trafilatura** specifically targets and removes boilerplate content (menus, navigation text, ads), providing **significantly cleaner and richer textual data** for training language models.

#### Effect and Validation:
- Authors compared models trained on standard WET-extracted data versus models trained on trafilatura-extracted WARC data, without applying additional filtering or deduplication beyond basic English-language filtering.
- The experiment clearly showed a **significant improvement in model performance** when using trafilatura-extracted WARC data (see Fig. 1 in the paper).

---

### ② Base Filtering (Filtered vs. Unfiltered WARC)

#### Problem (Noise remains after extraction):
- Even after careful extraction with trafilatura, raw WARC-extracted data contained significant noise:
  - Non-English text
  - Adult or inappropriate content
  - Short or repetitive text lacking meaningful information

- Such noise reduces the dataset’s effectiveness, decreasing the efficiency of model training and the overall quality of learned representations.

#### Authors' Solution:
- The authors applied several **basic yet crucial filtering steps** inspired by existing datasets (RefinedWeb, MassiveText):
  - **URL blocklists**: removed adult and inappropriate websites.
  - **Language filtering (fastText)**: retained only documents classified as English with ≥0.65 confidence.
  - **Quality and repetition filters**: basic thresholds from MassiveText to remove clearly low-quality documents (e.g., repetitive short phrases).

#### Effect and Validation:
- Compared models trained on base-filtered WARC data versus unfiltered data.
- Clear performance improvement was observed, proving that even basic filtering significantly boosts data quality and downstream performance.

---

### ③ Deduplication (Individual Snapshot vs. Global MinHash)

#### Problem (The duplication dilemma):
- Web content is highly duplicated due to mirrors, aggregation websites, SEO spam, and re-posting, which negatively impacts efficiency and model generalization.
- Initially, authors believed global deduplication (across all crawls at once) would significantly enhance quality by removing all duplicate content.

#### Experimental Analysis:
- **Global deduplication**:  
  - Removed too much valuable content because older, frequently duplicated but high-quality content (like detailed informative articles) was disproportionately deleted.
  - Accidentally kept newer but lower-quality content (SEO spam, short advertising text), reducing overall quality.

- **Individual snapshot deduplication**:  
  - Deduplicated each crawl snapshot separately, preventing older high-quality documents from being removed due to duplication in newer crawls.
  - Preserved content diversity, maintaining valuable high-quality articles and information.

#### Decision and Justification:
- Individual snapshot deduplication outperformed global deduplication in experiments, clearly improving benchmark performance.
- Authors thus adopted individual deduplication as the final approach.

---

### ④ Incorporating C4's Filters (Selective Application)

#### Problem (Performance gap remains):
- After the above steps, FineWeb matched RefinedWeb but **C4 still outperformed** FineWeb on certain benchmarks, particularly **HellaSwag**, a reasoning-intensive task.
- This indicated additional filtering strategies used by C4 might still be beneficial.

#### Authors' Solution (Learning from C4):
- Carefully analyzed the heuristic filters C4 applied, including:
  - Removing very short lines/documents.
  - Eliminating lines containing curly brackets `{}` (often non-language code snippets).
  - Removing JavaScript and privacy-policy-related text.
  - Terminal punctuation filter: removing lines without sentence-ending punctuation.

- The authors empirically tested these filters and found:
  - Terminal punctuation filtering improved quality significantly but removed too much valuable content (~30% tokens lost).
  - Other filters provided modest but effective improvements with less severe data loss (~7%).

#### Decision and Validation:
- Decided to adopt **all C4 filters except terminal punctuation**, balancing data quality and dataset size.
- This resulted in improved model performance without sacrificing excessive data.

---

### ⑤ Developing Custom Heuristic Filters (Systematic Approach)

#### Problem (Further improvements needed):
- Despite incorporating C4 filters, authors believed further, more refined filtering could yield additional performance gains.

#### Systematic Methodology:
- Instead of relying solely on intuitive inspection, authors developed a **systematic method**:
  1. Collected >50 statistical metrics from both high-quality and low-quality data sets (e.g., lines per document, average word length, repetitive content ratios).
  2. Compared distributions of these metrics between high-quality (individually deduplicated snapshot) and low-quality (globally deduplicated snapshot) datasets.
  3. Identified meaningful thresholds empirically, focusing on values where low-quality datasets showed clearly different statistical distributions.

- Three particularly effective new filters emerged:
  - **Fraction of lines ending with punctuation ≤ 0.12** (indicative of poorly formatted documents).
  - **Fraction of duplicated line characters ≥ 0.1** (typical of repetitive boilerplate content).
  - **Fraction of very short lines (<30 chars) ≥ 0.67** (documents mostly consisting of fragmented or trivial content).

#### Experimental Validation:
- Individually, these filters removed ~22% of data yet significantly improved model performance (~1% improvement on aggregate benchmarks), surpassing even the C4 dataset’s performance.

---

### **Final FineWeb Dataset (Combined Impact)**

- Each of these carefully validated steps incrementally improved FineWeb’s quality and benchmark performance.
- The final **15-trillion-token FineWeb** dataset combined all above methods (WARC extraction, base filtering, individual MinHash, selected C4 filters, custom heuristic filters), resulting in superior performance compared to major existing open datasets (RefinedWeb, C4, Dolma, The Pile, SlimPajama, etc.).
- FineWeb demonstrates the significant value in rigorous data curation and filtering to enhance large language model training.

---

This deeper explanation clearly communicates the rationale, approach, experiments, and results of each step, providing listeners with a full understanding of how FineWeb was methodically developed and optimized.
