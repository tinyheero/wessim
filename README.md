# Preamble

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

# Introduction

**Wessim** is a simulator for a targeted resequencing as generally known as 
exome sequencing. Wessim basically generates a set of *artificial* DNA fragments 
for next generation sequencing (NGS) read simulation. In the targeted 
resequencing, we constraint the genomic regions that are used to generated DNA 
fragments to be only a part of the entire genome; they are usually exons and/or 
a few introns and untranslated regions (UTRs).

# Install Wessim

Download Wessim using the links in this page, or go to https://github.com/sak042/Wessim   
To run Wessim, Python 2.7 or later is required. To install Python, go to http://python.org/

## Requirements

Depending on which version f

The following programs are required to run Wessim or to prepare input files:

* **pysam** library: go to http://code.google.com/p/pysam/ to install pysam
* **numpy** library: go to http://numpy.scipy.org/ to install numpy
* **gfServer** and **gfClient**: In probe hybridization mode, Wessim runs more than 100,000 queries against the reference genome. This essentially requires a local blat server. gfServer and gfClient are pre-compiled programs for establishing private blat server on your computer. go to http://hgdownload.cse.ucsc.edu/admin/exe/ to download gfServer and gfClient (and set your local path to access the two programs anywhere). For more details about the tools, please refer to http://genome.ucsc.edu/FAQ/FAQblat.html#blat5
* **faToTwoBit**: go to http://hgdownload.cse.ucsc.edu/admin/exe/ and download faToTwoBit. This is required to convert your FASTA file to .2bit 
* **samtools**: samtools is needed to index your sample genome FASTA file (samtools faidx).
* **GemSim** error models: Wessim uses GemSim's empirical error models for NGS read generation. Go to GemSim's project page (http://sourceforge.net/projects/gemsim/) to download GemSim. You will find several model files (e.g. ill100v4_p.gzip) under 'models' directory. Save them and remember their location.
 
# Preparing Input Files 

Wessim requires two major inputs. One is the sample genome sequence, and the 
other is the target region information.

## Sample genome sequence

This is a FASTA file (e.g. ref.fa). You will need to index the file and generate 
.2bit:

```bash
samtools faidx ref.fa
faToTwoBit ref.fa ref.2bit
```

## Target region information

Target regions can be specified by two different ways depending on how you are
running Wessim.

1. **Ideal targets**: In ideal target mode, you will provide a list of genomic 
    coordinates in a BED file (e.g. chr1   798833 799125). Ideal targets of major 
    exome capture platforms are freely available from vendor's website. For 
    Agilent's SureSelect platforms, go to https://earray.chem.agilent.com/suredesign/. 
    You must register at their site. After logging in, go to Find Designs and select 
    Agilent Catalog at the menu tab. You will be able to download all information of 
    currently available platforms including ideal target BED files and probe 
    sequence text files. For NimbleGen's SeqCap go to 
    http://www.nimblegen.com/products/seqcap/index.html and find BED files under 
    Design and Annotation Files. 

1. **Probe sequences**: Probe sequences are available for SureSelect platforms 
    in the SureDesign homepage (https://earray.chem.agilent.com/suredesign/) 
    (see above). Usually those files are named "[platform]_probe.txt"

The probe

# Running Wessim

There are two modes you can run Wessim in:

1. Ideal target mode
1. Probe hybridization mode:
    + For running this mode, you should refer back to the original repository. 
        A lot of the code changes to the code in this repository was designed
        for the ideal target mode. This README will not talk about this mode
        any further.

## Ideal Target Mode

To run ideal target mode:

```bash
# Generate the reference files needed to run Wessim1.py
./get_region_vector.py \
    --fasta-file reference.fa \
    --target-bed-file target.bed \
    --target-fasta-file target_reference.fa \
    --target-abd-file target_reference.abd

# Run Wessim1 in ideal target mode
./Wessim1.py \
    --target-fasta-file target_reference.fa \
    --target-abd-file target_reference.abd
    -n 1000000 \
    -l 100 \
    -M model.gzip \
    -z \
    -o result \
    -t 4
```

This will generate *result.fastq.gz* (single-end mode / gzip compressed) using 4 threads (CPU cores).

# Contact

* Fong Chun Chan <fongchun@alumni.ubc.ca>
