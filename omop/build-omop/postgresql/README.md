HOW TO BUILD
============

Clone the mimic-omop repository:

```bash
git clone git@github.com:MIT-LCP/mimic-omop.git
cd mimic-omop
```

All the following commands are assumed to be run in the root path of the repository, i.e. the mimic-omop folder you just cloned and changed into.

Now clone the OMOP Common Data Model DDL to a subfolder. Note that we reset the sub-repository to a specific commit to ensure that the DDL copied is always the same.

```bash
git clone https://github.com/OHDSI/CommonDataModel.git
cd CommonDataModel
git reset --hard 0ac0f4bd56c7372dcd3417461a91f17a6b118901
cd ..
cp CommonDataModel/PostgreSQL/*.txt omop/build-omop/postgresql/
```

Modify the DDL a bit:
Open up "omop/build-omop/postgresql/OMOP CDM postgresql ddl.txt" and replace all occurences of "CREATE TABLE" with "CREATE UNLOGGED TABLE".

Define the PSQL connection parameters you would like to use.
We assume the following schema, db and user names:
schema: omop
dbname: mimic
user: postgres

Build the schema:

```bash
psql -d "mimic" -U "postgres" -c "DROP SCHEMA IF EXISTS omop CASCADE;"
psql -d "mimic" -U "postgres" -c "CREATE SCHEMA omop;"
psql -d "mimic" -U "postgres"
alter user postgres set search_path to 'omop'
CTRL+C to exit
psql -d "mimic" -U "postgres" -f "omop/build-omop/postgresql/OMOP CDM postgresql ddl.txt"
```

We alter the character columns to `text`, as there is no performance degradation. This also adds ~4 columns to the NLP table:

```bash
psql -d "mimic" -U "postgres" -f "omop/build-omop/postgresql/mimic-omop-alter.sql"
```

We add some comments to the data model:

```bash
psql -d "mimic" -U "postgres" -f "omop/build-omop/postgresql/omop_cdm_comments.sql"
```
Create a "data" and a "data/vocab" folder at the root of the cloned repository.

```bash
mkdir data\vocab
```

Replace all occurences in "omop/build-omop/postgresql/omop_vocab_load.sql" of "extras/athena" with "data/vocab". 

The "data/vocab" folder contains the vocabulary that can be downloaded from: https://athena.ohdsi.org/
Note: An account on the website is required, download all files. Further, you need to execute the command thats described in the Readme.txt that comes with the vocabulary. For that to work you need an API-Key from https://uts.nlm.nih.gov/uts/, the account registration might take up to three days. Further, a JAVA Installation is required: https://www.java.com/de/download/manual.jsp

Import the vocabulary:
Note: Encodings might be an issue. Make sure server-side encoding and client-side encoding is the same. In my case server_encoding was set to UTF8, while client encoding was WIN1252. To check that connect to your database and execute the two commands.
Additionally, this step might take a few hours.

```bash
psql -d "mimic" -U "postgres"
SHOW server_encoding;
SHOW client_encoding;
```

If client_encoding is equal to WIN1252, execute the following, else leave the psql console with Control + C and go to the import section:

```bash
SET client_encoding TO 'UTF8';
```

Note: Maybe its sufficient if both encodings are the same, either UTF8 or WIN1252. Seems like a problem if they are not.

Then import the vocabulary:

```bash
psql -d "mimic" -U "postgres" -f "omop/build-omop/postgresql/omop_vocab_load.sql"
```
