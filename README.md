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

## FineWeb-Edu: Creating an Educational Dataset with Synthetic Annotations

FineWeb-Edu is a **specialized subset of FineWeb**, explicitly filtered to contain educationally valuable content, designed to boost model performance on tasks involving reasoning and knowledge.

---

### Motivation and Background

- While the general **FineWeb dataset** significantly improves language model performance compared to earlier open datasets (RefinedWeb, C4, Dolma, etc.), there's an opportunity to further specialize data for **knowledge-intensive educational tasks** (e.g., standardized tests, general knowledge Q&A tasks).
- Recent closed-source models (e.g., Llama 3, Phi-3) use **synthetic annotations** to filter large-scale data, but this method had not previously been explored publicly on large-scale web data.

**The authors thus decided to test a novel filtering approach:**  
> *Using large-scale synthetic annotations generated by high-quality LLMs to build classifiers for educational content selection.*

---

### Step-by-Step Construction of FineWeb-Edu

#### Step 1: Generating Synthetic Educational Annotations (with Llama-3-70B)

- **Goal**: Identify documents suitable for educational purposes at scale (billions of documents).
- **Method**:  
  - Randomly selected **460,000 documents** from the FineWeb CC-MAIN-2024-10 snapshot.
  - Used the powerful pretrained instruction model **Llama-3-70B-Instruct** to assign each document a score (**0–5**) indicating its suitability as educational material.
  - Specifically instructed the LLM to rate documents based on their appropriateness for grade-school and middle-school students, deliberately avoiding highly specialized academic texts (e.g., research abstracts from arXiv), which could bias scoring towards overly technical or niche material.

#### Why "Synthetic" annotations?

- These labels are termed **"synthetic annotations"** because they were **not created by humans**, but generated automatically by the LLM. This method allows creating large-scale labels economically and efficiently.
- Previous approaches often used costly human annotations. Synthetic annotations significantly scale the data labeling process.

#### Step 2: Training a Classifier on Synthetic Data

To efficiently scale the labeling from thousands of samples to the entire 15-trillion-token FineWeb dataset, the authors trained a dedicated **classification model**:

- **Input**: Document embeddings from the publicly available embedding model (**Snowflake-arctic-embed-m**).
- **Training**: A linear regression model was fine-tuned using **410,000 synthetic annotations** (scores from Llama-3) for 20 epochs with a learning rate of 3e-4. The embeddings themselves were kept frozen.
- **Evaluation**: Performance was validated on a held-out validation set (50,000 samples), achieving a strong classification F1 score (**82% accuracy at threshold ≥ 3**).
- **Threshold Selection**: A classification threshold of **3** was empirically determined to provide the best trade-off between:
  - High performance on knowledge-intensive benchmarks (**MMLU, ARC, OpenBookQA**)
  - Minimal negative impact on general benchmarks (e.g., HellaSwag).

#### Step 3: Applying the Classifier at Scale

- The trained classifier was applied to the **entire 15-trillion-token FineWeb dataset**, identifying high-quality educational documents.
- This filtering step required **~6,000 GPU hours** on Nvidia H100 GPUs, demonstrating its large-scale feasibility.

- The final resulting **FineWeb-Edu dataset** comprises approximately **1.3 trillion tokens**, specifically enriched with educational content.

---

### Experimental Results and Model Performance

The authors conducted extensive ablation studies (350 billion-token scale experiments) to validate the quality and impact of FineWeb-Edu:

- **FineWeb-Edu substantially outperformed** all previously compared open web-based datasets on educational benchmarks:
  - **MMLU** (knowledge-intensive benchmark):  
    - Base FineWeb: ~33%  
    - FineWeb-Edu: **~37%** (**+12% relative improvement**)
  - **ARC** (reasoning-intensive benchmark):  
    - Base FineWeb: ~46%  
    - FineWeb-Edu: **~57%** (**+24% relative improvement**)
  - **OpenBookQA** and similar reasoning benchmarks also significantly improved.

- Remarkably, FineWeb-Edu matched or surpassed the performance of other large datasets such as **Matrix**, using nearly **10 times fewer tokens**. This strongly validated the efficiency and quality improvement brought by synthetic annotation-based filtering.

---

### Topic and Domain Analysis of FineWeb-Edu

To understand how the educational classifier influenced the content distribution, the authors conducted a detailed topic analysis:

- Embedded 50,000 random documents from both FineWeb and FineWeb-Edu into a vector space using sentence-transformer embeddings (**all-MiniLM-L6-v2**).
- Projected embeddings to 2D with **UMAP**, then clustered into 100 dense topics using DBSCAN clustering, labeled with Llama-3.1.

**Observations:**
- FineWeb-Edu classifier strongly favored:
  - **Education, Learning, Teaching**  
  - **History, Culture, Politics**  
  - Academic sources (e.g., Wikipedia articles, educational content).
- Significantly reduced representation of:
  - **Business, Finance, Law**
  - **Entertainment, Film, Theater**
  - **Places, Travel, Real Estate**

---

### Domain Fit Analysis (Perplexity Evaluation)

To further validate domain coverage, the authors compared the perplexity of FineWeb vs. FineWeb-Edu across various web and academic domains (Paloma domains):

- **FineWeb** generally performed better on broad, general internet domains (e.g., Reddit, Twitter, common web crawl sources).
- **FineWeb-Edu** outperformed on datasets heavily associated with **academic, technical, and educational content**:
  - Wikipedia-based corpora (WikiText-103, M2D2 Wikipedia)
  - Academic papers (M2D2 S2ORC, arXiv dataset)
  - Programming languages and coding-related topics (100 PLs dataset)

This clearly illustrates how the educational classifier shaped FineWeb-Edu towards a targeted educational domain.

---

### Key Takeaways and Contributions of FineWeb-Edu

- Developed a large-scale method using **synthetic annotations** generated by powerful LLMs to label educational quality systematically.
- Built an efficient, highly accurate classifier that scales economically across trillions of tokens, selecting high-quality educational content.
- Achieved substantial performance improvements on key **knowledge-intensive** and **reasoning-intensive benchmarks** (MMLU, ARC, etc.).
- Demonstrated a significant shift towards educational topics, verifying that data quality and thematic focus can be systematically enhanced through classifier-based filtering.

---

### Why FineWeb-Edu Matters

- FineWeb-Edu proves synthetic annotation can achieve high-quality filtering at scale, making it practical for the wider research community to replicate.
- Sets a new benchmark for openly accessible, specialized datasets, potentially driving future directions in dataset construction, model training, and research prioritization within educationally-oriented LLM applications.

## Bias Analyses in FineWeb and FineWeb-Edu

### Background
Language models commonly reflect biases present in their training datasets, especially biases toward historically sensitive or protected social subgroups.

### Findings (FineWeb)
- Contains moderate biases reflective of broader internet text, with relative overrepresentation of words linked to mainstream cultural norms (e.g., `'man'`, `'christian'`).
- Most noticeable biases involved associations between religious groups and intimacy-related topics (e.g., `'christian dating'`, `'jewish singles'`).

### Findings (FineWeb-Edu)
- Shows less biased associations, more relevant to educational contexts (e.g., historical associations such as `'man–king'`, biological/health contexts like `'woman–pregnancy'`).
- Reduces intimacy-based stereotypical associations, focusing instead on educational, historical, and health-related topics.

### Additional Results
- FineWeb-Edu achieves impressive results on knowledge-intensive benchmarks like MMLU (33.6% accuracy at just 38B tokens, significantly better than other large-scale datasets like Matrix).
- Domain coverage analyses indicate FineWeb-Edu effectively specializes in educational, programming, academic, and Wikipedia-related content, while FineWeb covers general web topics more broadly.

This analysis highlights the impact of dataset filtering choices on social biases and thematic content, emphasizing the importance of responsible and systematic dataset construction.

---
## Impact of FineWeb and FineWeb-Edu Datasets

### Impact of FineWeb

#### 1. **Improvement in Data Quality at Scale**
- FineWeb demonstrates a rigorous, systematic approach to data cleaning, setting a new standard for open-source datasets.
- Provides a large-scale (~15 trillion tokens), **high-quality text dataset**, significantly improving upon existing widely-used public datasets (e.g., C4, OSCAR, RefinedWeb).

#### 2. **Influencing Future Dataset Construction**
- Presents a clear, reproducible, and systematic pipeline (WARC extraction, rigorous filtering, snapshot-level deduplication, and heuristic-based quality enhancement).
- Establishes a benchmark methodology for other researchers to build upon, influencing future data curation efforts.

#### 3. **Bridging the Gap Between Open and Proprietary Datasets**
- Significantly narrows the gap between proprietary and open datasets, promoting transparency, reproducibility, and accessibility within the research community.
- Encourages more equitable, democratic development of powerful LLMs.

---

### Impact of FineWeb-Edu

#### 1. **Superior Performance on Educational and Knowledge-Intensive Tasks**
- Specifically optimized for tasks requiring extensive knowledge or reasoning capabilities, such as:
  - **MMLU**: Improved performance by approximately **12%** compared to the baseline FineWeb.
  - **ARC benchmark**: Achieved a remarkable relative improvement of **~24%**, clearly indicating significant advantages on reasoning tasks.

#### 2. **Demonstrating Effectiveness of Synthetic Annotation**
- Pioneers large-scale, systematic usage of synthetic annotations (automatic labels from powerful LLMs) for dataset filtering, a scalable approach previously unexplored publicly.
- Validates synthetic labeling as a practical, cost-effective way of dataset curation, setting a valuable precedent for future datasets.

#### 3. **Reduced Societal Bias and Improved Domain Representation**
- Reduces biases typically present in general web text, shifting dataset content towards educational, historical, scientific, and health-related material.
- Provides clearer educational context with fewer stereotypical associations (e.g., less intimacy-related biases, more historical and educational associations).

#### 4. **Shaping Future Research and Dataset Design**
- Sets a benchmark for educationally-focused datasets, influencing future LLM research towards domain-specific and task-oriented dataset creation.
- Opens possibilities for targeted LLM applications in education, tutoring, academic research, and knowledge-intensive tasks.

---

## Critical Analysis: Potential Issues with FineWeb & FineWeb-Edu

While **FineWeb** and **FineWeb-Edu** represent significant advancements in dataset construction, there are some critical aspects that warrant further scrutiny:

---

### **Main Concern: Reliability of Synthetic Annotations (Llama-3)**

- FineWeb-Edu heavily relies on synthetic annotations generated by **Llama-3-70B-Instruct**, without explicit human validation. 
- Authors validate annotation reliability only through **self-consistency** (classification F1 ~82%), measuring how well the classifier matches the synthetic labels provided by Llama-3 itself.
- **Potential issue**: 
  - The model’s internal biases or misconceptions may propagate directly into annotations. 
  - No human verification was conducted to ensure annotations align with human standards of educational quality.

**Possible improvements**:
- Introducing human or expert validation on a representative subset of annotations.
- Conducting comparative analyses between human-generated labels and model-generated labels to establish real-world annotation accuracy.

---

### **Other Noteworthy Concerns (briefly mentioned)**

#### **English-centric Data Construction**
- The pipeline primarily supports English-language content, limiting its applicability and relevance to multilingual or global contexts.

#### **High Computational Cost**
- The extensive resources required (~80k GPU hours) limit replicability, posing accessibility and fairness issues for smaller research teams or institutions.

#### **Transferability to Non-English Languages**
- Due to resource and language constraints, the current methodology offers limited guidance or reference value for constructing datasets in other languages.

#### **Residual Dataset Biases**
- Although some bias analyses were conducted, deeper or more subtle biases (racial, socioeconomic, political) were not thoroughly explored or addressed.

---

## Additional Resource Links
- [FineWeb Blogpost and Demo (Hugging Face)](https://huggingface.co/spaces/HuggingFaceFW/blogpost-fineweb-v1)
- [Trafilatura Library](https://github.com/adbar/trafilatura)
- [MinHash Deduplication (RefinedWeb)](https://github.com/huggingface/refinedweb)
- [C4 Dataset](https://huggingface.co/datasets/allenai/c4)
- [MassiveText Dataset and Methodology](https://github.com/bigscience-workshop/data_tooling)

---

## Citation for Paper
Penedo G, Kydlíček H, allal LB, et al. The FineWeb Datasets: Decanting the Web for the Finest Text Data at Scale. Published online 2024. doi:10.48550/arxiv.2406.17557
