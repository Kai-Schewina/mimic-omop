MIMIC-OMOP-WINDOWS
==========

This repository contains an Extract-Transform-Load (ETL) process for mapping the [MIMIC-III database](mimic.physionet.org) to the [OMOP Common Data Model](https://github.com/OHDSI/CommonDataModel). This process involves both transforming the structure of the database (i.e. the relational schema), but also standardizing the many concepts in the MIMIC-III database to a standard vocabulary (primarily the [Athena Vocabulary](https://www.ohdsi.org/analytic-tools/athena-standardized-vocabularies/), which you can explore [here](athena.ohdsi.org)). I changed the relevant ETL Scripts to work on Windows. For more in-depth documentation see the original mimic-omop repository by MIT.

To run the ETL, see the README-run-etl.MD file.

