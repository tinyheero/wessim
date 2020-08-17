## Preamble

This is a fork of https://github.com/sak042/Wessim. As the original repository 
appears to be abandoned, the following new features have been added to this 
fork:

* Ability to call Wessim from outside of repository folder
* Option to add a prefix for simulated read names (`--read-name-prefix`)
    + This is useful for situations where you want to track which reference 
        (e.g. haplotype) a read came from.
* Bug fix for paired-end mode when using the "ideal target" approach.
* Insert sizes are drawn from a normal distributon
* The original `getRegionVector` function has now been moved into its own 
    script (`get_region_vector.py`). 
* Cleaned up the original repository (e.g. removed the `Wessim_ver_1.0` folder)

**Enhancements to the code have been done for the "ideal target" mode (i.e.
`Wessim1.py`. If you are looking to run the probe hybridization module, you 
should refer back to the original repository (https://github.com/sak042/Wessim)**

## Introduction

Wessim is a simulator for a targeted resequencing as generally known as 
exome sequencing. Wessim basically generates a set of **artificial** DNA 
fragments for next generation sequencing (NGS) read simulation. In the targeted 
resequencing, we constraint the genomic regions that are used to generated DNA 
fragments to be only a part of the entire genome; they are usually exons and/or 
a few introns and untranslated regions (UTRs).

## Installation and Requirements

The following programs are required to run Wessim (in ideal target mode) or to 
prepare input files:

* Python (2.7): Wessim was written for Python 2.7. It will not work with 
    Python 3. You'll need these packages:
    + `pysam`
    + `numpy`
* `faToTwoBit`: go to http://hgdownload.cse.ucsc.edu/admin/exe/ and download 
    faToTwoBit. This is required to convert your FASTA file to .2bit 
* `samtools`: samtools is needed to index your sample genome FASTA file (samtools faidx).
* GemSim error models: Wessim uses GemSim's empirical error models for NGS 
    read generation. Go to GemSim's project page 
    (http://sourceforge.net/projects/gemsim/) to download GemSim. You will find 
    several model files (e.g. `ill100v4_p.gzip`) under models directory. Save 
    them and remember their location.
 
## How to Run

**As mentioned at the top, only instructions for running Wessim in ideal
target mode are provided. Please see the original repository for running Wessim
in probe hybridization mode**

Before you can run Wessim, you need to ensure that your genome FASTA file is
indexed. You can do this by running:

```bash
samtools faidx reference.fa
faToTwoBit reference.fa reference.2bit
```

Once that is complete, you need to provide Wessim with a target [BED file](https://genome.ucsc.edu/FAQ/FAQformat.html) 
to indicate where fragments/reads should be generated from. For example:

```
chr1	100	1000
chr9	1000	5000
```

By default, wessim will simulate from these target regions proportional to the
length of the region. if you want to have a bit more control over this, you can
specify a "relative capture efficiency" (rce) as the 5th column in the bed file.
For example:

```
chr1	100	1000	chr1:100-2000	2
chr9	1000	5000	chr9:1000-5000	1
```

The rce value of a region represents the relative weight that wessim should be 
sampling reads from. in this case, it was sample twice as many reads from 
region `chr1:100-1000` than `chr9:1000-5000`. 

## Example usage

As an example of how to use wessim, we will generate 10,000 reads of 100 bps 
long from two regions of hg19. First, let's grab the input data we need:

```bash
# Get the GemSim error model
wget \
        https://sourceforge.net/projects/gemsim/files/GemSIM_v1.6.tar.gz/download \
        -O GemSIM_v1.6.tar.gz
tar -xzvf GemSIM_v1.6.tar.gz

# Download hg19 
wget http://hgdownload.cse.ucsc.edu/goldenpath/hg19/bigZips/hg19.fa.gz 
wget http://hgdownload.cse.ucsc.edu/goldenpath/hg19/bigZips/hg19.2bit
gunzip hg19.fa.gz
```

Now we run Wessim:

```bash
# Generate the reference files needed to run Wessim1.py
./get_region_vector.py \
        --fasta-file hg19.fa \
        --target-bed-file example_data/example.bed \
        --target-fasta-file target_reference.fa \
        --target-abd-file target_reference.abd

# Run Wessim1 in ideal target mode
./Wessim1.py \
        --target-fasta-file target_reference.fa \
        --target-abd-file target_reference.abd
        -n 10000 \
        -l 100 \
        -M GemSIM_v1.6/models/ill100v4_p.gzip \
        -z \
        -o result \
        -t 4
```

This will generate `result.fastq.gz` (single-end mode / gzip compressed) using 
4 threads (CPU cores). 

If you want to try it with RCE values, use the `example_data/example_rce.bed`
BED file.

## Contact

* Fong Chun Chan <fongchun@alumni.ubc.ca>
