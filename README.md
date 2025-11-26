# Introduction to  working with phylociraptor

## Installing phylociraptor

1. Create a conda environment with snakemake (this is already provided for you!)
If you don't have conda installed, first look [here](https://docs.conda.io/en/latest/miniconda.html).

> [!NOTE]
> No need to run the first two commands. We have already prepared this for you. Simple activate the conda environment.


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

## Download the necessary input files:

Phylociraptor uses two main files which serve as input to the pipeline. One contains all the settings for the analysis in yaml format. The other one contains a list of all the samples.

Let's do it:

The are part of this Github repository. The commands below assume that you are inside your phylociraptor directory:

```
curl https://raw.githubusercontent.com/reslp/phylociraptor-workshop/refs/heads/main/config.yaml > data/config.yaml
curl https://raw.githubusercontent.com/reslp/phylociraptor-workshop/refs/heads/main/small.csv > data/small.csv 
```

We will analyse a fungal dataset to confirm that Dikarya are monophyletic and consist of the two fungal groups Ascomycota and Basidiomycota.
The dataset consists of 20 fungal taxa. You can get some more information about them [here](https://github.com/reslp/phylociraptor-workshop/blob/main/fungi-info.md).



## One extra step...

This step here is usually not necessary. We do it here the speed up the subsequent analysis. phylociraptor uses singularity containers to run different software so we don't have to install them individually (minimal number of dependencies). These containers can have large file sizes and are stored in large container repositories. To skip this download step we have predownloaded them for you. Now we just have to move them to the correct location.

Inside your phylociraptor directory run:

```
mkdir -p .snakemake/singularity
cp -v ~/Share/phylociraptor/.snakemake/singularity/* .snakemake/singularity/
```


## setup the pipeline:

This runs on a single thread (local=1) and the same machine. It should take about 5 minutes.

```
./phylociraptor setup -t local=2 --verbose
```

## Modify the used busco set

We will modify the downloaded BUSCO set to only 20 genes to speed up computation. These genes have been selected because they are present in almost all of the genomes.

```
./phylociraptor util modify-busco -b fungi_odb10 -g 101133at4751,126519at4751,129520at4751,20600at4751,23198at4751,235463at4751,300016at4751,310891at4751,312080at4751,371481at4751,384315at4751,386245at4751,244066at4751,245900at4751,251158at4751,26329at4751,47592at4751,488348at4751,490662at4751,73383at4751
```

This should take only a few seconds. After the script is finished (no errors should show up). 

> [!IMPORTANT] 
> You have to follow the instructions on screen and modify the config file accordingly to be able to use the modified BUSCO set!
> In short: In your `config.yaml` file, under **orthology** and **busco_options** modify the the line `set: "fungi_odb10"` to `set: "fungi-seed--genes-20_odb10"` 


## Run orthology to indentify single-copy orthologs:

Identifying the single-copy orthologs as determined by the BUSCO set specified in your config file can be done with the `./phylociraptor orthology` command.

We will not actually run the process, but we can get an impression of what `phylociraptor` would be doing by performing a so-called dry run, which we consider very useful. Note that the `--dry` option can be used at all stages of `phylociraptor`. Let's try it out.

```
./phylociraptor orthology -t local --dry --verbose
```

> [!CAUTION]
> We will not run this step ourselves due to the high computational demands. Instead we will use precomputed results.
> Copying this should take about 1-2 minutes.

Inside your phylociraptor directory run:
```
rsync -avz --progress /home/$USER/Share/phylociraptor/results/ ./results/
```

Verify that orthology step is now considered completed, i.e. all expected files are there, with `phylociraptor check`.

For completeness, here is the command we used to create these results on a HPC cluster - specifically using the SLURM queueing system - due to high computational demands - yes, indeed, `phylociraptor` can do this ..

```
./phylociraptor orthology -t slurm -c data/cluster-config-GSC.yaml.template
```

## Filter orthology results:

```
./phylociraptor filter-orthology -t local=1
```

## Create alignments of single-copy orthologs:

```
./phylociraptor align -t local=4 --verbose
```

## Filter alignments to remove poorly aligned regions:

```
./phylociraptor filter-align -t local=2 --verbose
```

## Use filtered alignments to estimate the best substitution model and calculate gene trees

This should take about 10 minutes.

```
./phylociraptor modeltest -t local=2
```

## Calculate concatenated Maximum-Likelihood trees

This should take about 15 minutes.

```
./phylociraptor mltree -t local=4 --debug
```

## Calculate species trees

```
./phylociraptor speciestree -t local=1
```

## Create a reports of the run

During the first time this takes about 3-5 minutes.

```
./phylociraptor report --verbose
```

## A posteriori analysis of tree

Download NCBI lineage information for tree annotation:

```
./phylociraptor util get-lineage -d data/small.csv -o lineage-info.txt
```

Estimate conflicts between tree:

```
./phylociraptor util estimate-conflict -i all -o tipcov200 -s 43 -l lineage-info.txt -t 2 -b tipcoverage=200
```

Plot overview of conflicts/disagreements between trees:

```
./phylociraptor util plot-heatmap -m tipcov200.similarity_matrix.csv -r tipcov200.treelist.tsv
./phylociraptor util plot-pca -r tipcov200.treelist.tsv -s 42
```

Create tree plots as PDFs:

```
./phylociraptor util plot-tree --intrees $(cat tipcov200.treelist.tsv | awk '{print $2}' | tr "\n" "," | sed 's/\(.*\),/\1/') --lineagefile lineage-info.txt --level class --seed 43 --outgroup Mucor_racemosus,Glomus_cerebriforme,Smittium_simulii
```

Plot conflicts between two tree:

This may not work when there are no conflicts between your trees.

```
./phylociraptor util plot-conflict -i T1,T3 -q tipcov200.quartets.csv -r tipcov200.treelist.tsv -s 42 -l lineage-info.txt -e class -g Mucor_racemosus,Glomus_cerebriforme,Smittium_simulii
```



