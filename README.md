# Shakespeare-Method [![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.4811611.svg)](https://doi.org/10.5281/zenodo.4811611)
#### Text mining electronic health records

The Shakespeare-Method repository contains the code we used (in a user friendly tutorial style) to develop a new method to identify attributed and unattributed potential adverse events using the unstructured notes portion of electronic health records. This repository consists of ipython notebooks that are meant to be linear and easy to follow without having to understand the entire codebase. The level of abstraction has been kept to a minimum for this reason. Copies of the notebooks rendered as PDFs are provided in the [/pdf/](./pdf) directory for quick viewing. 

Many methods for finding AEs in text rely on pre-defining possible AEs before searching for pre-specified words and phrases or manual labeling (standardization) by investigators. We developed a method to identify possible AEs, even if unknown or unattributed, without any pre-specifications or standardization of notes. Our method is inspired by word-frequency analysis methods used to uncover the true authorship of disputed works credited to William Shakespeare.

We chose two use cases, “Transfusion” and “Time Periods”.  Transfusion was chosen because new transfusion adverse event (AE) types were becoming recognized during the study data period; therefore, we anticipated an opportunity to find unattributed potential AEs (PAEs) in the notes. With the “Time Periods” case we wanted to simulate near real time surveillance. We chose time periods in the hope of detecting PAEs due to contaminated heparin during mid-2007 to mid-2008 that were announced in early 2008. We hypothesized that the prevalence of contaminated heparin may have been widespread enough to manifest in EHRs through symptoms related to heparin adverse events, independent of clinicians’ documentation of attributed AEs.

We used EHRs for adult critical care admissions at a major teaching hospital, 2001-2012 (MIMIC III Database). For each case, we formed a group of interest and a comparison group. We concatenated the text notes for each admission into one document sorted by date, and deleted replicate sentences and lists. We  identified statistically significant words in the group of interest vs. the comparison group. Documents in the group of interest were filtered to those words, followed by topic modeling on the filtered documents to produce  topics. For each topic, the three documents with the maximum topic scores were manually reviewed to identify PAEs. 

**The Shakespeare Method has five steps:**
 
+ Step 1. Convert each document into a vector of n-gram (term) frequencies.
+ Step 2. Create two groups of vectors: target and comparison.
+ Step 3. Extract terms in the target group that are significant for the target group.
+ Step 4. Apply topic analysis to the target group filtered vectors.
+ Step 5. Review the original documents that have topic scores of interest to interpret the topics and find potential adverse events.

Step 2 is described in detail in Bright RA, Bright-Ponte, SJ, Palmer LA, Rankin, SK, Blok S. Use of Diagnosis Codes to Find Blood Transfusion Adverse Events in Electronic Health Records. medRxiv 2020.12.30.20218610.  https://doi.org/10.1101/2020.12.30.20218610.

The other steps are described in detail in this repository. A flowchart of this process can be seen below.

![Shakespeare Method process with truncated examples](https://github.com/MIT-LCP/Shakespeare-Method/blob/main/images/SM_overview_pic.png)

# Citations
To acknowledge use of the code, please cite the DOI provided via Zenodo:

Summer K. Rankin, Katherine Dowdy, Roselie A. Bright.  (2021, May 26). MIT-LCP/Shakespeare-Method: Macbeth (Version v0.3). Zenodo. http://doi.org/10.5281/zenodo.4811611
```
@software{summer_k_rankin_2021_4811611,
  author       = {Summer K. Rankin and Kate Dowdy and Roselie A. Bright},
  title        = {MIT-LCP/Shakespeare-Method: Macbeth},
  month        = may,
  year         = 2021,
  publisher    = {Zenodo},
  version      = {v0.3},
  doi          = {10.5281/zenodo.4811611},
  url          = {https://doi.org/10.5281/zenodo.4811611}
}
```

and the papers describing the Shakespeare Method

Roselie A. Bright, Summer K. Rankin, Kate Dowdy, Sergey V. Blok, Susan J. Bright, and Lee Anne M. Palmer. (2021, Aug 11). Finding Potential Adverse Events in the Unstructured Text of Electronic Health Care Records: Development of the Shakespeare Method. JMIRx Med 2021;2(3):e27017  doi: 10.2196/27017 https://xmed.jmir.org/2021/3/e27017

```
@article{bright_roselie_2021_27017,
  author       = {Roselie A. Bright, Summer K. Rankin, Kate Dowdy, Sergey V. Blok, Susan J. Bright, and Lee Anne M. Palmer},
  title        = {Finding Potential Adverse Events in the Unstructured Text of Electronic Health Care Records: Development of the Shakespeare Method},
  month        = aug,
  year         = 2021,
  journal      = {JMIRx Med},
  volume       = {2},
  number       = {3},
  pages        = {e27017},
  doi          = {10.2196/27017},
  url          = {https://xmed.jmir.org/2021/3/e27017}
}
```

 
Bright RA, Bright-Ponte, SJ, Palmer LA, Rankin, SK, Blok S. Use of Diagnosis Codes to Find Blood Transfusion Adverse Events in Electronic Health Records. medRxiv 2020.12.30.20218610.  https://doi.org/10.1101/2020.12.30.20218610.


## Structure of repository
x.y_description
+ *x*=major section of analysis
+ *y*=order within section

  + 1.y = labeling, preprocessing, cleaning, tokenization, vectorization
  + 2.y = feature selection
  + 3.0 = LDA topic modeling

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
Anaconda environments were used to manage python packages and versions. We used both python2 and python3 environments that can be created by using the [shakesPy2_environment.yml](shakesPy2_environment.yml) for python 2.

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
# 1.0 Create Adult Inputs Table
[1.0_create_adult_inputs.ipynb](1.0_create_adult_inputs.ipynb)

+ Python2 environment
+ Time and Transfused studies
This notebook is the first of 2 steps to create the cohorts.

![flowchart_of_cohort_creation](./images/cohort.png)


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

# 1.1a Transfused Cohort Selection 
[1.1a_create_transfused_cohort.ipynb](1.1a_create_transfused_cohort.ipynb)

+ Python2 environment
+ Transfused Study Only

Label admissions and concatenate all notes for a single admission into one note (in chronological order).
Save notes in new tables `transfused_notes_sink` and `ctrl_notes_sink`.

+ Uses ICD-9 codes to create control, transfused, and grey (excluded) groups.
+ Get chart input items from the [20180717D_ITEMS_related_to_blood_full.csv](20180717D_ITEMS_related_to_blood_full.csv) to create a labeled dictionary of input items that belong to T=transfused, N=non-transfused, or G=grey(exclusion group).  

+ Transfused Group =  the union of admissions that have **ever** had a Transfused ICD-9 code **or** a Transfused input item.
+ Control (Non-transfused) = union of admissions that meet all 4 of the following conditions:
    1. No transfused ICD-9
    2. No grey ICD-9
    3. No transfused input item
    4. No grey input item


**Input:** 
```
procedures_icd
inputs_all
20180717D_ITEMS_related_to_blood_full.csv
D_items
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
```
# 1.1b Time-based Cohort Selection
[1.1b_create_time_based_cohort.ipynb](1.1b_create_time_based_cohort.ipynb)

+ Python2 environment
+ Time Study Only

Create a table in Postgres of admissions (and corresponding notes) related to heparin time periods by using a specific file that reveals the de-identified dates. 

1.1.0 Import hadm_ids for which time key exists <br />
1.1.1 Keep only adults from Carevue <br />
1.1.2 Pull admissions that match the hadm_ids with time key <br />
1.1.3 Pull admit times/ discharge times from admissions table <br />
1.1.4 Save to a new table in posgres <br />
1.1.5 Get notes for those admissions <br />
1.1.6 Calculate which admissions have any leap days <br />
1.1.7 Calculate and apply time shift to all dates to get true dates <br />
1.1.8 Double check the time shifts are correct (compare deltas) <br />
1.1.9 Make tables for notes in this study <br />

**Input:** 
```
year_key.csv
admissions
inputevents_cv_adult
noteevents
```
**Output:** Postgres tables
```
cv_real_dates
adult_cv_notes
time_study_id_date
time_study_notes
```

# 1.2 Concatenate Notes
[1.2_concat_notes.ipynb](1.2_concat_notes.ipynb)

+ Python 2 environment
+ Time and Transfused Studies

Retrieves all the notes from the cohort admissions, puts them in chronological order, and concatenates them into one large single document per admission.

**Input:** Postgres tables
```
transfused_hadm_id
grey_hadm_id
ctrl_ids
noteevents
```
**Output:** Postgres tables
```
transfused_notes
ctrl_notes
transfused_notes_sink
ctrl_notes_sink
```

# 1.3 Remove Notebloat
[1.3_bloatectomy.ipynb](1.3_bloatectomy.ipynb)

+ python3 environment b/c bloatectomy needs *python >= 3.7*
+ Time and Transfused studies. 

Modified bloatectomy code to remove duplicate sections of text within an admission's concatenated notes. 
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

# 1.4 Vectorization of Text
[1.4_vectorization.ipynb](1.4_vectorization.ipynb)

+ Python 2 environment (recommended in an AWS instance)
+ Time and Transfused studies

We used the collocation detection skip-gram method for extracting the n-grams with n = 1-5 consecutive words. We vectorized each document using a bag of words representation where each dimension is represented by the frequency (count) of each n-gram, resulting in a set of 7,422,044 words.

This notebook is used to tokenize, get collocations (ngrams), count vectorize the transfused and comparison concatenated notes.  This one has to be run on something with a large amount of ram (like 109Gb or so). Use the 'large' AWS instance. Alternatively, one could truncate the number of features (terms) to run this on a laptop or smaller instance.

We used an AWS (Amazon Web Services) EC2 memory-optimized instance (r4.8xlarge, 244GB memory, AMD64, Windows 10). The MIMIC database and POSTGRESQL must be accessible and the code in 1.0, 1.1, 1.2, should be run first to create the groups and deduplicate notes. The time to run the code was approximately 7.5 hours to create the groups, concatenate the notes into documents, and remove duplicate sections of text. We recommend using an expensive/large instance for only this step. All other steps can be done on a laptop with ~16 GB RAM.  

+ Saves as sparse matricies in pickle format (document-term matrix is broken up into 10 sections to make transfer back to a local computer or cheaper instance faster).
+ We recommend a transfer of the results (pickle files) to local computer or less expensive instance for further processing after vectorization is finished.

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

![Flowchart of the feature selection methods](./images/shakespeare_flowchart_publication.png)

# 2.0 Feature Selection via Classification Models
[2.0_classification_models.ipynb](2.0_classification_models.ipynb)

+ python 3 environment
+ Time and Transfused Study

Runs multiple classification models on any 2 groups (transfused and non-transfused/control) to select the features most associated with the one group (transfused) and save them for further analysis.

<img src="https://github.com/MIT-LCP/Shakespeare-Method/blob/main/images/supervised_flow.jpg" width="400" height="700">

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
[2.1_nBayes_remove_transfusion_terms.ipynb](2.1_nBayes_remove_transfusion_terms.ipynb)

+ python 3 environment
+ Transfused Study Only


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
[2.2_logReg_remove_transfusion_terms.ipynb](2.2_logReg_remove_transfusion_terms.ipynb)

+ python 3 environment
+ Transfused Study Only

After sorting the n-grams according to the feature log probability, we removed all terms making direct and exclusive reference to the act of transfusion, because they both merely repeat the criteria for membership in the transfused group and obscure terms from other notes that might be indications for or outcomes of transfusion; as would be expected, these terms were highly associated with being in the transfused group. These transfusion terms were identified for removal by manual inspection (necessary because of the variety of abbreviations and spellings) informed by the contexts of these terms in multiple notes. Each time these transfusion-related terms were removed, they were added to a dictionary of transfusion-related terms. In the end this list grew to contain almost 1000 different terms [terms_indicate_transfusion9.xlsx](terms_indicate_transfusion9.xlsx).

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
+ Time and Transfused Study

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
+ Time and Transfused Study

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
[3.0_topic_model_on_filtered_vocab.ipynb](3.0_topic_model_on_filtered_vocab.ipynb)

+ Python 3
+ Time and Transfused Study

We reduced the Transfused document vectors to include only the 41,664 terms from step 2.4.

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

# Plotting 
[time_study_plot.ipynb](time_study_plot.ipynb)

+ Python 3
+ Time Study Only

Plots for exploring the time-based cohort.
# Contributing

We encourage you to share any additions or changes to our code. To contribute, please:

Fork the repository using the following link: https://github.com/MIT-LCP/Shakespeare-Method/fork. For a background on GitHub forks, see: https://help.github.com/articles/fork-a-repo/

Commit your changes to the forked repository.

Submit a pull request to the MIMIC code repository, using the method described at: https://help.github.com/articles/using-pull-requests/

# License
By committing your code to the MIT-LCP/Shakespeare-Method Repository you agree to release the code under the GNU General Public License v3.0 in this repository.

# Bugs
Please feel free to create an issue for any questions, bugs, or suggestions you may have about our package or even the documentation (i.e. additional examples). We appreciate any feedback.
