# Atypical-endometriosis-and-normal-endometriosis-GCN
A lab designed to create GEMs from atypical endometriosis (which is linked to ovarian cancer) and normal endometriosis, and then use downstream workflows to create a GCN. 

### Lab Overview
This lab will take SRA data from NCBI, create GEMs with them, and then the GEMs will be processed to create a GCN. GCNs can be used to identify networks of gene relationships. Specifically, we will pull SRA data from atypical endometriosis samples and "normal" endometriosis samples, create GEMs from them, then use these GEMs to create a GCN that can attempt to identfy the causality of the phenotypic changes seen between atypical endometriosis and normal endometriosis. 

### Learning Objectives
 * Pull SRA data from NCBI
 * Download GEMmaker and be able to use it to create GEMs from SRA data
 * Use GEMprep to normalize and prep the GEMs
 * Preprocess the GEMs for KINC
 * Build a co-expression network with KINC
 * Visualize the network with Cytoscape

### Lab Tasks
* Make sure singularity and nextflow have been installed
  * to install nextflow: curl -s https://get.nextflow.io | bash 
  * install singlarity using instructions found here: https://sylabs.io/guides/3.7/user-guide/quick_start.html#quick-installation-steps
* Make a working directory
* Inside this directory, make two sub directories-one for endometriosis and one for atypical endometriosis (ex. GEMmaker_endometriosis and GEMmaker_atypical)
* Do steps 5-12 in both the endometriosis and atypical directories
* Clone GEMmaker
  * git clone https://github.com/SystemsGenetics/GEMmaker.git --branch develop
* Go to GEMmaker/demo/references 
  * In the demo/references directory, download the human genome and GTF files
    * wget http://ftp.ensembl.org/pub/release-101/fasta/homo_sapiens/dna/Homo_sapiens.GRCh38.dna.primary_assembly.fa.gz 
    * wget ftp://ftp.ensembl.org/pub/release-101/gtf/homo_sapiens/Homo_sapiens.GRCh38.101.gtf.gz
* Index the human genome
  * singularity exec -B ${PWD} docker://gemmaker/hisat2:2.1.0-1.1 hisat2-build Homo_sapiens.GRCh38.dna.primary_assembly.fa HG38
* Once the genome has been indexed, make a directory called HG38.genome.Hisat2.indexed and move all the .ht2 files into it
  * Mv *.ht2 HG38.genome.Hisat2.indexed
* In the demo directory, remove the 3 fastq files
* Open the SRA_IDs.txt file in the demo directory, remove what is there, and paste the new SRA IDs
  * For endometriosis:
     * SRR12549333
     * SRR12549334
     * SRR12549335
     * SRR12549336
     * SRR12549337
     * SRR12549338
  * For atypical:
    * SRR12549342
    * SRR12549343
    * SRR12549344
    * SRR12549345
    * SRR12549346
    * SRR12549347
* Configure the nextflow.config file in the GEMmaker directory
  * Open nextflow.config with nano
  * Change local_sample_files from “*_{1,2}.fastq” to “none”.
  * Set method to hisat2
  * Change the index_dir to ./demo/references/HG38.genome.Hisat2.indexed #adjust based on approriate path
  * Change the gtf_file to ./demo/references/Homo_sapiens.GRCh38.101.gtf #adjust baed on appropriate path
* Run GEMmaker
  * nextflow run main.nf -profile standard -with-singularity -with-report -with-timeline
* Once GEMmaker has finished, you should see the GEMs in an output/GEMs directory
* Rename the GEMs to distinguish between endometriosis and atypical
  * ex. GEMmaker.GEM.atypical.FPKM.txt and GEMmaker.GEM.endometriosis.FPKM.txt
* Make a new directory called “GCN” in your working directory (not in the GEMmaker_atypical or GEMmaker_endometriosis)
* Copy the fpkm GEMs from atypical and endometriosis into this GCN directory
* Merge the GEMs
  * paste -d” “ GEMmaker.GEM.atypical.FPKM.txt GEMmaker.GEM.endometriosis.FPKM.txt > merged-atypical-endometriosis-gem.emx.txt
* Normalize the GEMs 
  * Get GEMprep
    * git clone https://github.com/SystemsGenetics/GEMprep.git
  * Activate GEMprep
    * conda activate gemprep
  * Log2 transform the merged GEM
    * python ./GEMprep/bin/normalize.py merged-atypical-endometriosis-gem.txt merged-atypical-endometriosis-gem.log2.txt --log2
  * Quantile normalize
    * python ./GEMprep/bin/normalize.py merged-atypical-endometriosis-gem.log2.txt merged-atypical-endometriosis-gem.log2.quantile.txt --quantile
* Create a tmpfiles directory for Nextflow to write files in
  * export TMPDIR=/home/student/tmpfiles
* Make an input directory in your working directory (in the GCN directory)
* Copy the merged-atypical-endometriosis-gem.emx.txt into this input directory
* Run the workflow
  * nextflow run systemsgenetics/KINC-nf -with-singularity -resume
* Install cytoscape on your computer
  * Instructions can be found at: https://cytoscape.org/
* Once the workflow has finished running, open Cytoscape and drag the GCN network file to visualize.










