# Data Lineage Tool
An end-to-end, static analysis data lineage system designed for telecom data warehouses. It automatically reconstructs data flows across SQL scripts, Shell/ETL orchestration, and dataset schemas - without executing any code, then exports standardized lineage events to OpenLineage/Marquez.

# Purpose
This tool extracts SQL & Shell lineage, enriches it with schema (DDL) metdata, scores column mappings with ML, classifies datasets (Tabels) and export OpenLineage events To MARQUEZ for visualization of the data lineage.


# Commands (User Manual)
## Entry Point
Use the Typer CLI from the repository root:
```bash
python main.py --help
```

## Pipeline
### Run
```bash
python main.py pipeline run
```
Runs the full chain:
- C1 SQL parsing
- C1 Shell parsing
- C2 schema enrichment
- C3 entity resolution
- C3 dataset classification
- C4 OpenLineage standarization/export


### Schema Validation : 
```bash
python main.py pipeline check schema
```
Validates that the lineage extracted from SQL scripts still matches the declared DDL schema.

For each SQL statement, it verifies:
- whether the target table exists in the DDL
- whether the target column exists
- whether the SQL data type adn the DDL data type look different



## ML
### List Candidate Mappings :
```bash
python main.py ml review list
```
Show more rows if needed : 
```bash
python main.py ml review list --limit 0
```

Shows the candidate column mappings that the ML resolver is not fully confident about yet

For each candidate mapping, it prints : 
- the SQL script name
- the target table
- the source column
- the mapped target column
- the score
- the current label

### Interactive Review : 
```bash
python main.py ml review interactive
```
Open a validation loop  for the candidate mappings

For each one, it shows :
- the SQL script name
- the source column
- the target column
- the score

Then it asks you whether to confirm the link or not

## Impact Analysis
### Table Impact :
```bash
python main.py impact table --name [TABLE_NAME]
```
Example :
```bash
python main.py impact table --name dwh_prod.fact_cdr
```
Shows all downstream tables that depend on <b>TABLE_NAME</b>

It start from the table you give and walks the lineage graph forward to find :
- direct children
- indirect children
- all tables that are affected downstream

And then it prints : 
- the upstream table <b>(TABLE_NAME)</b>
- a numbered list of downstream tables

### Column Impact :
```bash
python main.py impact column --name [TABLE_NAME.COLUMN_NAME]
```
Example : 
```bash
python main.py impact table --name dwh_prod.fact_cdr.network_type
```

Shows all downstream columns that depend on <b>COLUMN_NAME</b>

It follows :
- direct column dependencies
- indirect dependencies through intermediate transformations
- the full chain of affected columns

And it prints :
- the upstream columns you asked about <b>(COLUMN_NAME)</b>
- a numbered list of downstream columns

## Governance
### PII Scan : 
```bash
python main.py governance scan-pii
```
Scans the classified outputs and lists the tables that countains PII

It checks whether each table has the ``PII`` tag, based on the ``Dataset Classifier`` results

It prints a table with : 
- the table name
- the JSON files where that table was found
- the PII tag

## Export
### Generate OpenLineage Events : 
```bash
python main.py export openlineage
```
Converts the classified lineage outputs into the ``OpenLineage`` Standard Format (Events) and sends them to the configured backend for visualization, usually (for my case) ``MARQUEZ``

#### Dry-Run Mode :
```bash
python main.py openlineage --dry-run
```
Generate the OpenLineage Events, but does <b>NOT</b> send them to the BackEnd (``MARQUEZ``)

## Dev Utilities


### Clean-up (Caches) : 
```bash
python main.py dev clean --confirm
```
Removes local generated cache files and temporary Python artifacts

It asks for confirmation first, so you don't delete things by mistake

### Clean-up (JSONs) :
```bash
python main.py dev clear-json --confirm
```
Deletes generated JSON outputs from the pipeline so you can rerun everything from scratch

It asks for confirmation before deleting anything
