# Running the ETL on PostgreSQL

This README overviews running the MIMIC-OMOP ETL from the ground up on a PostgreSQL server. You will need an installation of PostgreSQL 9.6+ in order to run the ETL. You will also need MIMIC-III installed on this instance of postgres, see here for details: (https://mimic.mit.edu/docs/gettingstarted/local/install-mimic-locally-windows/)

This README will assume the following:

* MIMIC-III v1.4 is available in the `mimic` database under the `mimiciii` schema
* The standard concepts from Athena have been downloaded and are available somewhere (including running the extra script to download CPT code definitions)
* The R software library with remotes (`install.packages("remotes")`) and a GitHub package `remotes::install_github("r-dbi/RPostgres")`. Additionally, R and Rtools should be in the PATH environment variable (e.g., "C:\Program Files\R\R-4.2.2\bin" and "C:\rtools43\usr\bin").
```bash
Rscript install.packages("remotes")
Rscript remotes::install_github("r-dbi/RPostgres")
```
* The computer has an active internet connection (needed to clone certain repositories throughout the build)
* Postgresql bin and lib should be in the PATH environment variable (e.g., "C:\Program Files\PostgreSQL\15\lib" and "C:\Program Files\PostgreSQL\15\bin").
* Git should be installed and available in the PATH environment variable

## 0. Open up a terminal and navigate to the mimic-omop-Windows directory

## 1. Build OMOP tables with standard concepts

See [the omop/build-omop/postgresql/README.md file](omop/build-omop/postgresql/README.md) to build OMOP on postgres.

## 2. Create local MIMIC-III concepts

We need to create a `concept_id` for each MIMIC-III local code. OMOP reserves `concept_id` above 20,000,000+ for local codes, so we will use this range to insert ours. If your mimic schema ist different to 'mimiciii' you need to change the first line in the mimic/build-mimic/postgres_create_mimic_id.sql accordingly.

```sh
psql -d mimic -U postgres -f mimic/build-mimic/postgres_create_mimic_id.sql
```

N.B. this script is called by `etl_sequence.sql`

After this, every table in the MIMIC-III schema will have an additional column called `mimic_id`.

## 3. Load the concepts from the CSV files

Edit the configuration file for the R script `mimic-omop.cfg` in the root folder of this repository. Here is an example of the file structure:

```sh
dbname=mimic
user=postgres
port=5432
password=FILL HERE
```

After that, run the R script from the root folder:

```
Rscript etl/ConceptTables/loadTables.R mimiciii
```

This will load various manual mappings to the database under the `mimiciii` schema.

## 4. Run the ETL

Be sure to run this from the *root* folder of the repository, or the relative path names will cause errors. The script assumes mimic to be in the mimiciii schema, else change the "SET SEARCH_PATH TO mimiciii" line in the "etl/etl.sql".

```sh
psql -d mimic -U postgres -f "etl/etl.sql"
```

## 5. Check the ETL has run properly

In order to run the checks, you'll need [pgTap](http://pgtap.org/). pgTap is a testing framework for postgres.
You can install pgtap by either:

* using a package manager, e.g. on Ubuntu using: `sudo apt-get install pgtap`.
* from source, following the [install instructions here](https://pgxn.org/dist/pgtap/)

If building from source, pay careful attention to the make output. You may need to install additional perl modules in order to use functions such as pg_prove, using `cpan TAP::Parser::SourceHandler::pgTAP`.
You may also need to run the installation `make` files as the postgres user, who has superuser privileges to the postgres database.

After you install it, be sure to enable the extension as follows:

```sh
psql "$MIMIC" -c "CREATE EXTENSION pgtap;"
```

Now the extension is available database-wide, and we can run the ETL.

```sh
psql "$MIMIC" -f "etl/check_etl.sql"
```
