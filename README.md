# Nanopore_analysis
Scripts and instructions for Nanopore data processing and analysis

## Assembly pipeline
Takes **BASECALLED NANOPORE READS** as input, assembles with Flye, and polishes assembly with Medaka.

This pipeline is for **Nanopore-only** assemblies. Pipeline for polishing Nanopore assemblies with Illumina reads is in process.

### Required input

In working directory:

* Subdirectory containing basecalled, demultiplexed reads (usually from guppy). You can name this directory anything you want. Typically, the name is something like `[date]_demultiplexed/`
* `samples.txt` file relating desired sample IDs to their assigned barcode. Format: tab delimited with header; column 1 = sample names; column 2 = barcode assignments. Note that the barcode names must match the names of the subdirectories in the demultiplexed reads input directory. See example `samples.txt` file [here](assembly_pipeline/samples.txt).
* `slurm_genomics_pipeline/` subdirectory: contains config information for snakemake
* `conda_envs/` subdirectory: contains YAML files for making Flye and Medaka conda environments
* `Snakefile` containing pipeline instructions
* `start_snakemake.sh` script to submit the job to the cluster. Note that this file must be executable! To change permissions, use the command `chmod +x start_snakemake.sh`

### Expected output

In working directory:

* `fastqs/` subdirectory: for each barcode, will contain a single file with all input fastq reads concatenated into a single file
* `flye/` subdirectory: all output from flye assembler
* `medaka/` subdirectory: all output from medaka polisher. **The final, polished genomes you probably want are here:** `.../medaka/{samplename}/consensus.fasta`
* `logs/` subdirectory: snakemake log files
* `nanopore_flye_medaka.err` file: contains everything that the processes in the Snakefile pipe to STDOUT during the pipeline's run. You can check this as the pipeline runs to see where you are. You can also look at it if something fails to try to figure out what went wrong.
* `nanopore_flye_medaka.out` file: contains anything piped to STDOUT when the cluster submission finishes; usually this file is empty if all has gone well!

In the directory where your conda environments are stored:

The pipeline will install new flye and medaka conda environments for its own use. These will be named some crazy long string of characters and will not conflict with existing flye or medaka environments you might have. You can manually delete these once you are satisfied that you have the output you are looking for.

### Running the pipeline

#### Edit the following files:

* In `slurm_genomics_pipeline/config.yaml`, change the path listed next to `conda-prefix:` to the directory where your own conda environments are stored. (This is where the flye and medaka environments for the pipeline will be installed.)
* In `start_snakemake.sh`, enter your own email address in the header to receive an email when the job finishes successfully. Optionally, you can add additional #SBATCH instructions to e.g. get emails if the job quits as a result of an error.
* In `Snakefile`, change the `get_input_fastqs` function so that it contains the name of the directory containing your demultiplexed reads. The final line should read: `barcode_directory = "YOUR_DIRECTORY_NAME_HERE/" + barcode`
* In `Snakefile`, make sure the model specified in the medaka call with the `-m` flag matches the settings used for your Nanopore run and basecalling. You can read more about medaka models [here](https://github.com/nanoporetech/medaka/blob/master/README.md).

#### Submit the job

Activate the snakemake conda environment. (This does not come pre-installed on the cluster, so you may need to install it first.)

Submit your job to the cluster using the following command: `sbatch start_snakemake.sh`

You can check on the progress of your job with `squeue`, `sacct`, or by looking at the contents of `nanopore_flye_medaka.err`.


  
