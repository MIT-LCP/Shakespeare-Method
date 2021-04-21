# Shakespeare-Method
## Text mining electronic health records

Questions about this code can be sent to Summer Rankin (rankin_summer@bah.com, summerkrankin@gmail.com) or Kate Dowdy (dowdy_katherine@bah.com).

x.y_description
+ *x*=major section of analysis
+ *y*=order within section

  + 1.x = labeling, preprocessing, cleaning, tokenization, vectorization
  + 2.x = feature selection
  + 3.x = LDA topic modeling
  + 4.x = send to elasticsearch

## Data
This project uses the MIMIC III Database. The MIMIC-III database was ingested into in a PostgreSQL (version 10.11 downloaded January 12,2020) database using PostgreSQL scripts from (https://github.com/MIT-LCP/mimic-code/tree/master/buildmimic/postgres, accessed January 12, 2020.

### Connecting python to PostgreSQL
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
## Environment
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
### 1.0_create_groups
[1.0_create_groups.ipynb](1.0_create_groups.ipynb)

This file is the first of 2 steps to create the groups of adult admissions (>=16 years old), and the corresponding input events. Saves to new table in postgres database named `inputs_all`
+ uses python2 environment

## 1.1_create_groups_concat_notes
[1.1_create_groups_concat_notes.ipynb](1.1_create_groups_concat_notes.ipynb)

+ uses python2 environment

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
+ Saves as a new sql table `transfused_notes_sink` and `ctrl_notes_sink` in mimiciii postgres database for analysis.

## 1.2_duplicate_removal
[1.2_duplicate_removal.ipynb](1.2_duplicate_removal.ipynb)

+ uses python3 environment b/c bloatectomy needs *python >= 3.7*

Uses modified bloatectomy code to remove duplicate sections of text w/in an admission's concatenated notes. Saves to postgres database in new tables `transfused_notes_unique` and `ctrl_notes_unique`

## 1.3_vectorization.ipynb
[1.3_vectorization.ipynb](1.3_vectorization.ipynb)

+ uses python 2 environment in an AWS instance

Tokenize, get collocations (ngrams), count vectorize the data.  This one has to be run on something with a large amount of ram (like 109Gb or so). Use the 'large' AWS instance. Alternatively, one could truncate the number of features (terms) to run this on a laptop or smaller instance.

We used an AWS (Amazon Web Services) EC2 memory-optimized instance (r4.8xlarge, 244GB memory, AMD64, Windows 10). The MIMIC database and POSTGRESQL must be accessible and the code in 1.0, 1.1, 1.2, should be run first to create the groups and deduplicate notes. The time to run the code was approximately 7.5 hours to create the groups, concatenate the notes into documents, and remove duplicate sections of text. We recommend only using an expensive/large instance for this step. All other steps can be done on a laptop with ~16 GB RAM.  

+ Saves as sparse matricies in pickle format (document-term matrix is broken up into 10 sections to make transfering back to a local computer or cheaper instance faster)
+ We recommend a transfer of the results (pickle files) to local computer or less expensive instance for further processing.
Output:
```
textfeatures_mat1
...
textfeatures_mat10 (data)
textfeatures_vocab (features/terms)
textfeatures_id (hadm_ids)
textfeatures_source (transfused/non-transfused)
```

## 2.0_classification_models
[2.0_classification_models.ipynb](2.0_classification_models.ipynb)

+ python 3 environment

Runs multiple classification models on the 2 groups (transfused and non-transfused/control) to select the features most associated with the transfused group and save them for further analysis.

+ test/train split
+ naive bayes classification
+ logistic regression classification
+ other classification models
+ plot confusion matrix and roc auc for multiple models
+ save model, and metrics + vocab
+ saves top (transfusion group) 5,000 terms from naive bayes (log probability) and logistic regression (coef) models for topic modeling and/or further review (pickle and csv) 
```
top_logit_coef_5000.csv
logits_top_5000_matrix.pickle
NB_top_5000_feat_[date].csv
NB_top_5000_matrix.pickle
```

