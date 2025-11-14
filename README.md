# Introduction into working with phylociraptor

## Installing phylociraptor

1. Create a conda environment with snakemake (this is already provided for you!)
If you don't have conda installed, first look [here](https://docs.conda.io/en/latest/miniconda.html).

```
$ conda install -n base -c conda-forge mamba
$ mamba create -c conda-forge -c bioconda -n snakemake snakemake=6.0.2
$ conda activate snakemake
```

2. Clone the phylociraptor repository:

Phylociraptor is also hosted on GitHub: [https://github.com/reslp/phylociraptor](https://github.com/reslp/phylociraptor).


```
(snakemake) $ git clone --recursive https://github.com/reslp/phylociraptor.git
(snakemake) $ cd phylociraptor
(snakemake) $ ./phylociraptor

			     Welcome to
           __          __           _                  __            
    ____  / /_  __  __/ /___  _____(_)________ _____  / /_____  _____
   / __ \/ __ \/ / / / / __ \/ ___/ / ___/ __ `/ __ \/ __/ __ \/ ___/
  / /_/ / / / / /_/ / / /_/ / /__/ / /  / /_/ / /_/ / /_/ /_/ / /    
 / .___/_/ /_/\__, /_/\____/\___/_/_/   \__,_/ .___/\__/\____/_/     
/_/          /____/                         /_/                      

	  the rapid phylogenomic tree calculator, ver.1.0.0


Usage: phylociraptor <command> <arguments>

Commands:
	setup			Setup pipeline
	orthology		Infer orthologs in a set of genomes
	filter-orthology	Filter orthology results
	align			Create alignments for orthologous genes
	filter-align		Trim and filter alignments
	modeltest		Calculate gene trees and perform model testing
	mltree			Calculate Maximum-Likelihood phylogenomic trees
	speciestree		Calculate species trees
	njtree			Calculate Neighbor-Joining trees
        bitree                  Calculate Bayesian-inference phylogenomic trees

	report			Create a HTML report of the run
	check			Quickly check status of the run
	util			Utilities for a posteriori analyses of trees

	-v, --version 		Print version
	-h, --help		Display help

Examples:
	To see options for the setup step:
	./phylociraptor setup -h

	To run orthology inferrence for a set of genomes on a SLURM cluster:
	./phylociraptor orthology -t slurm -c data/cluster-config-SLURM.yaml

```



Run these commands from inside the phylociraptor directory in the given order to reproduce the example.

## Copy the necessary files:

```
cp data/test_cases/small/config.yaml data/
cp data/test_cases/small/small.csv data/
```

## setup the pipeline:

This runs on a single thread (local=1) and the same machine. It should take about 5 minutes.

```
./phylociraptor setup -t local=1
```

## Modify the used busco set

We will modify the downloaded BUSCO set to only 20 genes to speed up computation. These genes have been selected because they are present in almost all of the genomes.

```
./phylociraptor util modify-busco -b fungi_odb9 -g 101133at4751,126519at4751,129520at4751,20600at4751,23198at4751,235463at4751,300016at4751,310891at4751,312080at4751,371481at4751,384315at4751,386245at4751,244066at4751,245900at4751,251158at4751,26329at4751,47592at4751,488348at4751,490662at4751,73383at4751
```

This should take only a few seconds. After the script is finished (no errors should show up). 

**IMPORTANT**: You have to follow the instructions on screen and modify the config file accordingly to be able to use the modified BUSCO set!


## Run orthology to indentify single-copy orthologs:

The subsequent steps use a SLURM cluster. You may have to modify the correpsonding cluster config file accordingly.

```
./phylociraptor orthology -t slurm -c data/cluster-config-GSC.yaml.template
```

## Filter orthology results:

```
./phylociraptor filter-orthology -t local -c data/cluster-config-GSC.yaml.template
```

## Create alignments of single-copy orthologs:

```
./phylociraptor align -t slurm -c data/cluster-config-GSC.yaml.template
```

## Filter alignments to remove poorly aligned regions:

```
./phylociraptor filter-align -t slurm -c data/cluster-config-GSC.yaml.template
```

## Use filtered alignments to estimate the best substitution model and calculate gene trees


```
./phylociraptor modeltest -t slurm -c data/cluster-config-GSC.yaml.template
```

## Calculate concatenated Maximum-Likelihood trees


```
./phylociraptor mltree -t slurm -c data/cluster-config-GSC.yaml.template
```

## Calculate species trees

```
./phylociraptor speciestree -t slurm -c data/cluster-config-GSC.yaml.template
```

## Create a reports of the run

```
./phylociraptor report
./phylociraptor report --figure
```

## A posteriori analysis of tree

Download NCBI lineage information for tree annotation:

```
./phylociraptor util get-lineage -d data/small.csv -o lineage-info.txt
```

Estimate conflicts between tree:

```
./phylociraptor util estimate-conflict -i all -o tipcov200 -s 43 -l lineage-info.txt -t 8 -b tipcoverage=200
```

Create tree plots as PDFs:

```
./phylociraptor util plot-tree --intrees $(cat tipcov200.treelist.tsv | awk '{print $2}' | tr "\n" "," | sed 's/\(.*\),/\1/') --lineagefile lineage-info.txt --level class --seed 43 --outgroup Mucor_racemosus,Glomus_cerebriforme,Smittium_simulii
```

Plot conflicts between two tree:

```
./phylociraptor util plot-conflict -i T5,T15 -q tipcov200.quartets.csv -r tipcov200.treelist.tsv -s 42 -l lineage-info.txt -e class -g Mucor_racemosus,Glomus_cerebriforme,Smittium_simulii```



