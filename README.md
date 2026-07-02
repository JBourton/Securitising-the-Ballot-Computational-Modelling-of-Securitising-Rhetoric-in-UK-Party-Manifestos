This folder contains all the main code files used in my securitisation regression model pipeline. I explain them all on a high level in this README, but you can find more details about what they do on lower-levels in the notebooks themselves, as I've added plenty of md cells throughout each one to provide context.

I've split this code submission up into 3 subfolders, namely the preprocessing, the models themselves and the temporal analysis (explained in more detail below). I've included the juypter notebooks but also the additional files I use in my analysis (like my custom legislation timeline for the temporal analysis) or the files that I create from these notebooks (like excels & parquets etc) where relevant.

### Subfolder 1: preprocessing
The preprocessing is everything I needed to do to get the model inputs in the right form for training and inference. These are:
- OCR-powered extraction of the manifesto corpus in `Extracting_and_preprocessing_manifesto_corpus.ipynb`
- Stratified corpus sampling to create the gold label batches for human annotation, alongside weak label pool construction and tier-weighted lexicon scoring for the auxiliary supervision., in `stratified_corpus_sampler.ipynb`
- Creation of the 3 lexicons I use for weak labelling in `Securitisation_Lexicons_Construction.ipynb`
- Exploratory expansion of semantically similar corpus terms using SBERT in `SBERT_Lexicon_Expansion.ipynb`
- Intra-annotator reliability assessed via blind re-annotation of a 150-sentence subset, using QWK scores to indicate substantial agreement across all three dimensions. Done in `Intra_Annotator_Agreement_Analysis.ipynb`

The supplementary files I include in this folder are:
- `manifesto_corpus.parquet`: the full ~45,500 sentence strong manifesto corpus produced by `Extracting_and_preprocessing_manifesto_corpora.ipynb`.
- `manifesto_corpus_weak_labels_scored.xlsx`, which is the remaining ~44,000 sentences from the corpus the tier-weighted lexicon scoring functions, produced by `stratified_corpus_sampler.ipynb`.
- `securitsiation_lexicon`: the final lexicon produced by `Securitisation Lexicons Construction.ipynb` after going through SBERT-based expansion in `SBERT Lexicon Expansion.ipynb`. It's categorised into 3 sheets, 1 per each securitisation dimension, with dimension intensity labels added in the final column.
- `party_map_uk.csv`, which holds the mapping between MARPOR party codes and their party names. Not substantive, but it's needed to run the `Extracting_and_preprocessing_manifesto_corpus.ipynb` file.

### Subfolder 2: securitisation regression models
This folder contains the three securitisation regression models I compare against each other. Full details are in my paper, but the auxiliary supervised models extend the baseline's capabilities by using lexicon-generated ~44,000 psuedo-labels generated from the unlabelled corpus to supplement the 1500 hand-labelled gold-sentences. The full auxiliary variant applies that for all three securitisation dimensions and I selected it as the final reported model using results from 5-fold cross-validation (see my paper for more details). I'm keeping the selective (threat-only) variant as an ablation to show how I isolated the contribution of dimension-selective supervision. 

The models themselves are named `baseline`, `full_aux` and `threat_only`.

I've chosen to exclude the model weights because they're ~800MB each. You can generate them by running the models with the provided inputs, though they do take a while to train (~1hr each) and need GPU. Alternatively you can reach out to me on xtzv61@durham.ac.uk to ask for them.

The supplementary files I include for reference (which are needed in the files themselves if you choose to run them) are:
- `manifesto_corpus_annotation_batches.xlsx`, which holds the 1500 sentences I hand-labelled for training the models.
- `pre2005_annotation_workbook.xlsx`, containing the hand-annotated versions of the top 150 urgency sentences and top 100 justification sentences produced by running a version of the model over the 1987-2001 manifesto corpora (3 elections) to supplement the lack of high-intensity urgency and justification labels.
- `manifesto_corpus_weak_labels_scored.xlsx`: a duplicate of what's in subfolder 1 but needed to run the model. so I put it here too for simplicity's sake.

### Subfolder 3: temporal analysis
This contains the full code for my temporal analysis in the file `temporal_analysis.ipynb`

It consists of 2 independent functions that I combined into one file for simplicity's sake, namely:
- Inference (running my chosen full auxiliary supervision model over the full corpus and storing the result).
- Analysis of the inference results, which I combine with supplementary Ipsos "Most prevalent issue in the UK" data (explained in the report, and that file) and with my own created timeline of securitising legislation brought into law in the UK (included in the appendix as a formatted table, and also in this folder as an excel).

The supplementary included in subfolder are:
- `ipsos_securitisation_analysis.csv`: the monthly 2005–2024 [Ipsos most prevalent issues index](https://www.ipsos.com/en-uk/topic/issues-index) tracking five policy domains aligned with securitisation theory (defence, migration, economic, governance, health)
- `corpus_sentence_predictions_full_aux.parquet`: the results of inference that I already pre-computed (so you don't need to re-run inference, which needs both GPU & time).
- `uk_securitising_legislation_sectors.csv`: the legislation timeline I made for this analysis, which I use in the temporal analysis and show in my appendix.


## Use of Generative AI
During development of the code files I present here I selectively used generative AI (specifically, Claude Sonnet 4.6 by Anthropic) to help with low-level implementation (including regex and general debugging) and syntax/formatting tasks (including code refactoring for compactness, formatting visuals for my report). I have dedicated md cells in notebooks where I used these more extensively (namely `Extracting_and_preprocessing_manifesto_corpus.ipynb`, `SBERT_Lexicon_Expansion.ipynb` and `temporal_analysis.ipynb`), but have also made lighter isolated use of it for low-level tasks in other places in the codebase, which I intend this disclaimer to cover.

However, all of the substantive work is entirely my own, including the design of the various methodologies in each file, all of the logic behind the analysis, the model architecture and training procedure, the constrution of the lexicons, the evaluation framework and the construction and annotation of the gold dataset.

For each use of it, I used chat-by-chat messages by writing a targeted query to help me fix/brainstorm a specific implementation or bit of formatting and I always tested these integrations before keeping them. Through doing this, I kept the AI's contribution to just helping me implement low-level logic, which kept all the planning, building and refining tightly under my own control.

I made sure to bear in mind the known limitations of generative AI that the department has highlighted to me before, including its tendency to confirm its user's prompts to avoid challenging it and to produce confidently incorrect outputs, and also that it can be quite weak on edge cases. I was able to mitigate all of those by only keeping its role to the limited low-level logic I mentioned and carefully validating what it generated.

I did not use AI in my report in any capacity.