# Classifying Music Production Aesthetic from Spotify Audio Features

**Language:** R  
**Key packages:** tidymodels, ranger, xgboost, SuperLearner, tidyverse, pROC  
**Dataset:** [Spotify Tracks Dataset — Kaggle](https://www.kaggle.com/datasets/priyamchoksi/spotify-dataset-114k-songs)

---

## Objective

This project developed and evaluated machine learning models to classify music 
production aesthetic as alternative or maximalist using Spotify audio features. 
The goal was to determine whether production aesthetic, the sonic character of 
a track shaped by mastering, compression, mixing, and arrangement decisions, is 
computationally detectable from audio characteristics alone, and to identify the 
features most strongly associated with the distinction. Loudness was deliberately 
excluded despite being the most direct correlate of maximalism, to avoid circular 
reasoning given that the aesthetic labels were derived from producer communities 
associated with loudness practices.

---

## Data and Methods

### Dataset
The analysis uses the Spotify Tracks Dataset, a publicly available collection of 
approximately 114,000 songs spanning 114 genres with 11 standard audio features 
per track. The dataset was sourced from Kaggle (link above).

### Aesthetic Labels
Production aesthetic labels were assigned through a three-stage computational 
pipeline. First, 60 producers were identified through systematic review of music 
criticism and industry publications, 30 associated with the alternative aesthetic 
and 30 with the maximalist aesthetic. Second, a producer co-credit network was 
constructed from Apple Music metadata. Third, Louvain community detection was 
applied to this network to identify aesthetic communities by maximizing modularity,
which finds groups of producers more densely connected to each other through shared 
credits than to the rest of the network. Each song was then assigned the aesthetic 
label of its credited producers' community. This procedure yielded a near-perfectly 
balanced dataset of approximately 57,000 alternative and 57,000 maximalist songs.

### Models
Six models were evaluated in order of increasing complexity:

1. Logistic Regression — linear baseline
2. Lasso Logistic Regression — L1 regularization for feature selection
3. Decision Tree — recursive partitioning, fully interpretable split rules
4. Random Forest — bagged ensemble of decorrelated trees
5. XGBoost — sequential gradient boosting
6. Super Learner — stacked ensemble with non-negative least squares meta-learner

### Tuning and Evaluation
All models were tuned using 5-fold cross-validation with ROC-AUC as the tuning 
criterion. The dataset was split 80/20 into training and test sets with 
stratification on the outcome. Final evaluation on the held-out test set used 
ROC-AUC, accuracy, precision, recall, F1-score, calibration plots, directional 
error analysis (alternative misclassified as maximalist versus maximalist 
misclassified as alternative), and error overlap analysis across all six models.

---

## Main Findings

- Production aesthetic is highly predictable from Spotify audio features, with 
  the best models achieving ROC-AUC near 0.96 and accuracy approaching 90%.
- Energy was the most important feature across all modeling approaches. 
  Acousticness, instrumentalness, speechiness, valence, danceability, liveness, 
  and tempo also showed consistent importance across linear and tree-based methods.
- Performance improved steadily from logistic regression through the decision tree 
  and then to the ensemble methods, demonstrating that the aesthetic boundary 
  contains substantial non-linear structure. High energy predicts maximalism more 
  strongly when acousticness is simultaneously low, a pattern no linear model can 
  fully capture.
- Random forest emerged as the strongest individual model and received the full 
  weight in the Super Learner ensemble, indicating that the other candidate 
  learners contributed little additional information beyond what random forest 
  already captured.
- All models made more alternative-to-maximalist errors than maximalist-to-
  alternative errors, suggesting a structural asymmetry in the audio feature space 
  where ambiguous songs are pulled toward the maximalist prediction regardless of 
  model complexity.
- A small set of songs remained misclassified by every model, indicating genuine 
  aesthetic ambiguity in the data rather than a limitation of any particular model. 
  These songs occupy the middle of the energy and acousticness distributions and 
  likely reflect producers who work fluidly across aesthetic traditions.

---

## Practical Implications

The results suggest that production aesthetic can be detected reliably enough to 
support applications such as playlist organization by sonic character rather than 
genre, A&R catalog analysis, and large-scale musicological research on how 
production practices have shifted over time. The strongest predictors, energy, 
acousticness, and instrumentalness, are features related to production intensity 
and texture rather than simple structural characteristics like key, mode, or time 
signature. This validates the theoretical claim that the alternative/maximalist 
distinction is fundamentally a production phenomenon detectable from audio alone.

---

## Future Directions

Future work could improve the labeling framework through more advanced community 
detection methods, incorporate more globally representative producer networks 
beyond predominantly Anglophone music traditions, and move beyond Spotify summary 
features by directly modeling audio recordings, spectrograms, or learned audio 
embeddings.

---

## Repository Structure

```
├── Final_Project.Rmd        # Full analysis notebook
├── Final_Project.html       # Knitted HTML output (readable without running R)
├── labeled_dataset.csv      # Labeled Spotify tracks dataset
│                              (if file exceeds GitHub size limit, 
│                               download from Kaggle link above and 
│                               place in the same directory as the .Rmd)
└── README.md                # This file
```

---

## How to Run

Open `Final_Project.Rmd` in RStudio and knit to PDF. All required packages are 
loaded and listed in Section 1 of the notebook. Install any missing packages with 
`install.packages()` before knitting. 

**Runtime note:** The SuperLearner fitting step in Section 6.6 is the most 
computationally intensive part of the analysis. On a standard laptop with 
parallelization across available cores, expect approximately 20-40 minutes for 
the full notebook to knit. The random forest and XGBoost tuning steps also 
benefit from multiple cores and are parallelized automatically.

---

*Any minor numerical discrepancies between section outputs reflect differences 
between intermediate interactive runs and the final knitted execution. All 
interpretations were based on the final knitted output, and overall findings, 
model rankings, and substantive conclusions are consistent throughout.*
