# CodFreq

**Cod**on **Freq**uency Table Format

The HIVDB Sequence Reads Interpretation Program accepts a codon frequency table
that stores in the **CodFreq** format. The CodFreq format consists of five
columns:

1. gene (`PR`, `RT`, or `IN`);
2. position;
3. total number of reads of this position;
4. codon nucleotide triplet; and
5. total number of reads of this codon.


## Examples

[This repository](https://github.com/hivdb/codfreq-sra-hiv) contains CodFreq files
generated from publicly available SRA sequences. We have also included three
selected files from studies that utilize Illumina sequencing. To analyze these
files, first download one or more CodFreq example files. Then, submit them to the
[HIVDB Interpretation Program](https://hivdb.stanford.edu/hivdb/by-reads/) for
analysis.

- [SRR4071760](https://github.com/hivdb/codfreq-sra-hiv/blob/main/codfreqs/SRR4071760.codfreq.gz)
- [SRR6937100](https://github.com/hivdb/codfreq-sra-hiv/blob/main/codfreqs/SRR6937100.codfreq.gz)
- [DRR030302](https://github.com/hivdb/codfreq-sra-hiv/blob/main/codfreqs/DRR030302.codfreq.gz)

<!--
## Create `.codfreq` file from `.sam`/`.bam` file

The `.sam` or `.bam` files are the alignment output from many multiple sequence alignment tools such as Bowtie2, BWA, SNAP, etc.

1. Install Docker CE (https://docs.docker.com/install/).

2. Download script:

   ```bash
   sudo curl -sL https://raw.githubusercontent.com/hivdb/codfreq/master/bin/sam2codfreq-docker -o /usr/local/bin/sam2codfreq
   sudo chmod +x /usr/local/bin/sam2codfreq
   ```

3. Use following command to process FASTQ files and generate CodFreq files.

   ```bash
   sam2codfreq /path/to/folders/containing/sam-bam/files
   ```
   The script will automatically find every file named with an extension of `.sam` or `.bam`, then extract the codon freqency
   table into `.codfreq` file.
-->

## Create `.codfreq` file from `.fastq`/`.fastq.gz` file

1. Install Docker CE (https://docs.docker.com/install/).

2. Download script:

   ```bash
   sudo curl -sL https://raw.githubusercontent.com/hivdb/codfreq/main/bin-wrapper/align-all-docker -o /usr/local/bin/fastq2codfreq
   sudo chmod +x /usr/local/bin/fastq2codfreq
   ```

3. Download alignment profiles:

   ```bash
   mkdir profiles
   curl -sL https://raw.githubusercontent.com/hivdb/codfreq/main/profiles/HIV1.json -o profiles/HIV1.json
   curl -sL https://raw.githubusercontent.com/hivdb/codfreq/main/profiles/SARS2.json -o profiles/SARS2.json
   ```

4. Use following command to process FASTQ files and generate CodFreq files.

   ```bash
   fastq2codfreq -r profiles/HIV1.json -d path/to/fastq/folders
   ```

   The script will automatically find every file named with an extension of
   `.fastq`, align them to `.sam` file and then extract the codon freqency table
   into `.codfreq` file.
   
   The above command is adequate for most case of both paired or unpaired FASTQ
   files generated by Illumina with the filename pattern looks like
   `*_L001_R1_001.fastq.gz` and `*_L001_R1_002.fastq.gz`. However, if your FASTQ
   files are in other naming convention, please read [Advanced usages § Manually
   pairing FASTQ files](#manually-pairing-fastq-files).

Note: the `fastq2codfreq` script can only be executed in an Unix-like system. If you are using Microsoft Windows 10,
you need to install the [Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/install-win10) to
use this script.

### Offline usage

The `fastq2codfreq` command can be used offline, although the usage is slightly
different from the above description. Followings are the differences:

- Docker's installation package, the `fastq2codfreq` script and the alignment
  profiles can be transfered to the offline server using a portable drive.
- Docker image used by `fastq2codfreq` can be downloaded into a binary file, and
  transfer to the offline server using a portable drive.
  ```bash
  # Run this command on a computer with Internet access
  docker save hivdb/codfreq-runner:latest | gzip > codfreq-runner.tar.gz
  
  # Run this command on the offline server
  docker load < codfreq-runner.tar.gz
  ```
- The auto-update option of `fastq2codfreq` should also be disabled with
argument `-s`:
  ```bash
  fastq2codfreq -s -r profiles/HIV1.json -d path/to/fastq/folders
  ```

### Advanced usages
#### Disable auto-pairing FASTQ files
A flag argument `-m` can be added to `fastq2codfreq` command to dissable
auto-pairing FASTQ files.

```bash
fastq2codfreq -m -r profiles/HIV1.json -d path/to/fastq/folders
```

#### Manually pairing FASTQ files
With paired FASTQ files, a single CodFreq file will be generated by the process.
The program will try to match the FASTQ files with similar names as paired FASTQ
files. To change this behavior, a `pairinfo.json` file can be supplied under the
same folder that includes FASTQ files. We have provided an example file at
[`examples/pairinfo.json`](https://github.com/hivdb/codfreq/tree/main/examples/pairinfo.json).

#### Customize fastp options
Program [fastp](https://github.com/OpenGene/fastp) is by default used to trim
adapters, filter low quality regions and reads which are too short.
[`examples/fastp-config.json`](https://github.com/hivdb/codfreq/tree/main/examples/fastp-config.json)
listed all fastp options supported by this pipeline. Please refer to [fastp's
documentation](https://github.com/OpenGene/fastp#all-options) for the usage and
explanation of these options.

To apply your customized settings, make a `fastp-config.json` file and save it
under the same folder that includes FASTQ files. You can also disable adapter
trimming, low phred quality filtering or length filtering by set the
corresponding disabling flags to `true`.

### Primer trimming - FASTA
CodFreq pipeline supports trimming FASTA format primer sequences by using
[cutadapt](https://cutadapt.readthedocs.io/en/v4.1/guide.html).
[`examples/cutadapt-config.json`](https://github.com/hivdb/codfreq/tree/main/examples/cutadapt-config.json)
listed all cutadapt options supported by this pipeline. Please refer to
[cutadapt's
reference guide](https://cutadapt.readthedocs.io/en/v4.1/reference.html) for the
usage and explanation of these options.

Three type of optional FASTA primer files can be supplied under the same folder
that includes the FASTQ files: `primers3.fa`, `primers5.fa` and `primers53.fa`
which corresponding to the “3’ adapters”, “5’ adapters”, and “5’ or 3’ adapters”
described in [cutadapt's user
guide](https://cutadapt.readthedocs.io/en/v4.1/guide.html#overview-of-adapter-types).

To enable primer trimming (FASTA), you must make a valid `cutadapt-config.json`
file under the same folder that includes FASTQ files.

### Primer trimming - BED
CodFreq pipeline supports trimming BED format primer locations by using
[ivar](https://andersen-lab.github.io/ivar/html/manualpage.html).
[`examples/ivar-trim-config.json`](https://github.com/hivdb/codfreq/tree/main/examples/ivar-trim-config.json)
listed all `ivar trim` options supported by this pipeline. Please refer to
[ivar's
manual](https://andersen-lab.github.io/ivar/html/manualpage.html) for the
usage and explanation of these options.

A BED primer file can be supplied under the same folder that includes the FASTQ
files: `primers.bed` (example:
[`examples/primers.bed`](https://github.com/hivdb/codfreq/tree/main/examples/primers.bed)).
ivar requires a BED6 format which is a tab-delimited file include following six
columns (no header): reference, start, end, name, score, and strand. We have
reviewed ivar 4.1 source code and have confirmed that only four columns - start,
end, name, and strand are used by ivar. The other two (reference and score) can
be just supplied in any values for completing the BED6 format.

To enable primer trimming (BED), you must make a valid `ivar-trim-config.json` file
under the same folder that includes FASTQ files.

<!--

You can specify your FASTQ file naming convention by passing `-p <PATTERN>`, `-r <REPLACE>`, `-1 <PAIR1_SUFFIX>` and 
`-2 <PAIR2_SUFFIX>` parameters to `fastq2codfreq`. Noted `-p` and `-r` are paired regular expression replacement.
For example, if your FASTQ files are downloaded from Sequence Reads Archive (SRA), their naming convention would be
`*_1.fastq.gz` and `*_2.fastq.gz`. Here is the command:

```bash
fastq2codfreq -p '(.+)_[12]\.fastq\.gz$' -r '\1' -1 '_1.fastq.gz' -2 '_2.fastq.gz' /path/to/folders/containing/fastq/files
```

#### Specify concurrency rate
You can also specify concurrency rate (how many CPUs you want to use) by passing `-n <NTHREADS>` to `fastq2codfreq`.
By default, the script will use fomula `FLOOR(TOTAL_CPUS * 19 / 20)` to calculate the `NTHREADS` value.

```bash
# Use at most 5 CPUs
fastq2codfreq -n 5 /path/to/folders/containing/fastq/files
```

Note: if you are using Docker on Windows or MacOS, the upper limit of `NTHREADS` depends on the number of CPUs allocated
for the Docker software. Please check these general answers from Stackoverflow:

- Windows: https://stackoverflow.com/a/56583203/2644759
- MacOS: https://stackoverflow.com/a/39720010/2644759
-->

## Other tools

### Consolidate codon frequency table to amino acid freqency table

A script using only the standard Python library is provided to consolidate a
codon frequency table (.codfreq or .codfreq.gz file) into an amino acid
frequency table (.aafreq.csv file). The script merges rows of codons that can be
translated into the same amino acid.

This script requires Python 3.9 or higher version to be installed. This required
Python runtime is included in the latest version of MacOS and most Linux
releases. To install the latest Python version, please follow the [official
website](https://www.python.org/downloads/).

To use this script:

1. Download the script:

   ```bash
   sudo curl -sL https://raw.githubusercontent.com/hivdb/codfreq/main/scripts/codfreq2aafreq.py -o /usr/local/bin/codfreq2aafreq
   sudo chmod +x /usr/local/bin/codfreq2aafreq
   ```

2. Run the script:

   ```bash
   codfreq2aafreq dir/to/read/codfreqs dir/to/write/aafreqs
   ```
