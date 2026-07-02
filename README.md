# Securitising the Ballot

Computational modelling of securitising rhetoric in UK party manifestos.

This repository accompanies my project, **Securitising the Ballot: Computational Modelling of Securitising Rhetoric in UK Party Manifestos**. It contains the main notebooks, processed data files, weak-label resources, model training code and temporal analysis artifacts used to investigate how UK political parties have framed security-relevant issues in manifestos after 9/11.

The final writeup is included as [`Securitsing The Ballot writeup, Joshua Bourton.pdf`](Securitsing%20The%20Ballot%20writeup,%20Joshua%20Bourton.pdf), with the LaTeX source and figures bundled in [`LaTeX writeup.zip`](LaTeX%20writeup.zip).

## Project Context

Since 9/11, national security has become increasingly central to political discourse across Western democracies. In the UK, this has developed alongside legislation in areas such as counter-terrorism, surveillance, immigration and digital regulation, much of which has been criticised for expanding state power at the expense of civil liberties.

This project examines that shift through **securitisation theory**, developed by the Copenhagen School of security studies. Securitisation theory does not treat security as a purely objective condition. Instead, it asks how political actors use discourse to frame an issue as an existential threat, assert urgency, and justify extraordinary measures that would otherwise sit outside normal democratic politics.

In this project I operationalise securitising rhetoric through three dimensions:

- **Threat**: language that constructs an issue as dangerous, existential or hostile.
- **Urgency**: language that compresses decision time and implies the need for immediate action.
- **Justification of extraordinary power**: language that legitimises exceptional authority, enforcement or restrictions.

Party manifestos are a useful corpus for this because they are deliberate, formally edited statements of party priorities. Unlike speeches or social media posts, they are less reactive to immediate events and can show how parties embed security framings into longer-term policy commitments.

## Research Questions

The project asks two linked questions:

1. To what extent do UK political party manifestos post-9/11 exhibit rhetorical convergence in securitising language across parties and electoral cycles?
2. How do shifts in party-level securitising rhetoric relate temporally to public opinion salience on security-relevant issues and to securitising legislation criticised as restrictive of civil liberties?

The first question is about whether parties become more similar in either the volume or intensity of securitising language over time. The second asks whether those rhetorical shifts precede, follow or co-evolve with public opinion and legislative change.

## What This Repository Contains

This repository is organised around the project pipeline rather than the structure of the final report:

```text
.
+-- preprocessing/
|   +-- corpus extraction and sentence-level preprocessing
|   +-- securitisation lexicon construction and SBERT expansion
|   +-- stratified sampling, gold labels and weak-label scoring
|   +-- intra-annotator agreement analysis
+-- securitisation regression models/
|   +-- baseline gold-only regression model
|   +-- full auxiliary supervision model
|   +-- threat-only auxiliary supervision ablation
+-- temporal analysis/
|   +-- corpus-level inference outputs
|   +-- party and election-level aggregation
|   +-- comparison with public opinion and legislation timelines
+-- Securitsing The Ballot writeup, Joshua Bourton.pdf
+-- LaTeX writeup.zip
+-- README.md
```

The notebooks contain more detailed markdown commentary at the point where each methodological choice is implemented. This README is intended as a high-level map for readers who are seeing the project for the first time.

## Pipeline at a Glance

1. **Corpus preparation**: Extract UK party manifesto text, repair OCR artifacts, segment into sentences and create the sentence-level manifesto corpus.
2. **Lexicon construction**: Build three theory-derived lexicons for threat, urgency and justification, with terms graded by rhetorical intensity.
3. **Weak-label generation**: Use tier-weighted lexicon matches to produce continuous pseudo-labels for the unlabelled corpus.
4. **Gold annotation**: Sample and hand-label 1,500 manifesto sentences on a 0-3 scale across the three securitisation dimensions, with an additional 250 pre-2005 sentences used for training augmentation.
5. **Model training**: Compare a gold-only baseline against two auxiliary-supervision variants using a shared DeBERTa-v3-base encoder and dimension-specific regression heads.
6. **Temporal analysis**: Run the selected model over the corpus, aggregate sentence-level predictions into party/election time series, and compare those trends against public issue salience and a curated UK legislation timeline.

The selected full auxiliary model achieved macro Spearman 0.524 and macro QWK 0.623 on the held-out set, with 5-fold cross-validation confirming generalisation at macro Spearman 0.487 +/- 0.016. The final analysis found that securitising rhetoric did not simply converge across UK parties: threat became a shared baseline among the largest parties, while urgency more clearly distinguished the governing party.

## Suggested Reading Order

For a quick understanding of the whole project, I recommend:

1. Read Sections 1 and 3 of the PDF writeup for the motivation, theory and methodology.
2. Skim this README to understand the repository structure.
3. Open the preprocessing notebooks in the order listed below.
4. Read `full_aux.ipynb` before the other model notebooks if you only want the final selected model.
5. Open `temporal_anlaysis.ipynb` last, because it depends on the model outputs and brings the project back to the research questions.

## Reproducibility Notes

The repository includes the main processed files needed to inspect the project and rerun much of the analysis. It does not include every raw or heavyweight artifact.

- Trained model weights are not included because they are approximately 800 MB each. The models can be regenerated from the provided inputs, but training takes roughly one hour per model and requires a GPU.
- Raw manifesto PDFs from the source corpus are not included. The processed sentence-level corpus is included instead.
- Some notebooks reference intermediate audit, resume or scratch outputs that are not necessary for understanding the final project state.
- The temporal analysis includes precomputed full-auxiliary corpus predictions so that the inference stage does not need to be rerun.
- The project currently does not include a pinned `requirements.txt` or `environment.yml`. Adding one would make the repository easier to rerun on a fresh machine.

## Folder 1: `preprocessing`

The preprocessing folder contains everything needed to turn the manifesto corpus into model inputs and weak-supervision resources.

The main notebooks are:

- `Extracting_and_preprocessing_manifesto_corpus.ipynb`: OCR-powered extraction of the manifesto corpus, sentence segmentation, artifact repair, deduplication and export of the sentence-level corpus.
- `Securitisation_Lexicons_Construction.ipynb`: construction of the three initial securitisation lexicons for threat, urgency and justification.
- `SBERT_Lexicon_Expansion.ipynb`: exploratory expansion of semantically similar corpus terms using SBERT.
- `stratified_corpus_sampler.ipynb`: stratified corpus sampling for gold annotation, weak-label pool construction and tier-weighted lexicon scoring for auxiliary supervision.
- `Intra_Annotator_Agreement_Analysis.ipynb`: intra-annotator reliability analysis using blind re-annotation and quadratic weighted kappa scores.

The supplementary files in this folder are:

- `manifesto_corpus.parquet`: the full sentence-level manifesto corpus produced by the extraction and preprocessing notebook.
- `manifesto_corpus_weak_labels_scored.xlsx`: the weak-label pool produced by the stratified sampler and lexicon scoring functions.
- `securitisation_lexicon.xlsx`: the final securitisation lexicon, split into threat, urgency and justification sheets, with terms categorised by signal type.
- `party_map_uk.csv`: the mapping between MARPOR party codes and UK party names, used during corpus extraction.

## Folder 2: `securitisation regression models`

This folder contains the three securitisation regression models compared in the project.

The models are:

- `baseline.ipynb`: the gold-only baseline model trained on the hand-labelled annotation set.
- `full_aux.ipynb`: the final selected auxiliary-supervision model, using lexicon-derived pseudo-labels for all three securitisation dimensions.
- `threat_only.ipynb`: a selective auxiliary-supervision ablation using only the threat pseudo-label signal.

All three models use the same broad architecture: a shared DeBERTa-v3-base encoder, mean pooling over non-padding tokens, and separate regression heads for the three securitisation dimensions. The auxiliary variants extend the baseline by adding weak-supervision heads trained on the lexicon-derived pseudo-labels from the unlabelled corpus.

The supplementary files in this folder are:

- `manifesto_corpus_annotation_batches.xlsx`: the 1,500 hand-labelled gold sentences used for training and evaluation.
- `pre2005_annotation_workbook.xlsx`: 250 manually labelled pre-2005 sentences selected to improve sparse high-intensity urgency and justification coverage in training.
- `manifesto_corpus_weak_labels_scored.xlsx`: a duplicate of the weak-label file in `preprocessing`, included here so the model notebooks can be run from this folder more easily.

The trained model weights are excluded because of their size. You can regenerate them by running the model notebooks with the provided inputs, or contact me at `xtzv61@durham.ac.uk` if you need the saved weights.

## Folder 3: `temporal analysis`

This folder contains the temporal analysis notebook and the external/contextual data used to interpret the model outputs.

The main notebook is:

- `temporal_anlaysis.ipynb`: corpus inference, party/election aggregation, visualisation, and comparison with issue salience and legislation timing.

The analysis has two main stages:

1. Running the selected full auxiliary model over the full manifesto corpus and storing sentence-level predictions.
2. Aggregating those predictions by party and election cycle, then comparing the resulting securitisation time series with Ipsos issue-salience data and a curated timeline of UK securitising legislation.

The supplementary files in this folder are:

- `corpus_sentence_predictions_full_aux.parquet`: precomputed sentence-level predictions from the selected full auxiliary model.
- `ipsos_securitisation_analysis.csv`: monthly 2005-2024 Ipsos Issues Index data for security-relevant policy domains.
- `uk_securitising_legislation_sectors.csv`: the curated UK legislation timeline used in the temporal analysis and reported in the appendix.

## Report Files

- `Securitsing The Ballot writeup, Joshua Bourton.pdf`: the final project report.
- `LaTeX writeup.zip`: the LaTeX source, bibliography and figure assets used to compile the report.

## Use of Generative AI

During development of the code files presented here, I selectively used generative AI, specifically Claude Sonnet 4.6 by Anthropic, to help with low-level implementation tasks such as regex, debugging, syntax, formatting and refactoring. I have dedicated markdown cells in the notebooks where I used it more extensively, especially in `Extracting_and_preprocessing_manifesto_corpus.ipynb`, `SBERT_Lexicon_Expansion.ipynb` and `temporal_anlaysis.ipynb`.

All substantive work is my own, including the methodological design, modelling choices, analysis logic, lexicon construction, evaluation framework, gold dataset construction and annotation.

I used AI chat-by-chat by writing targeted queries for specific implementation or formatting problems, and I tested generated integrations before keeping them. This kept AI's role limited to low-level support while the planning, building and refinement remained under my control.

I also accounted for known limitations of generative AI, including its tendency to affirm user prompts and produce confidently incorrect outputs, by keeping its role narrow and validating generated code carefully.

I did not use AI in any capactiy in the report itself.
