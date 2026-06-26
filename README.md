# Directed Evolution Monitoring Web Portal – Staging Pipeline
QMUL MSc Bioinformatics Software Development Group Project 2026

## Overview
This repository contains my contribution to a group MSc Bioinformatics software engineering project: the development of the experimental staging and validation pipeline for a Directed Evolution Monitoring Web Portal.

The overall web portal was designed to support droplet-based microfluidic directed evolution experiments by providing an end-to-end computational workflow for validating experiments, analysing sequencing data, ranking protein variants, and visualising evolutionary trends.

My contribution focused on ensuring that experiments are biologically valid before downstream analysis by confirming that the uploaded plasmid genuinely encodes the user-selected UniProt protein.

## Overall Project
Directed evolution is a laboratory technique used to engineer proteins with improved properties through iterative cycles of mutation and selection.

Modern droplet-based microfluidic platforms generate very large experimental datasets, creating several computational challenges:

* automated validation of experimental inputs
* identification of coding sequences in circular plasmids
* reproducible activity scoring
* biological interpretation of variant data
* interactive visualisation of evolutionary trajectories

The web portal addresses these challenges by implementing the following workflow:
1. Experimental staging
2. Validation
3. Sequence parsing & QC
4. Variant analysis
5. Activity scoring
6. Visualisation
7. Reporting

## Repository Context
This repository contains only my contribution to the larger Directed Evolution Monitoring Web Portal developed as part of an MSc Bioinformatics group project.
My primary responsibility was the design and implementation of the experimental staging pipeline, including sequence validation, ORF identification, protein alignment, and experiment validation.

The complete web portal source code is available at:
https://github.com/sumaiyazainab/group-project-monetery-
* My primary contribution to the web portal is in the **Experimental Staging Pipeline** and the **Variant Analysis Pipeline - Mutation counting**.

The user manual I created for the web portal is available at:
https://sumaiyazainab.github.io/group-project-monetery-/
* My primary contribution to the user manual is writing the contents, uploading screenshots, and created the manual using MkDocs.

The full technical web portal design documentation is at:
[Directed Evolution Web Portal – Technical Documentation](docs/Directed_Evolution_Web_Portal_Technical_Documentation.pdf)
* My primary contribution to the documentation is in the **Purpose of Document**, **Scientific Context and Software Motivation**, **Design Philosophy**, **Database Design and Schema**, **Staging Pipeline**, **Limitations**, and **Future Development** sections.

## My Contribution
I designed and implemented the experimental staging pipeline, which validates user input before any downstream analysis is performed.
The pipeline verifies that:

* the supplied UniProt accession is valid
* the uploaded plasmid FASTA is valid
* the plasmid genuinely contains the expected coding sequence
* the best matching ORF is identified
* validated experiment metadata is stored for downstream analyses

This prevents incorrect reference proteins from propagating through the rest of the analysis pipeline.

## Staging Pipeline Workflow
The staging workflow consists of 7 major stages:
1. UniProt accession
2. Retrieve reference protein
3. Validate plasmid FASTA
4. Detect candidate ORFs
5. Length pre-filtering
6. Global protein alignment
7. Store validated experiment

## Design Principles
The staging pipeline was designed to be:
* Generalisable to any UniProt accession
* Robust to circular plasmids
* Compatible with alternative bacterial start codons
* Computationally efficient through ORF pre-filtering
* Biologically meaningful through protein-level alignment
* Fully reproducible through structured database storage

## Features
### 1. UniProt Reference Retrieval
Given a UniProt accession ID, the pipeline automatically retrieves:
* reference protein sequence
* protein length
* annotated domains
* protein feature metadata
Protein records are obtained through the UniProt REST API and parsed into structured pandas DataFrames for downstream analysis.

### 2. FASTA Validation
Uploaded plasmid FASTA files are validated before analysis.
Validation checks include:
* exactly one sequence
* valid DNA nucleotides only
* successful sequence parsing
* sequence length extraction
Invalid inputs terminate the staging process before downstream computation.

### 3. Circular Plasmid ORF Detection
The pipeline identifies candidate open reading frames across all six reading frames.
Features include:
* circular plasmid handling by sequence concatenation
* forward and reverse strand scanning
* configurable bacterial start codons
    * ATG
    * GTG
    * TTG
* minimum ORF length filtering
* ORF metadata extraction

Each detected ORF stores:
* strand
* reading frame
* nucleotide coordinates
* amino acid sequence
* translated DNA sequence
* whether the ORF spans the plasmid origin

### 4. Custom Translation Logic
A custom translation function was implemented because standard Biopython translation tables translate:
* GTG → Valine
* TTG → Leucine
whereas biologically these codons initiate translation as Methionine when acting as start codons.
The pipeline therefore converts alternative bacterial start codons into Methionine to ensure correct comparison against UniProt reference proteins.
Unknown codons are translated as X, allowing robust handling of ambiguous nucleotides.

### 5. Length-based Candidate Filtering
To improve computational efficiency, ORFs are filtered before alignment.
Candidate ORFs are retained only if their translated protein length lies within ±20% of the UniProt reference protein length.
This substantially reduces:
* alignment runtime
* false positive ORFs
* unnecessary protein comparisons

### 6. Global Protein Alignment
Remaining ORFs are aligned against the UniProt reference protein using:
* Biopython PairwiseAligner
* global alignment
* BLOSUM62 substitution matrix
* affine gap penalties

For each candidate ORF:
''' bash
Score fraction = Alignment score / Reference self-alignment score
'''
The ORF with the highest score fraction is selected.
Experiments are accepted when score fraction ≥ 0.85.
This threshold allows moderate biological variation while rejecting unrelated ORFs.

## Data Stored
Following successful validation, the pipeline stores:
* experiment metadata
* UniProt reference protein
* protein domains
* plasmid sequence
* all detected ORFs
* best matching ORF
* alignment score
* experiment status
This ensures complete traceability and reproducibility.

## Technologies
* Python
* Biopython
* pandas
* requests
* UniProt REST API
* JSON
* FASTA
* Pairwise sequence alignment
* BLOSUM62 substitution matrix
