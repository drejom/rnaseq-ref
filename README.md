# RNAseq Ref
Pipeline for RNA sequencing analysis using a reference genome at The University of Sydney, based on the NGI best practice pipleine at Scilifelab Stockholm, Sweden 

Original pipleine by Phil Ewels (@ewels) and Rickard Hammarén (@Hammarn). Updated to run on USyd HPC Artemis by Denis O'Meally (@drejom)

## Installation
### NextFlow installation
To use this pipeline, you need to have a working version of NextFlow installed. You can find more
information about this pipeline tool at [nextflow.io](http://www.nextflow.io/). The typical installation
of NextFlow looks like this:

```
curl -fsSL get.nextflow.io | bash
mv ./nextflow ~/bin
```

### NextFlow configuration
Next, you need to set up a config file so that NextFlow knows how to run and where to find reference
indexes. You can find an example configuration file for UPPMAX (milou) with this repository:
[`example_uppmax_config`](https://github.com/drejom/rnaseq-ref/blob/master/nextflow.config).

Copy this file to `~/.nextflow/config` and edit the line `'-P RDS-FVS-KoalaGen-RW'` to contain your own Artemis project
identifier instead.

### Pipeline installation
This pipeline itself needs no installation - NextFlow will automatically fetch it from GitHub when run if
`drejom/rnaseq-ref` is specified as the pipeline name.

If you prefer, you can download the files yourself from GitHub and run them directly:
```
git clone https://github.com/drejom/rnaseq-ref.git
nextflow run rnaseq-ref/rna-bp.nf
```

## Running the pipeline
The typical command for running the pipeline is as follows:
```
nextflow run drejom/rnaseq-ref --reads '*_R{1,2}.fastq.gz' --genome 'phaCin4'
```
or using a more manual approach ( require you to clone the git repository)

```
nextflow rnaseq-ref/rna-bp.nf -c path_to_your_nextflow_config --reads '*_R{1,2}.fastq.gz' --genome 'phaCin4'
```

Note that the pipeline will create files in your working directory:
```bash
work            # Directory containing the nextflow working files
results         # Finished results for each sample, one directory per pipeline step
.nextflow_log   # Log file from Nextflow
# Other nextflow hidden files, eg. history of pipeline runs and old logs.
```

### `--reads`
Location of the input FastQ files:
```
 --reads 'path/to/data/sample_*_{1,2}.fastq'
```

**NB: Must be enclosed in quotes!**

Note that the `{1,2}` parentheses are required to specify paired end data. Running `--reads '*.fastq'` will treat
all files as single end. The file path should be in quotation marks to prevent shell glob expansion.

If left unspecified, the pipeline will assume that the data is in a directory called `data` in the working directory.

### `--genome`
The reference genome to use of the analysis, needs to be one of the genome specified in the config file.
The koala `phaCin4` genome is set as default.
```
--genome 'phaCin3'
```
The `nextflow.config` file currently has the location of references for `phaCin4` (koala PacBio - falcon), `phaCin3` (koala hybrid PacBio)
and `phaCin1` (koala illumina - abyss).

### `--sampleLevel`
Used to turn of the edgeR MDS and heatmap, which require at least three samples to work. I.e use this when
running on one or two sample only.

### `--strandRule`
Some RSeQC jobs need to know the stranded nature of the library. By default, the pipeline will use
`++,--` for single end libraries and `1+-,1-+,2++,2--` for paired end libraries. These codes are for
strand specific libraries (antisense). `1+-,1-+,2++,2--` decodes as:

*  Reads 1 mapped to `+` => parental gene on `+`
*  Reads 1 mapped to `-` => parental gene on `-`
*  Reads 2 mapped to `+` => parental gene on `-`
*  Reads 2 mapped to `-` => parental gene on `+`

Use this parameter to override these defaults. For example, if your data is paired end and strand specific,
but same-sense to the reference, you could run:
```
nextflow run rnaseq-ref/rna-bp.nf --strandRule '1++,1--,2+-,2-+'
```
Use `--strandRule 'none'` if your data is not strand specific.


### `-c`
Specify the path to a specific config file (this is a core NextFlow command). Useful if using different UPPMAX
projects or different sets of reference genomes.

