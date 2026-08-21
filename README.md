# Polish Political YouTube — Transcript Analysis

A computational analysis of Polish political YouTube content spanning the full ideological spectrum, from left to right. The dataset covers recordings published in July 2026 across 25+ channels and includes a suite of NLP-based indicators derived from video transcripts.

---
 
## Repository structure
 
```

├── 6_fetch_transcripts.ipynb             # Fetch YouTube transcripts for videos already present in BigQuery
├── 7_translate_transcripts.ipynb         # Translating transcripts to English
├── 8_analyze_transcripts.ipynb           # Marking the levels of emotions, moral foundations, hate speech, linguistic essentialism, and group language 
├── 9_affective_polarization.ipynb        # Marking the levels of affective polarization
└── 10_moral_foundations_cluster.ipynb    # Transcripts cluster analysis based on moral foundations indices
```

## Channels

Channels were selected to represent four ideological orientations:

| Orientation | Channels |
|-------------|----------|
| Left | Nazwex, Michalina Kobla, Jan Śpiewak, Dwie Lewe Ręce, Reset Obywatelski, Partia Razem, OKO.press |
| Liberal | Dominika Wielowieyska, Eliza Michalik, TVN24, Stan Wyjątkowy |
| Centre | Fakt, Goniec, Interia, Jan Piński, Polsat News, Polskie Radio 24, Rymanowski Live, Super Express Official |
| Right | Kanał Zero, Klub Jagielloński, Patriotyczny Internet, Rafał Ziemkiewicz, Sławomir Mentzen, Telewizja Republika, Telewizja wPolsce24, Wolność i Niepodległość TV |

## Pipeline

### Translation
Polish transcripts were translated to English using GPT-4o via API (temperature = 0) to enable tools validated for English. Each transcript was translated as a single request to preserve register, idioms, and rhetorical structure.

### Emotion Classification (GoEmotions)
Model: [`SamLowe/roberta-base-go_emotions`](https://huggingface.co/SamLowe/roberta-base-go_emotions) — a RoBERTa-based multi-label classifier covering 27 emotion categories + neutral (Demszky et al., 2020).

Transcripts were segmented into non-overlapping 400-word chunks; chunk-level probability vectors were averaged to produce one 28-dimensional vector per video. Activation threshold: p ≥ 0.30.

Output: 28 numeric columns (`emotion_admiration`, `emotion_anger`, …) + `emotion_labels` (pipe-separated active labels).

### Moral Foundations (MFD 2.0)
Moral language was quantified using the Moral Foundations Dictionary 2.0 (Frimer et al., 2019), covering six foundations: Care/Harm, Fairness/Cheating, Loyalty/Betrayal, Authority/Subversion, Purity/Degradation, Liberty/Oppression. Each foundation is split into virtue and vice subcategories; scores are word-count proportions relative to total tokens.

Output: 6 aggregate columns + 12 virtue/vice columns.

### Hate Speech Detection
Two parallel approaches were used to assess cross-model and cross-language agreement:

**English:**
- Binary: [`facebook/roberta-hate-speech-dynabench-r4-target`](https://huggingface.co/facebook/roberta-hate-speech-dynabench-r4-target) → `hate_en_score`, `hate_en_binary`
- Multiclass (race, religion, gender, sexual orientation, disability, origin): [`cardiffnlp/twitter-roberta-base-hate-multiclass-latest`](https://huggingface.co/cardiffnlp/twitter-roberta-base-hate-multiclass-latest) → `hate_en_category`

**Polish:**
- Binary: [`hate-speech-CNERG/dehate-bert-multilingual`](https://huggingface.co/hate-speech-CNERG/dehate-bert-multilingual) → `hate_pl_score`, `hate_pl_binary`

Cross-variant agreement (PL vs. EN binary) is stored as a binary flag.

### Linguistic Essentialism
Noun ratio computed on original Polish text using spaCy `pl_core_news_sm` (NOUN + PROPN tags, Universal Dependencies). Based on Cichocka et al. (2016): higher noun usage is associated with more essentialist framing and right-leaning political discourse.

### Group Language (We/They)
Lexical counts of first-person plural pronouns (*my*, *nasz*, …) and second/third-person plural pronouns (*oni*, *wy*, *wasz*, …) on original Polish text, using exact-match lists to avoid false positives from stemming.

### Affective polarization
xxx

## Cluster Analysis

Cluster analysis was performed on 28-dimensional standardised emotion vectors (M = 0, SD = 1 per variable), with individual videos as units of analysis.

**Methods:**
- **K-Means** — optimal *k* selected over range 2–10 using elbow method, silhouette score, Calinski-Harabasz index, and Davies-Bouldin index (50 random restarts per *k*)
- **Hierarchical (Ward linkage)** — agglomerative, same *k* as K-Means for direct comparison; inter-method agreement assessed with Adjusted Rand Index (ARI)

**Visualisation:** UMAP (n_neighbors=15, min_dist=0.1) for 2D embedding, coloured by cluster assignment (three variants) and channel political orientation.

**Cluster interpretation:** For each cluster, mean emotion values and deltas from the global mean were computed. Dominant emotions (highest positive delta) and suppressed emotions (highest negative delta) were used to characterise each cluster substantively. Political orientation distributions within clusters were examined to assess overlap between emotional and ideological profiles.

## References

Carnaghi, A., et al. (2008). Nomina sunt omina. *JPSP, 94*(5), 839–859.

Cichocka, A., et al. (2016). On the grammar of politics. *Political Psychology, 37*(6), 799–815.

Demszky, D., et al. (2020). GoEmotions. *ACL 2020*.

Frimer, J. A., et al. (2019). Moral Foundations Dictionary 2.0. OSF: https://osf.io/xakyw

Graham, J., Haidt, J., & Nosek, B. A. (2009). Liberals and conservatives rely on different moral foundations. *JPSP, 96*(5), 1029–1046.

Honnibal, M., et al. (2020). spaCy. Zenodo.

Hubert, L., & Arabie, P. (1985). Comparing partitions. *Journal of Classification, 2*(1), 193–218.

McInnes, L., Healy, J., & Melville, J. (2018). UMAP. arXiv:1802.03426.
