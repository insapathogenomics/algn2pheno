# algn2pheno

**A bioinformatics tool for rapid screening of genetic features (nt or aa changes) potentially linked to specific phenotypes**


INTRODUCTION
------------------

The **algn2pheno** module screens a amino acid/nucleotide alignment against a "genotype-phenotype" database and reports the repertoire of mutations of interest per sequences and their potential impact on phenotype.


INPUT FILES
----------------

Table database in .tsv or .xlsx format 

**AND**        

Alignment (nucleotide or amino acid)   


(Mutation numbering will refer to the user-defined reference sequence included in the alignment).


INSTALLATION
----------------
```bash

pip install algn2pheno

```


USAGE
----------------

```bash

usage: algn2pheno [-h] [--db DB] [--sheet SHEET] [--table TABLE]
                     [--gencol GENCOL] [--phencol PHENCOL] -g GENE --algn ALGN
                     -r REFERENCE [--nucl] [--odir ODIR] [--output OUTPUT]
                     [--log LOG] [-f]

parse arguments

optional arguments:
  -h, --help            show this help message and exit
  --db DB               phenotype db. if excel, provide sheet name and columns
                        numbers for genotype and phenotype data. if not excel,
                        provide path to db, ensure 3 columns 'Mutation',
                        'Phenotype category' & 'Flag'
  --sheet SHEET, -s SHEET
                        Give sheet name (gene name or 'Lineages'). excel
                        input.
  --table TABLE, -t TABLE
                        table name in db. default: pokay. sqlite input.
  --gencol GENCOL       nr of column with genotype data. excel input.
  --phencol PHENCOL     nr of column with phenotype data. excel input.
  -g GENE, --gene GENE  Set gene or protein prefix
  --algn ALGN           Input alignment file
  -r REFERENCE, --reference REFERENCE
                        Give full reference sequence as in alignment file (use
                        quotation marks if this contains a space)
  --nucl                provide if nucleotide alignment instead of amino acid.
  --odir ODIR, -o ODIR  output directory
  --output OUTPUT       Set output file prefix
  --log LOG             logfile
  -f, --force           overwrite existing files


```


How to run (examples)
----------------------

**- Generic examples**

  1. database (.tsv) + amino acid alignment (SARS-CoV-2 Spike)

```bash
algn2pheno --db database.tsv --algn alignment_aa_Spike.fasta -g S -r reference_header --odir output_folder --output output_prefix
```

  2. database (.xlsx) + amino acid alignment (SARS-CoV-2 Spike)

```bash
algn2pheno --db database.xlsx --algn alignment_aa_Spike.fasta --sheet S --gencol ["Mutation" column number] --phencol ["Phenotype category" column number] -g S -r reference_header --odir output_folder --output output_prefix
```

___________________________

**- Screening of SARS-CoV-2 Spike epitope residues** listed in Carabelli et al, 2023, 21(3), 162–177, Nat Rev Microbiol (https://doi.org/10.1038/s41579-022-00841-7), Figure 1.

```bash
algn2pheno --db tests/DB_SARS_CoV_2_Spike_EpitopeResidues_Carabelli_2023_NatRevMic_Fig1.tsv -g S --algn alignment_aa_Spike.fasta -r Wuhan/Hu-1/2019 --odir output_folder --output output_prefix
```

NOTE: 

- The "alignment_aa_Spike.fasta" should contain the SARS-CoV-2 reference sequence (https://www.ncbi.nlm.nih.gov/protein/1796318598), and the fasta header should match the name included in the command line (**Wuhan/Hu-1/2019** in the example above). Find the Wuhan/Hu-1/2019 aa sequence in the "reference_gene_S.translation.fasta" file in "tests" folder. 

- If you do not have a Spike amino acid alignment, we recommend running **Nextalign (https://docs.nextstrain.org/projects/nextclade/en/stable/user/nextalign-cli.html)**. Example of command:

```bash
nextalign run -r reference.fasta -m nextalign_annotation.gff -g S -n nextalign -O nextalign_output_folder sequences.fasta
```
[to run this command you need the following files (available in the "tests" folder"): reference.fasta, nextalign_annotation.gff and reference.txt]

```bash
cat reference_gene_S.translation.fasta nextalign_gene_S.translation.fasta > alignment_aa_Spike.fasta 
```
[this command will add the Wuhan/Hu-1/2019 protein reference sequence (reference_gene_S.translation.fasta) to the output nextalign alignment (nextalign_gene_S.translation.fasta]




REQUIREMENTS
-----------------

The input database must contain mutations and associated phenotypes, listed in columns named `Mutation` and `Phenotype category` respectively. An additional column `Flag` may be provided for the ".tsv" input format.

The database can be provided in either tab separated (.TSV) or excel (.XLSX) format. In the case of using an excel file as input, it might contain several worksheets for distinct genes / proteins (being the gene/worksheet under analysis indicated using the argument --gene in the command line), and must contain two mandatory columns: “Mutation”, with the amino acid substitutions to screen (in relation to the reference sequence used), and “Phenotype category” with the associated phenotypes. Also, in the “.xlsx” database format, an intermediary .tsv database will be created and a "Flag" column will be generated automatically with Flags "D" and "P" to differentiate mutation(s) Directly and solely associated with the phenotype ("D"), and mutation(s) Partially associated with the phenotype (i.e., the mutation is Part of a constellation associated with that phenotype) ("P"). Constellation of mutations should be included in the same row separated by commas (e.g., H69del,H70del,L452R,N501Y). If providing a database in “.TSV” format, the "Flag" column should be added manually and can be used as a flexible field for additional attributes (e.g., in the COG-UK Antigenic mutations database, the antigenic mutation E484K occurs in the RBM domain (Phenotype category) and is associated with high confidence (Flag)).

Undefined amino acids should be included as "X" in the protein alignment.
Undefined nucleotides should be included as "N" in the gene alignment.

**Modules:**

> Main alignment to phenotype.
  - python=3.7
  - biopython
  - numpy 
  - pandas

> Further modules required to scrape Pokay database.
  - openpyxl
  - requests
  - pygithub
  - sqlalchemy
  - beautifulsoup4
  - lxml


INSTALLATION
-----------------

- git clone this repository.
- install and activate environment with required modules (see _pyenv.yml_).

CONFIGURATION
-----------------

This module does not require configuration.   



MAIN OUTPUTS
------------------------

> **_full_mutation_report.tsv**: provides the repertoire of "Flagged mutations" (i.e., database mutations detected in the alignment), the list of "phenotypes" supported by those mutations of interest and the list of "All mutations" detected in alignment for each sequence.

> **_flagged_mutation_report.tsv**: "Flagged mutation" binary matrix for all sequences and the "associated" phenotypes.


Other intermediate outputs:

> _all_mutation_matrix:  matrix of all mutations found in the alignment [mutations coded as 1 (presence), 0 (absence)]. At this stage, undefined amino acids ("X") or undefined nucleotides ("n") are not yet handled as no coverage positions.

> _all_mutation_report: matrix of all mutations found in the alignment [mutations coded as 1 (presence), 0 (absence) or nc (no coverage) positions] and associated phenotypes for the mutations in the database.

> _all_db_mutations_report: matrix of mutations in the database [mutations coded as 1 (presence), 0 (absence) or nc (no coverage) positions] and associated phenotypes, regardless if they were found or not in the alignment

> database.CLEAN.tsv: conversion of the ".xlsx" database into a clean ".tsv" format adding the gene "prefix" (-g argument) to each mutation (similar to when providing a .tsv database with the three headers 'Mutation', 'Phenotype category' and 'Flag')

MAINTAINERS
----------------

Current maintainers:

- Joao Santos (santosjgnd) 

- Bioinformatics Unit of the Portuguese National Institute of Health Dr. Ricardo Jorge (INSA) (insapathogenomics)



CITATION
----------

If you run algn2pheno, please cite:

João D. Santos,  Carlijn Bogaardt, Joana Isidro, João P. Gomes, Daniel Horton, Vítor Borges (2022). Algn2pheno: A bioinformatics tool for rapid screening of genetic features (nt or aa changes) potentially linked to specific phenotypes.


FUNDING
----------------

This work was supported by funding from the European Union’s Horizon 2020 Research and Innovation programme under grant agreement No 773830: One Health European Joint Programme.

