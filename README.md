# Topic modelling of the synthetic biology literature

A Google Colab notebook and its exported Plotly figures. The notebook builds a BERTopic topic model of 23,311 Web of Science abstracts on synthetic biology and analyses topics over time, journals, citations and countries.

It is the code for the topic modelling and bibliometric parts of this paper:

> Meier F, Dixon T, Williams T, Paulsen I. Navigating the Frontier of Synthetic Biology: An AI-Driven Analytics Platform for Exploring Research Trends and Relationships. *ACS Synthetic Biology* 12(11), 3229-3241 (2023). [doi:10.1021/acssynbio.3c00192](https://doi.org/10.1021/acssynbio.3c00192)

The paper's abstract also describes a co-authorship network analysis. That analysis is not in this repository. There is no web application here either: the interactive output is the set of exported HTML figures. The notebook dates from June 2023 and has not been updated since (see [Limitations](#limitations)).

## Contents

| Path | Description |
|---|---|
| `New_NLP_analysis_of_synthetic_biology.ipynb` | The whole analysis: 101 cells, about 60 MB with saved outputs. GitHub reports it as too big to display, so download it and open it in Colab or Jupyter. The tables below describe its code. |
| nine `.html` files | Interactive Plotly figures. GitHub does not render HTML, so download a file and open it in a browser. See [Saved outputs](#saved-outputs). |
| `docs/figures/` | Static previews of three of the HTML figures, rendered from the tracked files with headless Chrome. |

## Pipeline

```mermaid
flowchart TD
    A["Web of Science export<br/>29,862 records"] --> B["Filter and de-duplicate<br/>23,311 records with abstract,<br/>year, authors and addresses"]
    B --> C["Clean abstracts<br/>lower case, letters only"]
    C --> D["BERTopic<br/>all-mpnet-base-v2 embeddings,<br/>UMAP, HDBSCAN, c-TF-IDF<br/>99 topics in the saved output"]
    D --> E["Topic views<br/>document maps, hierarchy,<br/>similarity heatmaps"]
    D --> F["Topics over time<br/>yearly frequency, 2000 to 2021"]
    B --> G["Journals and citations<br/>treemap, trends, scatter plots"]
    B --> H["Countries<br/>addresses matched to a city list,<br/>counts and citations"]
    E --> I["Interactive Plotly figures<br/>HTML exports"]
    F --> I
    G --> I
    H --> I
    S["Scopus export and<br/>earlier Web of Science export"] -.-> J["DOI overlap check<br/>Venn diagram"]
```

Cell numbers count every cell of the notebook, code and Markdown, from 0.

| Stage | Cells | What the code does |
|---|---|---|
| Load and filter | 8-16 | Reads a Web of Science export of 29,862 records. Drops duplicate titles (29,412 left), keeps articles, reviews and proceedings papers (24,551), then keeps records that have an abstract, publication year, authors and addresses (23,311). |
| Clean abstracts | 17 | Lower-cases the text, removes URLs and words that start with "@", and keeps letters only. |
| Topic model | 34-47 | BERTopic with the settings below. The saved output has 99 topics. |
| Topic views | 48-53 | Document maps (`visualize_documents`, `visualize_hierarchical_documents`), topic hierarchy, and similarity heatmaps for all topics and for the 50 largest. |
| Topics over time | 54-64 | `topics_over_time` per publication year, 2000 to 2021. Line charts of yearly counts and of percentage share, and a bar chart of share. |
| Journals and citations | 65-77 | Papers and mean citations per journal (treemap), journals over time, the ten most cited papers, citations against publication year and against 180-day usage count. |
| Countries | 78-100 | Splits each address field at `;`, matches city and country names against the SimpleMaps World Cities list by substring, then plots a world map, a density map, papers and citations per country over time, and citation box plots. |
| Side checks | 20-32 | DOI overlap between an earlier Web of Science export and a Scopus export (9,598 DOIs in both, 6,046 only in Web of Science, 3,289 only in Scopus, with Scopus limited to records that have a PubMed ID). A word cloud of Keywords Plus. Publications per year. |

## Topic model settings

Cells 38 and 39.

| Component | Setting |
|---|---|
| Embeddings | `all-mpnet-base-v2` through sentence-transformers |
| UMAP | `n_neighbors=15`, `n_components=5`, `min_dist=0.0`, cosine metric, no random seed |
| HDBSCAN | `min_cluster_size=15`, euclidean metric, `cluster_selection_method='eom'` |
| Topic words | `CountVectorizer(ngram_range=(1, 3), stop_words="english")` and class-based TF-IDF |
| Topic number | `nr_topics=100` requested, `calculate_probabilities=True` |
| Outliers | `reduce_outliers`, then `update_topics` (cells 41 and 42) |
| Labels | Cell 38 sets a `MaximalMarginalRelevance` representation, then replaces it with an OpenAI `gpt-3.5-turbo` representation model that needs an API key. The topic names saved in cells 43 to 45 are keyword lists such as `0_metabolic_production_pathway_engineering`, not model-written labels. |

## Saved outputs

<p align="center"><img src="docs/figures/topic_hierarchy.png" width="520" alt="Dendrogram of 99 topics, coloured by cluster"></p>

Topic hierarchy from `hiearchy_color_threshold2-2.html`.

<p align="center"><img src="docs/figures/topics_over_time.png" width="640" alt="Line chart of publications per year for each topic, 2000 to 2021"></p>

Publications per year for each topic, from `dynamic_topic_modelling_absolute.html`.

<p align="center"><img src="docs/figures/journal_treemap.png" width="720" alt="Treemap of journals sized by paper count and coloured by mean citations"></p>

Journal treemap from `journal_treemap-2.html`. The previews are screenshots of the tracked HTML files. Nothing was re-plotted.

Eight of the HTML files hold exactly the figure saved in the output of the notebook cell listed here (same data and layout).

| File | Cell | Content |
|---|---|---|
| `hiearchy_color_threshold2-2.html` | 50 | Hierarchical clustering of the 99 topics |
| `heatmap_top100.html` | 52 | Similarity matrix of the 99 topics |
| `dynamic_topic_modelling_absolute.html` | 61 | Publications per year for each topic, 2000 to 2021 |
| `journal_treemap-2.html` | 70 | Treemap of 3,375 journals. Area is paper count, colour is mean citations. |
| `pie_chart_countrie.html` | 90 | Share of address records by country |
| `countries_over_time_others.html` | 97 | Address records per year for 15 countries and "other" |
| `top10countries_cit_count_desc.html` | 99 | Citation counts by country, box plots on a log axis. The file name says 10; the figure shows 15 countries and "other". |
| `all countries.html` | 100 | Citation counts by publication year, box plots on a log axis |
| `topic_vis.html` | none | "Documents and Topics" map from an earlier run: 14,854 documents in 50 topics. The current notebook does not write this file. |

Topic labels in the saved model include `0 metabolic production pathway` (2,143 abstracts), `4 circuits biological synthetic` (1,501), `6 crispr cas editing`, `13 light optogenetic control` and `40 microbial consortia communities`. Some topics are only loosely related to synthetic biology, for example `98 graphene nanoparticles silver`.

## Running the notebook

The notebook was not re-run when this README was written.

1. Use Colab with a GPU runtime (the notebook metadata records a T4). The saved pip output shows Python 3.10, pandas 1.5.3, torch 2.0.1 and plotly 5.13.1. BERTopic, sentence-transformers, UMAP and HDBSCAN were installed without version pins, and their versions are not recorded.
2. Put the input files listed under [Data](#data) in `MyDrive/Coding data/` on Google Drive.
3. Supply your own credentials in place of the placeholders: `openai.api_key` in cell 38 and the Chart Studio username and key in cell 5. Do not commit them. Delete the `py.plot(...)` lines, which upload figures to Chart Studio: its public chart pages now return HTTP 410 (checked on 20 November 2026).
4. Cell 21 replaces `file1` with the older, smaller export for the Scopus comparison, and cell 22 filters it. The saved journal outputs cover 23,237 papers, more than the 15,644 journal articles with a DOI that the older export gives (Venn diagram, cell 24), so they appear to come from the expanded query. After cell 24, reload `expanded_query.xlsx` before running the journal and country cells.

## Data

None of these files is in the repository.

| File | Use |
|---|---|
| `expanded_query.xlsx` | Web of Science export for the topic model and all later analyses (29,862 records before filtering) |
| `18k_articles.xlsx` | Earlier Web of Science export, used for the Scopus comparison |
| `scopus.csv` | Scopus export, used for the Scopus comparison |
| `worldcities.csv` | [SimpleMaps World Cities](https://simplemaps.com/data/world-cities), used to place addresses |

The search strings for the two exports are not recorded here. Text from the exports does appear in some saved outputs: cleaned abstracts in the hover labels of the document map (cell 48 and `topic_vis.html`) and article titles in the hover labels of cells 76, 77 and 85.

## Limitations

- **Age.** The notebook was last committed on 17 June 2023 and has not been updated since. The saved yearly series stop in 2021 (topics over time) and 2022 (publications per year), and the country plots include a partial 2023.
- **Reproducibility.** One notebook, no environment file, no pinned versions, and inputs that are not included. It relies on Colab, Google Drive paths, an OpenAI key and Chart Studio. UMAP has no random seed, so a new run can give different topics. The notebook does not measure topic coherence or stability.
- **Saved outputs.** They come from more than one Colab session. Cell 56 stores a `NameError`, and cell 57 reloads CSV files written earlier. The trained model (cell 46) and the topic tables (cells 47 and 56) are not in the repository.
- **Corpus.** Some of the 99 topics are only loosely related to synthetic biology.
- **Countries.** Each address counts once, so the country figures are address counts, and a paper with authors in several countries is counted several times. City and country names are found by substring matching, so mismatches are possible.
- **Scope.** No co-authorship network analysis (see above).

## History

An earlier version of the notebook (March 2023, 14,854 abstracts) was deleted from this repository in June 2023 and remains in the Git history. A copy of it is also in [Navigating-the-frontier-of-Synthetic-Biology-An-AI-driven-platform-for-exploring-research-trends-an](https://github.com/SynBioExplorer/Navigating-the-frontier-of-Synthetic-Biology-An-AI-driven-platform-for-exploring-research-trends-an), which has no other code and was last pushed in March 2023.
