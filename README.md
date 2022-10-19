# StandEnA: A customizable workflow to create a standardized presence/absence matrix of annotated proteins

## Introduction
This workflow was created to predict and annotate proteins 
from genomes and to build a standardized matrix of presence/absence from the annotated proteins.
The outcome of this workflow can be used for many purposes. For example, to infer the genetic potential 
of a given organism to perform a pathway of interest.

<p align="center">
<img
  src="img/StandEnA_schema.png"
  alt="Starting with enzyme identifiers for the pathways of interest, StandEnA has four steps"
  title="Workflow of StandEnA"
  style="display: inline-block; margin: 0 auto; max-width: 5000px">
</p>

**Workflow of StandEnA:** Starting with enzyme identifiers for the pathways of interest, StandEnA has four steps: 
Step 1 compiles enzyme synonyms and identifiers for these pathways from various databases. Step 2 creates a custom database from these enzyme protein sequences and annotates genomes. Step 3.1 creates a reference file with cross-database identifiers for each enzyme synonym used in the annotation and step 3.2 lists all of the enzymes of interest within the annotated genomes. Step 4 generates a standardized presence absence matrix for each enzyme within the desired pathway.

**Input and Output File Structure of StandEnA:** Refer to this figure for input and output files used at each step. The column structure of the figure is as follows: 1. input file name and description for specific step, 2. output file name and description for specific step, 3. description of the significance of the output for the pipeline. Color-filled boxes indicate files generated by StandEnA whereas the white background boxes indicate the files manually inputted by the user. Each input - output file pair is ordered according to its sequence of appearance in the pipeline and is seen under the corresponding main step number in line with the above figure. 

<p align="center">
<img
  src="img/StandEnA_input_output_files.png"
  alt="Starting with enzyme identifiers for the pathways of interest, StandEnA has four steps"
  title="Workflow of StandEnA"
  style="display: inline-block; margin: 0 auto; max-width: 5000px">
</p>

In **step 1**, the user inputs the [uniq_ec.tsv](examples/01_customdb/uniq_ec.tsv) file containing enzyme information separated by tab characters in this order: unique enzyme ID, pathway name, pathway step ID, enzyme name, and its enzyme commission (EC) number. Unique enzyme ID* and pathway step ID** are dependent on the naming convention preferred by the user. These IDs must be given using a consistent alphanumeric naming convention with no whitespace characters within the names. Pathway step ID*** is named according to the pathway number and the step at which the enzyme is working (e.g., pathway 1 step 1 is 1.1). This file is used to generate [01_customdb/id_synonyms_per_line.tsv](examples/01_customdb/id_synonyms_per_line.tsv) file containing the same information as the uniq_ec.tsv with the addition of available synonyms for each enzyme standard name from the KEGG database and the unique synonym ID**** for each of the retrieved synonyms. Synonym ID is a variation of unique enzyme ID to differentiate synonyms of the same enzyme. This information is used to retrieve protein sequences from the NCBI database via the Edirect application programming interface (API) and stored in the [01_customdb/edirect_fasta/](examples/01_customdb/edirect_fasta/) directory. Alternatively, the user can input EC numbers or KEGG Orthology (KO) identifiers within [ecs.txt](examples/01_customdb/ecs.txt) or [kos.txt](examples/01_customdb/kos.txt) files to retrieve protein sequences from KEGG database via OrtSuite. Furthermore, users are provided the option to use any means to download desired protein sequences from other databases which will be combined with the protein sequences downloaded by OrtSuite and stored within *01_customdb/manual_download_fasta/* directory.

In **step 2**, all protein sequences compiled in step 1 (within *01_customdb/manual_download_fasta/* and [01_customdb/edirect_fasta/](examples/01_customdb/edirect_fasta/)) are used to generate the custom database (*02_annotation/customdb.faa*). This database is used for genome annotation by Prokka along with its default database to generate the output [02_annotation/prokka_all.tsv](examples/02_annotation/example_prokka_all_results.tsv). 

In **step 3.1**, the files from step 1 ([01_customdb/id_synonyms_per_line.tsv](examples/01_customdb/id_synonyms_per_line.tsv), [ecs.txt](examples/01_customdb/ecs.txt), [kos.txt](examples/01_customdb/kos.txt)) are used to generate a reference file containing standard database identifiers for each enzyme including the EC number, standard enzyme name from KEGG, KO identifier, and enzyme synonyms used for the search with their EC numbers. Since some enzymes are defined by multiple EC numbers in KEGG, the last field sometimes contains additional EC numbers for the same enzyme. One reference file is generated for each pathway and compiled in the [03_standardization/pw_N_kegg_info.txt](examples/03_standardization/pw_1/pw_6_C_kegg_info.txt), [03_standardization/ortsuite_pw_N_kegg_info.txt](examples/03_standardization/pw_1/ortsuite_pw_1_kegg_info.txt), and/or *03_standardization/manual_pw_N_kegg_info.txt* for Edirect retrieved, OrtSuite retrieved and manually downloaded files, respectively. Moreover, the same files from step 1 are used to generate query files within the [03_standardization/pw_N/queries/](examples/03_standardization/pw_1/queries/) directory listing the possible synonym names for each enzyme.  In step **3.2**, [03_standardization/pw_N/queries/](examples/03_standardization/pw_1/queries/) files are used to select the matching annotations in [02_annotation/prokka_all.tsv](examples/02_annotation/example_prokka_all_results.tsv). The results are dumped to [03_standardization/standardized/results_pw_N.txt](examples/03_standardization/pw_1/results/standardized/) files. 

In **step 4**, [03_standardization/standardized/results_pw_N.txt](examples/03_standardization/pw_1/results/standardized/) is used to generate a standardized presence - absence matrix for all inputted genomes for pathway N. Alternatively, annotated enzymes from multiple pathways can be used to generate a single standardized presence - absence matrix. For this purpose, the [03_standardization/standardized/](examples/03_standardization/pw_1/results/standardized/) results are merged into *04_presabs/std_results_all.txt*. [02_annotation/prokka_all.tsv](examples/02_annotation/example_prokka_all_results.tsv) is processed to remove any problematic punctuation such as brackets and parentheses. This processed output file, *04_presabs/prokka_all_updated.tsv*, is used along with a user inputted file, (*04_presabs/ids_to_names.tsv*)[examples/04_presabs/ids_to_names.tsv], containing user-defined unique protein ID for each enzyme and corresponding standard protein names to be used in the standardized presence - absence matrix generation. The standardized presence - absence matrix output is generated in the file [04_presabs/presence_absence.csv](examples/04_presabs/example_presence_absence.csv).

## Installation instructions
Clone this repository
```bash
git clone https://github.com/mdsufz/std_enzymes
```
Install Miniconda3 and add channels
```bash
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh

conda config --add channels defaults
conda config --add channels bioconda
conda config --add channels conda-forge
```
Create and activate conda environment
```bash
conda create -n std_enzymes python=3.6.13 perl-lwp-simple perl-lwp-protocol-https prokka blast==2.9.0
```

Set Perl 5.22.0 default path for libraries
```bash
conda env config vars set PERL5LIB=$CONDA_PREFIX/lib/perl5/site_perl/5.22.0/ -n std_enzymes
```

Activate environment
```bash
conda activate std_enzymes
```

Install required packages inside the environment
```bash
conda install -c bioconda perl-lwp-simple prokka blast==2.9.0
```
## Dependencies
To manually download additional proteins using KEGG database identifiers, OrtSuite is required. 
Follow the installation steps for OrtSuite [here](https://github.com/mdsufz/OrtSuite/).

## System requirements and usage
A typical desktop (Linux) computer is capable of performing this workflow.
Disk space can be the most limiting resource for the annotation step as each annotated genome produces ~2 G of data. Therefore, it is recommended to have a fair ammount of free space depending on the number of genomes to be annotated.


## Workflow steps
1. [Compiling Protein sequences for the custom database from NCBI, KEGG and other databases](docs/CUSTOMDB.md)
2. [Generating a custom database and annotating genomes using Prokka with this custom database](docs/ANNOTATION.md)
3. [Generating the Reference File for enzymes used in the annotation and standardizing protein names in Prokka results](docs/STANDARDIZING.md)
4. [Generating matrix of standardized presence absence](docs/PRESABS.md)

## Contributions
Authors of pipeline: Fatma Chafra, Felipe Borim Corrêa,and Ulisses Nunes da Rocha

Principal Investigator: [Ulisses Nunes da Rocha](https://www.ufz.de/index.php?de=43142)

Institution: [Microbial Data Sciences group](https://www.ufz.de/index.php?de=43659), Helmholtz Center for Environmental Research, Department of Environmental Microbiology, Leipzig, Germany

All feedback is welcome. For errors and bugs, please open a new Issue thread on this github page, and we will try to address them as soon as possible. For general feedback you can contact us at mds@ufz.de. 
