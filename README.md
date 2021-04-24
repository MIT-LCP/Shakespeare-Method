# Shakespeare-Method [![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.4711819.svg)](https://doi.org/10.5281/zenodo.4711819)
### Text mining electronic health records

The Shakespeare-Method repository contains the code we used (in a user friendly tutorial style) to develop a new method to identify attributed and unattributed potential adverse events using the unstructured notes portion of electronic health records. 

We chose the case of transfusion adverse events  and potential transfusion adverse events because new transfusion adverse events types were becoming recognized during the study data period; therefore, we anticipated an opportunity to find unattributed potential transfusion adverse events in the notes. We used MIMIC-III (EHRs for adult critical care admissions at a major teaching hospital, 2001-2012) dataset and formed a Transfused group (21,443 admissions treated with packed red blood cells, platelets, or plasma), excluded 2,373 ambiguous admissions (grey), and formed a Comparison group of 25,468 admissions. We concatenated the text notes for each admission, sorted by date, into one document, and deleted replicate sentences and lists. We  identified statistically significant words in Transfused vs. Comparison. Then, Transfused documents were vectorized on only those words, followed by topic modeling on these Transfused documents to produce 45 topics. After assigning the maximum topic to each document, we separated the documents (admissions) that had a low maximum topic score for further review by subject matter experts to evaluate for potential adverse events.

**The Shakespeare Method has five steps:**
 
+ Step 1. Convert each document into a vector of n-gram (term) frequencies.
+ Step 2. Create two groups of vectors: target and comparison.
+ Step 3. Extract terms in the target group that are significant for the target group.
+ Step 4. Apply topic analysis to the target group filtered vectors.
+ Step 5. Review the original documents that have topic scores of interest to interpret the topics and find potential adverse events.

Step 2 is described in detail in Bright RA, Bright-Ponte, SJ, Palmer LA, Rankin, SK, Blok S. Use of Diagnosis Codes to Find Blood Transfusion Adverse Events in Electronic Health Records. medRxiv 2020.12.30.20218610.  https://doi.org/10.1101/2020.12.30.20218610.

Steps 1, 3, and 4 are described in detail in this repository. An flowchart of this process can be seen below

![Shakespeare Method process with truncated examples](https://github.com/MIT-LCP/Shakespeare-Method/blob/main/SM_overview_pic.png)

# Citations
To acknowledge use of the code, please cite the DOI provided via Zenodo:

Summer K. Rankin, Roselie A. Bright, Katherine Dowdy. (2021, April 22). MIT-LCP/Shakespeare-Method: Official release 2 (Version v0.2). Zenodo. http://doi.org/10.5281/zenodo.4711819
```
@software{summer_k_rankin_2021_4711819,
  author       = {Summer K. Rankin and Roselie A. Bright and Kate Dowdy},
  title        = {MIT-LCP/Shakespeare-Method: Official release 2},
  month        = apr,
  year         = 2021,
  publisher    = {Zenodo},
  version      = {v0.2},
  doi          = {10.5281/zenodo.4711819},
  url          = {https://doi.org/10.5281/zenodo.4711819}
}
```

and the papers describing the Shakespeare Method

Bright RA, Dowdy K, Rankin SK, Blok S, Palmer LA, Bright-Ponte SJ. New and Increasing Rates of Adverse Events Can be Found in Unstructured Text in Electronic Health Records using the Shakespeare Method. medRxiv 2021.01.12.21249674. https://doi.org/10.1101/2021.01.12.21249674.
 
Bright RA, Rankin SK, Dowdy K, Blok S, Bright-Ponte, SJ, Palmer LA. Potential Blood Transfusion Adverse Events Can be Found in Unstructured Text in Electronic Health Records using the ‘Shakespeare Method’. medRxiv 2021.01.05.21249239. https://doi.org/10.1101/2021.01.05.21249239.
 
Bright RA, Bright-Ponte, SJ, Palmer LA, Rankin, SK, Blok S. Use of Diagnosis Codes to Find Blood Transfusion Adverse Events in Electronic Health Records. medRxiv 2020.12.30.20218610.  https://doi.org/10.1101/2020.12.30.20218610.


## Structure of repository
x.y_description
+ *x*=major section of analysis
+ *y*=order within section

  + 1.y = labeling, preprocessing, cleaning, tokenization, vectorization
  + 2.y = feature selection
  + 3.y = LDA topic modeling
  + 4.y = send to elasticsearch

# Data
This project uses the MIMIC III Database. The MIMIC-III database was ingested into in a PostgreSQL (version 10.11 downloaded January 12,2020) database using PostgreSQL scripts from (https://github.com/MIT-LCP/mimic-code/tree/master/buildmimic/postgres, accessed January 12, 2020.


# Connecting python to PostgreSQL
The packages we used to connect python to posgreSQL are called `psycopg2` and `sqlalchemy`. You will need to install PostgreSQL and build the MIMIC Database and put the passwords and usernames into the environment variables or directily into the notebook in order to connect to the data. This is typically the first thing in each notebook under package imports.  

Directly putting in variables looks like the following:
```
    conn = psycopg2.connect("dbname=*the_name_of_your_mimic_postgres_database* \
           user=*the_postgres_username_you_set* \
           password=*the_password_you_set_for_postgres_database* \
           options=--search_path=*the _schema_name_in_postgres_for_mimic_database*");

    engine = create_engine('postgresql://username_for_postgres:password_for_posgres@localhost/*mimic_db*')
```
To see an example of this, let's say I have created a mimic database in my postgreSQL that is called `study_mimic`, and the username for my PostgreSQL is `cool_data_scientist` and the password for postgreSQL is `data_scientist_password`. When you build the mimic database you will give it a schema. You can use anything, but it is common for people to call it `mimiciii`
So, to connect to this fictional instance of the data we would use these variables like so:

```
import psycopg2
from sqlalchemy import create_engine, update
conn = psycopg2.connect("dbname=study_mimic user=cool_data_scientist \
                         password=data_scientist_password options=--search_path=mimiciii");
engine = create_engine('postgresql://cool_data_scientist:data_scientist_password@localhost/study_mimic')
cur = conn.cursor();
cur.execute("""SET search_path = mimiciii;""")
```

If you do not set a password for your local instance of postgres, you can remove that variable.
If your postgres is not running locally, you replace `localhost` with the url.

The alternative to putting passwords directly into your code (not the most secure practice) is to create environment variables in your new conda environments (see section below for this) and access those. If I create environment variables for the `connect` and `engine` parts, they can be accessed using os.environ.get.

```
import psycopg2
from sqlalchemy import create_engine


POSTGRES_CONNECT = os.environ.get("POSTGRES_CONNECT")
POSTGRES_ENGINE = os.environ.get("POSTGRES_ENGINE")

conn = psycopg2.connect(POSTGRES_CONNECT)

cur = conn.cursor();
cur.execute("""SET search_path = mimiciii;""")

engine = create_engine(POSTGRES_ENGINE)
```
# Environment
Anaconda environments were used to manage python packages and versions. We used both python2 and python3 environments that can be created by using the [shakesPy2_environment.yml](shakesPy2_environment.yml) for python 2

In a terminal window, after you have installed Anaconda, go into the folder where this repository exists and type the following. 
```
cd Shakespeare-Method
conda env create -f shakesPy2_environment.yml
conda activate shakesPy2_environment
```

and use [env_py3.yml](env_py3.yml) to create a python 3 environment (bloatectomy requires python 3.7 and above).

```
cd Shakespeare-Method
conda env create -f env_py3.yml
conda activate env_py3
```
# 1.0 Initial Cohort Creation
[1.0_create_groups.ipynb](1.0_create_groups.ipynb)

+ Python2 environment
+ 
This notebook is the first of 2 steps to create the groups of adult admissions (>=16 years old), and the corresponding input events. 
![flowchart_of_cohort_creation](./cohort.png)

**Input**
```
patients
admissions
inputevents_mv
inputevents_cv
chartevents
```
**Output**
```
patients_adult
inputevents_mv_adult
inputevents_cv_adult
chartevents_adult
inputs_all

```

# 1.1 Cohort Selection and Concatenate Notes
[1.1_create_groups_concat_notes.ipynb](1.1_create_groups_concat_notes.ipynb)

+ Python2 environment

Label admissions and concatenate all notes for a single admission into one note (in chronological order).
Save notes in new tables `transfused_notes_sink` and `ctrl_notes_sink`

+ Uses ICD-9 codes to create control, transfused, and grey (excluded) groups.
+ chart input items from the [20180717D_ITEMS_related_to_blood_full.csv](20180717D_ITEMS_related_to_blood_full.csv) to create a labeled dictionary of input items that belong to T=transfused, N=non-transfused, or G=grey(exclusion group).  

+ Transfused Group =  the union of admissions that have **ever** had a Transfused ICD-9 code **or** a Transfused input item.
+ Control (Non-transfused) = union of admissions that meet all 4 of the following conditions:
    1. No transfused ICD-9
    2. No grey ICD-9
    3. No transfused input item
    4. No grey input item

+ Retrieves all the notes from these admissions, puts them in chronological order, and concatenates them into one large 'document' per admission

**Input:** Postgres tables
```
procedures_icd
inputs_all
20180717D_ITEMS_related_to_blood_full.csv
D_items
noteevents
```
**Output:** Postgres tables
```
transfusion_icd9
grey_icd9
ctrl_icd9
transfusion_items_dict
D_items_labeled
inputs_all_labeled
transfused_hadm_id
grey_hadm_id
ctrl_ids
transfused_notes
ctrl_notes
transfused_notes_sink
ctrl_notes_sink
```
# 1.2 Remove Notebloat
[1.2_duplicate_removal.ipynb](1.2_duplicate_removal.ipynb)

+ uses python3 environment b/c bloatectomy needs *python >= 3.7*

Uses modified bloatectomy code to remove duplicate sections of text within an admission's concatenated notes. 
For details about how the package works and our reasons for developing it, read the paper here https://github.com/MIT-LCP/bloatectomy/blob/master/bloatectomy_paper.pdf

To acknowledge use of the Bloatectomy software, please cite the DOI provided via Zenodo:

Summer K. Rankin, Roselie Bright, & Katherine Dowdy. (2020, June 26). Bloatectomy (Version v0.0.12). Zenodo. http://doi.org/10.5281/zenodo.3909030


**Input**: postgres tables
```
transfused_notes_sink
ctrl_notes_sink
```
**Output**: postgres tables
```
transfused_notes_unique 
ctrl_notes_unique 
```

# 1.3 Vectorization of Text
[1.3_vectorization.ipynb](1.3_vectorization.ipynb)

+ Python 2 environment (recommended in an AWS instance)

We used the collocation detection skip-gram method for extracting the n-grams with n = 1-5 consecutive words. We vectorized each document using a bag of words representation where each dimension is represented by the frequency (count) of each n-gram, resulting in a set of 7,422,044 words.

This notebook is used to tokenize, get collocations (ngrams), count vectorize the transfused and comparison concatenated notes.  This one has to be run on something with a large amount of ram (like 109Gb or so). Use the 'large' AWS instance. Alternatively, one could truncate the number of features (terms) to run this on a laptop or smaller instance.

We used an AWS (Amazon Web Services) EC2 memory-optimized instance (r4.8xlarge, 244GB memory, AMD64, Windows 10). The MIMIC database and POSTGRESQL must be accessible and the code in 1.0, 1.1, 1.2, should be run first to create the groups and deduplicate notes. The time to run the code was approximately 7.5 hours to create the groups, concatenate the notes into documents, and remove duplicate sections of text. We recommend only using an expensive/large instance for this step. All other steps can be done on a laptop with ~16 GB RAM.  

+ Saves as sparse matricies in pickle format (document-term matrix is broken up into 10 sections to make transfer back to a local computer or cheaper instance faster)
+ We recommend a transfer of the results (pickle files) to local computer or less expensive instance for further processing.

**Input**
```
transfused_notes_unique (postgres)
ctrl_notes_unique (postgres)
```
**Output:**
```
textfeatures_mat1
...
textfeatures_mat10 (data)
textfeatures_vocab (features/terms)
textfeatures_id (hadm_ids)
textfeatures_source (transfused/non-transfused)
```
# Section 2: Feature Selection
Our goal for feature selection was to filter document vectors to only include terms that were significant to the transfused group and then model the topics within those terms in the transfused group to identify experiences emblematic of transfusion. We formalized the process of extracting these terms by looking at term coefficients associated with a classifier that learns to differentiate the two groups. We underwent an iterative process of trying multiple hyperparameters and classification models to identify these terms. We observed that an ensemble of two classification methods (Naive Bayes and logistic regression) and filtering was useful for capturing common, infrequent, and rare terms that were significant for the Transfused group.  This term selection resulted in in 41,664 terms. 

Features (terms) are selected using 2 methods:
1. Embedded methods: supervised learning and statistical models
2. Filtering

![Flowchart of the feature selection methods](https://github.com/MIT-LCP/Shakespeare-Method/blob/main/shakespeare_flowchart_publication.png | width=100)


# 2.0 Feature Selection via Classification Models
[2.0_classification_models.ipynb](2.0_classification_models.ipynb)

+ python 3 environment

Runs multiple classification models on the 2 groups (transfused and non-transfused/control) to select the features most associated with the transfused group and save them for further analysis.

![flowchart of the supervised feature selection process](https://github.com/MIT-LCP/Shakespeare-Method/blob/main/supervised_flow.jpg)

+ test/train split
+ naive bayes classification
+ logistic regression classification
+ other classification models
+ plot confusion matrix and roc auc for multiple models
+ save model, and metrics + vocab
+ saves top (transfusion group) 5,000 terms from naive bayes (log probability) and logistic regression (coef) models for topic modeling and/or further review (pickle and csv) 

**Input**
```
textfeatures_mat1
...
textfeatures_mat10 (data)
textfeatures_vocab (features/terms)
textfeatures_id (hadm_ids)
textfeatures_source (transfused/non-transfused)
```
**Output**
```
top_logit_coef_5000.csv
logits_top_5000_matrix.pickle
NB_top_5000_feat_[date].csv
NB_terms_ratio_all.pkl
NB_top_5000_matrix.pickle
```

# 2.1 Remove Transfusion Terms From Naive Bayes Features
[2.1_nb_remove_xf_term_collapse_ngrams.ipynb](2.1_nb_remove_xf_term_collapse_ngrams.ipynb)

+ python 3 environment

After sorting the n-grams according to the feature log probability, we removed all terms making direct and exclusive reference to the act of transfusion, because they both merely repeat the criteria for membership in the transfused group and obscure terms from other notes that might be indications for or outcomes of transfusion; as would be expected, these terms were highly associated with being in the transfused group. These transfusion terms were identified for removal by manual inspection (necessary because of the variety of abbreviations and spellings) informed by the contexts of these terms in multiple notes. Each time these transfusion-related terms were removed, they were added to a dictionary of transfusion-related terms. In the end this list grew to contain almost 1000 different terms [terms_indicate_transfusion9.xlsx](terms_indicate_transfusion9.xlsx)

The team observed that many of the top n-grams had significant word overlap and the same feature vector. We decided to collapse features into the longest possible n-gram where the features had the same feature vector (i.e., they all occurred in the same documents). To accomplish this, we selected all duplicate feature vectors and used fuzzy string matching to determine whether the corresponding features overlapped. We combined overlapping features to create the longest possible n-gram and dropped the smaller n-grams. For example, the following three n-grams had the same pattern of occurrence: 
```
comments heparin induced
in comments heparin induced
comments heparin induced thrombocytopenia
```
Our script collapses these three n-grams into one long n-gram.
```
in comments heparin induced thrombocytopenia
```

+ Remove terms from the Naive Bayes vocabulary list that are clearly related to transfusion via [terms_indicate_transfusion9.xlsx](terms_indicate_transfusion9.xlsx)
+ Collapse duplicate ngrams into longest n-gram
+ Analyze frequency of terms
+ Save for next analysis as `NB_5000_final.pkl`
+ Plot basic visuals
+ Save the terms, log probability ratio, and frequency count  as `NB_top_xxxx_terms_only_dist.csv`
+ Save terms and hadm_ids with hadm_ids

**Input**
```
NB_top_5000_matrix.pickle
transfused_notes_unique (postgres table)
terms_indicate_transfusion9.xlsx
textfeatures_id.pickle
```
**Output**
```
NB_5000_final.csv
NB_5000_final.pkl
NB_top_xxxx_terms_only_dist.csv
NB_top_xxxx_hadmids_forSME.csv
```
# 2.2 Remove Transfusion Terms From Logistic Regression Results
[2.2_lr_remove_xf_term_collapse_ngrams.ipynb](2.2_lr_remove_xf_term_collapse_ngrams.ipynb)

+ python 3 environment

After sorting the n-grams according to the feature log probability, we removed all terms making direct and exclusive reference to the act of transfusion, because they both merely repeat the criteria for membership in the transfused group and obscure terms from other notes that might be indications for or outcomes of transfusion; as would be expected, these terms were highly associated with being in the transfused group. These transfusion terms were identified for removal by manual inspection (necessary because of the variety of abbreviations and spellings) informed by the contexts of these terms in multiple notes. Each time these transfusion-related terms were removed, they were added to a dictionary of transfusion-related terms. In the end this list grew to contain almost 1000 different terms [terms_indicate_transfusion9.xlsx](terms_indicate_transfusion9.xlsx)

For the logistic regression terms, none were overlapping and this step can be removed but the code has been left in the script.


+ Remove terms that are clearly related to transfusion
+ Collapse duplicate ngrams into longest n-gram
+ Analyze frequency of terms
+ Plot basic visuals
+ Save for next analysis as `LR_5000_final.pkl`
+ Save the terms, coef, and frequency count for SMEs

**Input**
```
logits_top_5000_matrix.pickle
transfused_notes_unique (postgres table)
terms_indicate_transfusion9.xlsx
textfeatures_id.pickle
```
**Output**
```
LR_5000_final.pkl
LR_top_5000_terms_only.csv
```
# 2.3 Feature Selection Via Classification and Statistical Models
[2.3_classification_vocabs.ipynb](2.3_classification_vocabs.ipynb)
+ python 3 environment

Run new classification on full dataset (no test-train split) using Logistic Regression (l1 penalty) and filter out low scoring words coef<0.2.

+ load cleaned Naive Bayes vocab from 2.1
+ load cleaned Logistic Regression (l2 penalty) from 2.2
+ run chi squared on full vocab and filter for terms p=<.05
+ outer join together in one df and save as `final_classification_features.csv`

**Input**
```
textfeatures_mat1
...
textfeatures_mat10 (data)
textfeatures_vocab (features/terms)
textfeatures_id (hadm_ids)
textfeatures_source (transfused/non-transfused)
LR_5000_final.pkl
LR_top_5000_terms_only.csv
NB_5000_final.pkl
NB_top_4879_terms_only_dist.csv
```
**Output**
```
final_classification_features.csv
```
# 2.4 Feature Selection via Frequency-Based Filtering
[2.4_filtered_vocab.ipynb](2.4_filtered_vocab.ipynb)

+ Python 3 environment

This notebook filters features using word frequencies (criteria below) and joins this list with the classification features.
+ Remove terms that are only digits (no letters)
+ Keep words that appear in < 10% of non-transfused documents, AND also change by more than 30% between the 2 groups (non transfused - transfused)
+ or are only in the transfused group
+ Inner Join with the vocabulary from 2.3

**Input**
```
transfused_notes_unique
ctrl_notes_unique
final_classification_features.csv
```
**Output**
```
all_filtered_features.csv
```
# 3.0 Topic Modeling
[3.0_topic_model_transfused_filtered_vocab.ipynb](3.0_topic_model_transfused_filtered_vocab.ipynb)

+ Python 3
We reduced the Transfused document vectors to include only the 41,664 terms from 2.x.

Vectorization (count) the transfused admissions using vocabulary from [2.4_filtered_vocab.ipynb](2.4.0_filtered_vocab.ipynb) `all_filtered_features.csv`, then LDA topic modeling. Plot results using pyLDAvis and send to SMEs for review. Several models, visualizations and dataframes can be saved from this notebook, but this example contains the parameters for the best/final model parameters.

**Input**
```
transfused_notes_unique
all_filtered_features.csv
```
**Output**
```
*_filtered_v1.html (pyLDA interactive visualization of topic model)
*_filtered_v1vect_model.pkl (count vectorizer)
*_filtered_v1vect_data.pkl (vectorized data)
*_filtered_v1_params.txt (topic modeling parameters)
*_filtered_v1_model.pkl (topic model)
*_filtered_v1_data.pkl (results of topic model)
*_filtered_v1_xx_words_scores.csv (top xx words scores for each topic)
*_filtered_v1_xx_words_proba.csv (top xx words proba for each topic)
*_filtered_v1_all_topic_scores_hadmids.csv
*_filtered_v1_max_topic_all_hadmids.pkl
*_filtered_v1_max_topic_all_hadmids.csv
*_filtered_v1_thresh_15_outlier_hadmids.pkl
*_filtered_v1_thresh_15_outlier_hadmids.csv
```
# Contributing

We encourage you to share any additions or changes to our code. To contribute, please:

Fork the repository using the following link: https://github.com/MIT-LCP/Shakespeare-Method/fork. For a background on GitHub forks, see: https://help.github.com/articles/fork-a-repo/

Commit your changes to the forked repository.

Submit a pull request to the MIMIC code repository, using the method described at: https://help.github.com/articles/using-pull-requests/

# License
By committing your code to the Bloatectomy Repository you agree to release the code under the GNU General Public License v3.0 in this repository.

# Bugs
Please feel free to create an issue for any questions, bugs, or suggestions you may have about our package or even the documentation (i.e. additional examples). We appreciate any feedback.
