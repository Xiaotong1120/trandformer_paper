# Presentation on "FineWeb Datasets: Decanting the Web for Finest Text Data at Scale" by Guilherme Penedo et al., Hugging Face, NeurIPS 2024.
Xiaotong Ma

## Background: Why FineWeb?

### 🚨 The Problem: High-Quality Data is Crucial but Scarce
- Large Language Models (LLMs), typically built using **autoregressive Transformer architectures**, heavily depend on the **quality and scale of pretraining data**.
- Ideally, datasets should be **large enough** to enable robust learning and **high-quality enough** to avoid degrading performance with noisy information.

### 🌐 Common Crawl: Opportunities & Challenges
- A popular source for training LLMs is the **Common Crawl**, a publicly available, vast collection of web data.
- However, raw Common Crawl data suffers from several quality issues, including:
  - Significant amounts of **boilerplate text** (navigation menus, templates, ads, copyright notices)
  - **Spam and nonsensical content** (SEO keyword stuffing, gibberish)
  - **Extensive duplication**, where identical content appears repeatedly across many web pages, wasting computational resources.

> For example, copyright disclaimers from a news website can appear verbatim on thousands of different web pages, adding no value to model training.

### 🛠️ Why Filtering & Deduplication Are Essential
- **Low-quality web data negatively impacts model performance** since most real-world applications do not involve repetitive or meaningless content.
- However, **excessive filtering** also poses risks—potentially **reducing dataset diversity** and unintentionally removing valuable information.
- Balancing the trade-off between **scale and quality** makes the curation of pretraining datasets both challenging and critical.

### ⚠️ Limitations of Existing Datasets
Several existing public datasets have attempted to address these issues, each with its shortcomings:

- **OSCAR**: Uses **fastText for language filtering and basic line-level deduplication** but lacks deep quality filtering, retaining considerable low-quality content.
- **C4**: Employs heuristic filtering methods (e.g., removing lines without terminal punctuation) but is coarse-grained and does not easily scale to larger datasets.
- **The Pile**: Combines multiple data sources but suffers from internal **redundancy and coarse filtering**, limiting its overall effectiveness.
- **RefinedWeb**: Utilizes high-quality extraction (with **Trafilatura**) and **MinHash deduplication**, but its filtering strategy still has room for improvement.

### 🚀 The Goal of FineWeb
**FineWeb aims precisely at addressing these gaps** by:
- Systematically investigating and empirically validating more effective filtering and deduplication strategies.
- Offering openly accessible, transparent, high-quality datasets to help bridge the gap between proprietary datasets used by commercial models and datasets publicly available to the research community.
