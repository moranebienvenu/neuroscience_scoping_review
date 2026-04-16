# Scoping Review: Reproducibility Practices in Neuroscience (2015–2025)

## Rationale

Reproducibility has been recognized as a central challenge in neuroscience, with multiple studies reporting difficulties in replicating experimental findings. Over the last decade, initiatives promoting open science practices, including code and data sharing, have emerged to address this issue. However, the extent to which these practices have been adopted across different neuroscience subfields remains unclear.

This scoping review aims to systematically map the evolution of reproducibility practices in neuroscience publications from 2015 to 2025, focusing specifically on the presence of code and data sharing. By examining trends over time and identifying which subfields are more likely to implement open science practices, this study seeks to provide a quantitative overview of the field's trajectory towards enhanced transparency and reproducibility. Rather than evaluating scientific outcomes, the review emphasizes the availability and accessibility of reproducible research artefacts, such as repository links, open-source code, and shared datasets, detectable through titles and abstracts.

## Objectives

This scoping review was conducted to systematically map the evolution of reproducibility practices in neuroscience publications in this last decade, as well as to identify any existing neuroscience subfields gaps in the adoption of reproducible research practices. The following research question was formulated:

> *What is known from the literature published between 2015 and 2025 about reproducibility practices in neuroscience, particularly regarding data and code sharing, and what factors influence the adoption of these practices across different subfields?*

## Protocol and Registration

This review follows the framework proposed by the Joanna Briggs Institute and the Preferred Reporting Items for Systematic Reviews and Meta-analysis extension for scoping reviews (PRISMA-ScR). The GitHub repository with all pipeline code is freely available at: [https://github.com/moranebienvenu/literature_review_pipeline](https://github.com/moranebienvenu/literature_review_pipeline).

## Eligibility Criteria

To be included in the review, papers needed to be neuroscience articles. These articles were included if they were: published between 2015 and 2025, written in English, and openly accessible from the OpenAlex research interface. Papers were excluded if they were not primarily focused on neuroscience. The goal was to detect sharing behaviour embedded in primary research articles.

## Information Source

All articles were retrieved from OpenAlex, a free and open bibliographic database providing structured metadata for over 480 million scholarly works. OpenAlex was selected as the sole source due to its open programmatic access via a REST API, inclusion of full-text abstracts, and concept-level tagging for neuroscience. The search was limited to publications between 2015 and 2025, in the neuroscience domain (OpenAlex concept ID C41008148), and open-access research articles. No additional sources such as reference scanning or hand-searching of journals were used. The final search was executed on March 26th, 2026.

## Search

Queries were constructed first using one filter and secondly, after a pre-analysis, using two complementary filters applied to the title and abstract fields. The first filter targeted neuroscience subject matter:

```
brain OR neuroscience
```

The second filter targeted reproducibility-relevant content:

```
reproducible OR reproducibility OR "data sharing" OR "code sharing" OR github 
OR repository OR "data availability" OR "code availability" OR "supplementary data" 
OR "shared dataset"
```

This second filter allows the inclusion of articles with any explicit mention of reproducibility or sharing practices in their metadata, without restricting to articles whose primary topic is open science, to allow fine clustering of subfields which have a greater tendency to share data or code.

Additional constraints were applied at the API level:

- Publication years: 2015–2025
- Article type: research article only (excluding reviews, editorials, preprints)
- Open-access status: open-access only (`is_oa: true`)
- Domain concept: Neuroscience (OpenAlex concept ID C41008148)

## Selection of Sources of Evidence

To select sources of evidence for this scoping review, articles were retrieved from the initial search query sent to the OpenAlex API. After filtering for minimum abstract length (≥50 characters), articles were retained for semantic analysis. Articles lacking sufficient textual metadata were excluded.

## Data Charting Process

Given the large scale of the dataset, manual screening of all articles was not feasible. Data charting was performed using a structured and automated extraction pipeline implemented in Python. Bibliographic metadata and abstract text were retrieved via the OpenAlex API and stored in a standardized DataFrame (`.csv` file). Code and data sharing indicators were identified using predefined regular-expression (regex) rules applied to titles and abstracts. The detection rules targeted mentions of specific repositories (e.g., GitHub, Zenodo, OSF) and natural-language phrases indicating sharing practices.

To ensure reliability, the automated detection rules were validated on a manually reviewed sample of 25 articles using the RRInsights custom GPT []{cite}`Karakuzu2024` tool prior to full deployment. Each article was then categorized into one of four mutually exclusive sharing categories, as shown in Table 3.1.

:::{table} Sharing category classification scheme.
:label: tbl-sharing-categories

| Category    | Code | Data | Description                                           |
|-------------|------|------|-------------------------------------------------------|
| Code + Data | Yes  | Yes  | Article shares both code and data                     |
| Code Only   | Yes  | No   | Code shared; no data sharing detected                 |
| Data Only   | No   | Yes  | Data shared; no code sharing detected                 |
| Neither     | No   | No   | No sharing artefact detected in abstract              |

:::

## Data Items

The following data items were extracted from each included article: bibliographic metadata (OpenAlex ID, DOI, title, publication year, venue, citation count, article type, open-access status), authorship and institutional information, abstract text, and OpenAlex concept classifications. In addition, variables related to research sharing practices were extracted, including binary indicators of code sharing and data sharing, repository platform mentions, keyword-based availability statements, and the final four-level sharing classification as shown in Table 3.1.

## Critical Appraisal of Individual Sources of Evidence

A formal critical appraisal of individual sources of evidence was not conducted, as the objective of this review was to map sharing practices rather than evaluate methodological quality.

## Synthesis of Results

The extracted data were synthesized using a combination of descriptive statistical analyses and computational text mining approaches.

First, quantitative summaries were generated to describe overall sharing practices by a sample of 1,000 neuroscience articles per year from 2015 to 2025 (11,000 articles in total), including the total number and proportion of articles sharing code and/or data, yearly sharing rates (2015–2025), citation statistics, and repository usage frequencies (e.g., GitHub, Zenodo, OSF). Sharing categories were also summarized using counts and percentages.

Temporal trends were analyzed by computing annual frequencies and proportions of sharing practices. These results were visualized using stacked bar charts (absolute counts) and line plots (percentage trends over time).

To examine whether sharing practices differ across neuroscience sub-disciplines, a semantic clustering pipeline was applied to the 2,898 articles with sufficient abstract content, retrieved with the two-filter queries to OpenAlex, leading to a total sample size of 4,429 articles. The pipeline consisted of four sequential steps: (1) semantic embedding via the SPECTER2 model, (2) dimensionality reduction via UMAP, (3) density-based clustering via HDBSCAN, and (4) automatic cluster naming via the Claude API. This approach, illustrated in Figure 3.1, was inspired by the SAKURA plot methodology described by Karakuzu et al. (2025), adapted here for large-scale bibliometric analysis of neuroscience reproducibility using OpenAlex.

:::{figure} static/fig3_1_pipeline.png
:label: fig-pipeline
:align: center
:width: 90%

Semantic clustering pipeline for neuroscience reproducibility literature. The four sequential steps are: (1) SPECTER2 embedding, (2) UMAP dimensionality reduction, (3) HDBSCAN clustering, and (4) automatic cluster naming via Claude API.

:::

### Text Embedding: SPECTER2

Articles were represented using SPECTER2 embeddings, a transformer-based model specifically designed for scientific literature. Each article's title and abstract were concatenated with a separator token (`[SEP]`) to form the input. SPECTER2 generates 768-dimensional embeddings that capture semantic relationships between articles, including citation-informed context.

Embeddings were computed in batches of 32 on CPU, L2-normalized, and saved incrementally to disk to prevent data loss in case of interruptions, via the Hugging Face space ([Mobien/neuro-reproducibility](https://huggingface.co/spaces/Mobien/neuro-reproducibility)).

In addition, prior to SPECTER2 embedding, non-informative terms such as journal names and recurring publication-related expressions were removed to improve semantic coherence and prevent clustering artefacts driven by venue-specific terminology.

### Dimensionality Reduction: UMAP

The high-dimensional SPECTER2 embeddings were projected into a two-dimensional space using UMAP (Uniform Manifold Approximation and Projection) for visualization and clustering. Parameters were set as follows: `n_neighbors = 30`, `min_dist = 0.0`, `metric = cosine`, `random_state = 42`. The `random_state` parameter was set to ensure reproducibility of the UMAP projection. A cosine metric was selected because it captures semantic similarity between document vectors more reliably than Euclidean distance in high-dimensional text spaces. This 2D representation provides a human-interpretable map of the semantic landscape of neuroscience reproducibility literature, highlighting clusters of thematically related articles.

### Clustering: HDBSCAN

HDBSCAN (Hierarchical Density-Based Spatial Clustering of Applications with Noise) was applied to the 2D UMAP projection to identify thematic clusters without requiring a pre-specified number of clusters. Parameters: `min_cluster_size = 60`, `min_samples = 20`. These settings were selected to produce clusters of meaningful size (≥60 articles) while avoiding over-fragmentation of the semantic space. The `min_samples` parameter further required each article to be embedded within a local neighborhood of at least 20 nearby articles to be considered part of a stable cluster, thereby reducing the formation of spurious or weakly defined groups.

Articles not sufficiently embedded within a dense neighborhood were labeled as noise (`-1`). Clusters with centroids exhibiting high cosine similarity (>0.99) in the original 768-dimensional SPECTER2 embedding space were merged to reduce redundancy, ensuring that semantically overlapping clusters are consolidated into a more coherent set of thematic domains.

The initial HDBSCAN run detected 16 clusters and 163 noise points sharing code (5.63% of valid articles). Cluster names were assigned using the Claude API. The 16 resulting thematic clusters, their article counts, and their sharing rates are shown in Figure 3.2.

:::{figure} static/fig3_2_clusters.png
:label: fig-clusters
:align: center
:width: 90%

Thematic clusters identified by HDBSCAN with sharing statistics. *n* = total articles in cluster across full dataset (*n* = 2,898); sharing rate [%]: articles sharing code and/or data per cluster.

:::

### Automatic Cluster Naming: Claude API

Cluster labels were generated automatically using the Claude language model (Sonnet 4.6, Anthropic). For each cluster, the five articles closest to the cluster centroid in the SPECTER2 embedding space were selected as representative documents. Their titles and truncated abstracts were provided to the model, which was prompted to infer a concise thematic label summarizing the cluster. The following prompt was used:

```
You are a scientific librarian specialising in neuroscience.
Below are 5 representative papers from a neuroscience research cluster 
identified by semantic analysis.
[Title + abstract snippets]
Give a concise thematic label of 2 to 5 words that best summarizes 
the neuroscience topic of this cluster.
Reply with the label only — no explanation, no punctuation, no numbering.
```

This procedure allows clusters to be labeled automatically based on representative articles while minimizing subjective manual interpretation. By constraining the model output to short thematic phrases (3–5 words) and prohibiting explanatory text, the resulting labels remain concise, consistent, and suitable for figure legends and quantitative analysis.

## Selection of Sources of Evidence

Figure 3.3 presents the article selection workflow for the first metadata analysis with 11,000 articles. A duplication filter was applied to remove duplicate articles across years, yielding a final sample of 10,967 articles.

:::{figure} static/fig3_3_prisma_first.png
:label: fig-prisma-first
:align: center
:width: 80%

Flow diagram of article identification with the first filter, providing an overview of sharing trends over the last decade.

:::

Figure 3.4 presents the article selection workflow for the second analysis using the two complementary OpenAlex filters. This query retrieved 4,429 articles used for filtering and semantic clustering (HDBSCAN). Of these, only 2,898 articles with valid abstracts were retained for clustering, and 575 articles for subfield analysis.

:::{figure} static/fig3_4_prisma_second.png
:label: fig-prisma-second
:align: center
:width: 80%

Flow diagram of article identification, screening, and semantic clustering for the subfield analysis.

:::

## Characteristics of Sources of Evidence

Of the 10,967 articles retrieved, 335 (3.1%) reported at least one form of open sharing (Table 3.2). Data sharing was the most prevalent practice, followed by code-only sharing, and sharing both code and data simultaneously.

:::{table} Distribution of sharing categories across the 10,967 retrieved articles.
:label: tbl-sharing-distribution

| Sharing Category | N articles | % total |
|------------------|-----------|---------|
| Code + Data      | 12        | 0.1%    |
| Code Only        | 133       | 1.1%    |
| Data Only        | 200       | 1.8%    |
| Neither          | 10,632    | 96.9%   |

:::

GitHub emerged as the most frequently referenced platform for code sharing detection, appearing 103 times across the abstracts of the 335 articles identified as sharing code. This frequency reflects GitHub's dominance as the de facto repository for computational neuroscience workflows. By comparison, Jupyter was mentioned in only 8 abstracts among articles reporting code sharing.

In contrast, data sharing was predominantly signaled through informal narrative expressions rather than references to specific repositories. Across the abstracts, 207 occurrences were detected for phrases indicating data availability, such as "data available," "dataset provided," "raw data," "data accessibility," "publicly available data," "data repository," "supplementary data," and "open data." References to named data repositories Zenodo and OSF were comparatively rare, appearing in fewer than 6 articles combined. This discrepancy suggests that authors more frequently signal data availability through descriptive statements rather than structured repository links, which may complicate automated reproducibility audits.

As illustrated in Figure 3.5, sharing practices increased over the 2015–2025 period, with the number of articles containing a sharing component rising from 10 out of 997 in 2015 to 52 out of 997 in 2025 — a fivefold increase over the decade. The sharing rate rose from 1.0% in 2015 to 5.2% in 2025. This trajectory aligns with the broader open science movement and the progressive adoption of data availability mandates by major journals. Notably, the journals contributing the most articles to the corpus included *Scientific Reports* (n = 648), *NeuroImage* (n = 591), and *Frontiers in Neuroscience* (n = 295), all of which have established policies encouraging or requiring data availability statements.

:::{figure} static/fig3_5a_stacked_bars.png
:label: fig-sharing-trends-a
:align: center
:width: 75%

**(a)** Annual distribution of articles reporting code only, data only, or combined open sharing practices across the study period (2015–2025).

:::

:::{figure} static/fig3_5b_line_plot.png
:label: fig-sharing-trends-b
:align: center
:width: 75%

**(b)** Temporal trends in sharing rates (%) by category (2015–2025).

:::

A notable increase in data sharing is observed in 2022 compared to 2021. One possible explanation relates to the COVID-19 pandemic. During the pandemic, open science practices became essential to accelerate research and global collaboration, with the OECD (2020) reporting unprecedented levels of open-access publishing and research data sharing. However, while open data practices intensified from 2020 onward, much of this early sharing occurred through preprints and collaborative platforms rather than formal journal publications. The more pronounced increase observed in 2022 may therefore reflect the delayed institutionalization of these practices within journal policies. Additionally, the growing integration of AI-assisted tools such as GitHub Copilot and ChatGPT, widely adopted from 2022–2023, likely lowered technical barriers to code documentation and data organization, facilitating compliance with open sharing norms. The increasing trend observed post-2023 may further reflect a continuous institutional push toward open science and research transparency.

## Results of Individual Sources of Evidence

The full list of included articles (n = 11,000), along with their associated metadata (title, journal, publication year, DOI, and sharing category), is provided in Supplementary Material ([Supplementary File S1](content/Supplementary_File_S1.csv)).

## Synthesis of Results

To examine whether open sharing practices are evenly distributed across neuroscience subfields or concentrated in specific domains, a semantic clustering analysis was performed on the 2,898 articles retained after HDBSCAN filtering. Figure 3.6 shows the UMAP projection of the 738 articles reporting at least one sharing practice, colored by thematic cluster, revealing a structured landscape where sharing behaviors are far from uniform across the field.

Substantial heterogeneity in sharing rates was observed across thematic clusters, consistent with the hypothesis that reproducibility culture is discipline-specific rather than uniformly distributed across neuroscience. The highest sharing rates were observed in the *Open NeuroScience* cluster (49.2%) and the *Personalized Neuroimaging Brain Fingerprinting* cluster (30.5%), both benefiting from mature discipline-specific sharing infrastructures. The *Brain MRI Classification* cluster (24.3%) and *Brain MRI Segmentation* cluster (23.3%) also showed above-average sharing rates, likely driven by the availability of well-established open-source toolboxes (Spinal Cord Toolbox, FreeSurfer, MRtrix) that normalize code-sharing within these communities.

The largest cluster with a sharing component, *Open NeuroScience* (n = 150), is both the biggest and clearly defined in the UMAP projection, indicating strong semantic coherence (Figure 3.6). It likely contains articles focused on methodological contributions — tools, pipelines, and frameworks for reproducible neuroscience research — rather than primary empirical findings. Its prominence within the sharing subset suggests that reproducibility practices are mainly concentrated in this methodologically aware segment of the literature.

Beyond this cluster, sharing practices appear across diverse thematic domains, including *Brain Tumor MRI Classification* (n = 60), *EEG BCI Research* (n = 74), and *AI Frameworks* (n = 54), as well as clinically oriented clusters such as *Brain Organoid Disease Modeling* (n = 14). A persistent gap exists in sharing practices between clinical neuroscience and BCI or quantitative imaging, with higher rates of sharing observed in subfields employing AI or informatics.

:::{figure} static/fig3_6_umap.png
:label: fig-umap
:align: center
:width: 85%

UMAP projection of neuroscience articles reporting code and/or data sharing, colored by thematic cluster (n = 732 articles; 2015–2025).

:::

Notably, 163 articles were classified as unclustered by HDBSCAN, suggesting that a substantial fraction of sharing articles occupy isolated or ambiguous semantic positions, potentially reflecting interdisciplinary work or atypical use of reproducibility language.

To further illustrate the diversity of sharing practices across clusters, a selection of representative articles was examined. Within the *Open Science* cluster, Poldrack et al. (2019) argue how data and code sharing are central to reproducibility in data analysis, evoking initiatives such as the Human Connectome Project (HCP) and the functional MRI Data Center (fMRIDC) as pioneers of data sharing in the neuroimaging community, along with the use of BIDS — a data science concept describing how a dataset is organized and annotated to maximize reusability. They also highlight the emergence of a consistent open ecosystem based on Python that enables reproducible data analysis and visualization.

In the *Deep Learning MRI Segmentation* cluster, Magadza & Viriri (2023) share only their code via GitHub for the use of nnU-Net for brain tumor segmentation, while Getmanskaya et al. (2021) provide a more comprehensive sharing package encompassing both U-Net code on GitHub and multiple open electron microscopy datasets. Conversely, Yang et al. (2023) describe their federated learning framework in methodological detail but provide no direct code link, relying instead on dataset references embedded in the text — a practice that signals data awareness without ensuring reproducibility.

Beyond abstract-level detection, a LLM review of 25 articles was conducted with the custom GPT RRInsights []{cite}`Karakuzu2024` to assess sharing practices in their broader publication context. This qualitative examination revealed a recurrent structural pattern: even when code, data, and the article itself are technically accessible, they are systematically hosted across disparate platforms — typically a journal website, a GitHub repository, and a third-party data archive such as Zenodo or OSF. This decentralization of research artefacts creates a non-trivial reproducibility barrier: a researcher attempting to replicate a study must independently locate, version-match, and integrate components spread across multiple repositories, often without explicit cross-referencing between them. Accessibility, in this sense, does not equate to reproducibility — a distinction that current sharing metrics, including those used in this review, cannot fully capture. Furthermore, none of the articles retrieved manually used interactive figures, allowing the reader to engage directly with the data; everything was static, without a possibility of direct verification of the authors' claims.

## Summary of Evidence

This scoping review mapped the evolution of reproducibility practices in neuroscience publications from 2015 to 2025, focusing on code and data sharing. Across the 10,967 initially retrieved articles, only 3.1% reported at least one sharing practice, with data sharing (1.8%) slightly more common than code sharing (1.1%) and combined sharing rare (0.1%). The overall trend shows a clear increase in sharing practices over the decade, particularly after 2019, reflecting both the influence of the COVID-19 pandemic on open research and the growing integration of AI-assisted tools facilitating code and data management.

Semantic clustering of the 2,898 articles with sufficient abstract content revealed 16 thematic clusters. The largest, *Open Science*, was primarily methodologically focused, suggesting that reproducibility efforts are concentrated in domains already engaged with open practices. Other clusters — including *Brain MRI Segmentation*, *Brain Tumor Classification*, and *EEG BCI* — show that sharing occurs across diverse subfields, although clinically oriented clusters such as *Brain Organoid* remain underrepresented. These findings highlight gaps in the adoption of reproducibility practices in clinical neuroscience, despite growing adoption in computational and methods-focused research.

Qualitative examination of representative articles further revealed variability in how sharing is implemented: some studies provide fully integrated code and data packages, while others host materials across multiple platforms, creating potential barriers to actual reproducibility.

## Limitations

This scoping review has several limitations. First, article detection was limited to abstract-level mentions, meaning that sharing artefacts reported only in methods sections, supplementary materials, or data availability statements appended after the abstract were not captured, likely leading to an underestimation of actual sharing rates. Second, the keyword-based regex patterns were designed to maximize recall, which may have introduced false positives (e.g., statements like "data available upon request" being flagged as open sharing). Third, restricting the corpus to open-access articles may introduce selection bias, as open-access publishing is often associated with institutional mandates and funder requirements that encourage sharing; sharing rates in the broader neuroscience literature may therefore be lower. Finally, 22.1% of articles with a sharing practice were classified as noise by HDBSCAN and excluded from cluster-level analyses; these articles may represent methodologically heterogeneous or interdisciplinary work that does not align with any single thematic cluster.

## Conclusion

This scoping review mapped reproducibility practices in neuroscience publications from 2015 to 2025, revealing that despite a clear upward trend, open sharing remains a minority behaviour: only 3.1% of articles reported any sharing practice, and fully integrated code-and-data sharing was observed in just 0.1% of the corpus. Adoption is markedly uneven across subfields, with computationally intensive domains leading while clinically oriented fields lag behind. Importantly, accessibility alone does not guarantee reproducibility, as fragmented materials across multiple platforms create significant barriers that current sharing metrics cannot fully capture.

These findings suggest that the next frontier for open science in neuroscience is not simply increasing the volume of shared materials but improving their integration and usability. Hosting code, data, and the article together within a single, coherent publication could make research more immediately reproducible, reduce the technical barriers required to replicate studies, and help narrow the gaps in sharing practices between subfields. Future efforts should prioritize unified platforms that allow readers to directly interact with figures, analyze underlying data, and execute code within the publication environment. Initiatives such as NeuroLibre point in this direction, enabling fully executable, interactive articles where the entire research artefact — rather than isolated components — is reproducible. It is precisely in this direction that the present thesis is oriented, aiming to move beyond fragmented sharing practices toward article-level reproducibility as a new standard for open neuroscience.
