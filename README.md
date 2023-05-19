# Bias in De-identification

<a href="">In the Name of Fairness: Assessing the Bias in Clinical Record De-identification</a> (FAccT 2023).

In this project, we investigate the bias of de-identification systems on names in clinical notes via a large-scale empirical analysis. <br >
We create 16 name sets that vary along four demographic dimensions: _gender_, _race_, _name popularity_, and _the decade of popularity_. <br >
We insert these names into 100 manually curated clinical templates and evaluate the performance of nine public and private de-identification methods: ```spaCy```, ```Stanza```, ```flair```, ```Amazon```, ```Microsoft```, ```Google```, ```NeuroNER```, ```Philter```, and ```MIST```.

## Data

Our data can be downloaded from <a href="">PhysioNet</a>.
- **General**: the data used for our general analysis.
    - **Input/names-first.csv** and **Input/names-last.csv** store the first and last names in the 16 name sets we created. Please refer to Section 3.2 in our paper and **Preparation/name.ipynb** in this repository for the full detail.
    - **Input/notes-base.csv** stores the 100 manually curated clinical templates where we marked the occurrence of names and other categories of protected health information (PHI).
    - **Input/notes-input.jsonl** and **Input/notes-label.jsonl** store the 16,000 notes and the corresponding labels, respectively, for evaluating the performance on de-identifying names. Please refer to Section 3.4 in our paper and **Preparation/note.ipynb** in this repository for the full detail.
    - **Output/notes-{method}.jsonl** stores the prediction output of each of the nine evaluated de-identification baseline methods.
- **Polysemy**: the data used for assessing how polysemy names affect model performance.
    - **Input/polysemies-input.jsonl** and **Input/polysemies-label.jsonl** keep the notes and the corresponding labels, respectively, for evaluating the performance on de-identifying polysemy names. Please refer to Section 5.1 in our paper and **Preparation/polysemy.ipynb** in this repository for the full detail.
    - **Output/polysemies-{method}.jsonl** keeps the prediction output of each of the nine evaluated de-identification baseline methods on polysemy names.
- **Finetune**: the data used for fine-tuning ```spaCy``` and ```NeuroNER```. Here, we consider two types of context (i.e., _general_ and _clinical_) and two types of names (i.e., _popular_ and _diverse_). Please refer to Section 6.1 in our paper and **Preparation/finetune.ipynb** in this repository for the full detail.
    - **Input/context-general.jsonl** and **Input/context-clinical.jsonl** contain the two types of context.
    - **Input/names-popular.jsonl** and **Input/names-diverse.jsonl** contain the two types of names. **Input/names-test.jsonl** contains the names used for populating the test notes.
    - **Input/inputs-{context}+{name}.jsonl** and **Input/labels-{context}+{name}.jsonl** contain the notes and the corresponding labels, respectively, for fine-tuning.
    - **Input/inputs-test.jsonl** and **Input/labels-test.jsonl** contain the notes and the corresponding labels, respectively, for evaluating fine-tuned methods.
    - **Output/finetunes-{context}+{name}-{method}-{seed}.jsonl** contains the prediction output of each of the two fine-tuned methods.

## Service

In this folder, we examine each of the nine de-identification methods in the corresponding notebook. <br >
We describe below the steps to set up the evaluation environment necessary for each method.
- <a href="https://spacy.io/">``spaCy``</a>
    - ``pip install -U 'spacy[cuda-autodetect]'``
    - ``python -m spacy download en_core_web_trf``
- <a href="https://stanfordnlp.github.io/stanza/">``Stanza``</a>
    - ``pip install stanza``
- <a href="https://github.com/flairNLP/flair">``flair``</a>
    - ``pip install flair``
- <a href="https://docs.aws.amazon.com/comprehend-medical/latest/dev/textanalysis-phi.html">``Amazon``</a>
    - Sign up for AWS and create an IAM user account
    - Set up the AWS Command Line Interface (CLI) and add a named profile in the AWS CLI config file
    - ``pip install boto3``
- <a href="https://learn.microsoft.com/en-us/azure/cognitive-services/language-service/personally-identifiable-information/overview">``Microsoft``</a>
    - Sign up for Microsoft Azure
    - Create a language resource and keep the key and endpoint
    - ``pip install azure``
- <a href="https://cloud.google.com/dlp/docs/deidentify-sensitive-data">``Google``</a>
    - Sign up for Google Cloud and create a project with billing and Data Loss Prevention (DLP) API enabled
    - Set up a service account as DLP user and get credential
    - Install Google Cloud CLI
- <a href="https://github.com/Franck-Dernoncourt/NeuroNER/tree/master">``NeuroNER``</a>
    - Create and activate a new conda environment: ``conda create -n neuroner python=3.7 tensorflow-gpu=1.15 spacy=2.3 && conda activate neuroner``
    - Install the gpu version of NeuroNER: ``pip install pyneuroner[gpu]``
    - Download the English language module for Spacy: ``python -m spacy download en``
    - Download word embeddings: ``wget -P External/NeuroNER/Data/Embedding http://neuroner.com/data/word_vectors/glove.6B.100d.zip && unzip External/NeuroNER/Data/Embedding/glove.6B.100d.zip -d External/NeuroNER/Data/Embedding/``
    - Move and rename the fetched model to **External/NeuroNER/Model/original+original**
    - Create the input folder **External/NeuroNER/Data/General/deploy/**
    - Clean up all the intermediate outputs of NeuroNER during its prediction
- <a href="https://www.nature.com/articles/s41746-020-0258-y">``Philter``</a>
    - Download the source code to **External/Philter** by ``git clone https://github.com/BCHSI/philter-ucsf.git``
    - Install the required packages: chardet, nltk, xmltodict
- <a href="https://mist-deid.sourceforge.net/">``MIST``</a>
    - Download and unzip <a href="https://sourceforge.net/projects/mist-deid/files/">MIST 2.0.4</a> to **External/MIST/Model**
    - Install MIST by running ``bash External/MIST/Model/install.sh`` (you may need Java and Python 2)
    - Download <a href="https://portal.dbmi.hms.harvard.edu/projects/n2c2-nlp/">2006 i2b2 De-identification Training Set</a> to **External/MIST/Data** (you may need an account with DBMI Data Portal)

## Analysis

In this folder, we analyze the prediction results of the examined de-identification methods and plot the figures in our paper. <br >
Below is a snippet of these methods' overall performance (higher is better) and bias along dimensions (measured by recall equality difference, lower is better).

| Method | Precision | Recall | F1 | Gender | Race | Popularity | Decade |
|---|---|---|---|---|---|---|---|
| ``spaCy`` | 0.917 ± 0.001 | 0.629 ± 0.001 | 0.746 ± 0.001 | 0.002 ± 0.001 | 0.013 ± 0.002 | 0.028 ± 0.002 | 0.007 ± 0.002 |
| ``Stanza`` | 0.678 ± 0.001 | 0.881 ± 0.001 | 0.766 ± 0.001 | 0.002 ± 0.001 | 0.016 ± 0.002 | 0.011 ± 0.001 | 0.005 ± 0.001 |
| ``flair`` | 0.920 ± 0.001 | 0.974 ± 0.000 | 0.946 ± 0.000 | 0.003 ± 0.000 | 0.006 ± 0.001 | 0.008 ± 0.001 | 0.002 ± 0.000 |
| ``Amazon`` | 0.923 ± 0.001 | 0.925 ± 0.001 | 0.924 ± 0.001 | 0.005 ± 0.001 | 0.022 ± 0.001 | 0.032 ± 0.001 | 0.001 ± 0.001 |
| ``Microsoft`` | 0.664 ± 0.001 | 0.960 ± 0.001 | 0.785 ± 0.001 | 0.003 ± 0.001 | 0.023 ± 0.001 | 0.010 ± 0.001 | 0.006 ± 0.001 |
| ``Google`` | 0.609 ± 0.001 | 0.869 ± 0.001 | 0.716 ± 0.001 | 0.009 ± 0.001 | 0.025 ± 0.001 | 0.014 ± 0.002 | 0.010 ± 0.001 |
| ``NeuroNER`` | 0.946 ± 0.001 | 0.944 ± 0.001 | 0.945 ± 0.000 | 0.001 ± 0.001 | 0.045 ± 0.001 | 0.026 ± 0.001 | 0.002 ± 0.001 |
| ``Philter`` | 0.227 ± 0.001 | 0.794 ± 0.001 | 0.353 ± 0.001 | 0.000 ± 0.001 | 0.000 ± 0.001 | 0.003 ± 0.002 | 0.000 ± 0.001 |
| ``MIST`` | 0.474 ± 0.001 | 0.751 ± 0.001 | 0.581 ± 0.001 | 0.013 ± 0.001 | 0.022 ± 0.002 | 0.017 ± 0.002 | 0.003 ± 0.002 |


## Citation

```
@inproceedings{xiao2023in,
  title={In the Name of Fairness: Assessing the Bias in Clinical Record De-identification},
  author={Xiao, Yuxin and Lim, Shulammite and Pollard, Tom Joseph and Ghassemi, Marzyeh},
  booktitle={FAccT},
  year={2023}
}

@article{lim2023annotated,
  title={Annotated MIMIC-IV discharge summaries for a study on deidentification of names (version 1.0)},
  author={Lim, Shulammite and Johnson, Alistair and Xiao, Yuxin and Moukheiber, Dana and Moukheiber, Lama and Moukheiber, Mira and Ghassemi, Marzyeh and Pollard, Tom},
  journal={PhysioNet},
  year={2023},
  note={"https://doi.org/10.13026/ngc0-0f54"}
}
```