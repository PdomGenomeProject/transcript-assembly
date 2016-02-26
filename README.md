# Transcript assembly

This data set is part of the [*Polistes dominula* genome project][pdomproj], and details the assembly of the *P. dominula* transcriptome, as described in ([Standage *et al.*, *Molecular Ecology*, 2016][ref]).
Included in this data set is the final transcript assembly itself, as well as documentation providing complete disclosure of the assembly procedure.

## Synopsis

RNA-Seq reads were processed using [Trimmomatic][] version [0.22][] to remove adapter contamination and low-confidence base calls, and the processed reads were assembled using [Trinity][] version [r20131110][].
The assembled transcripts were then processed with [mRNAmarkup][] version [10-3-2013][] to remove contaminants and correct erroneously assembled chimeric transcripts.

## Data access

Raw Illumina data is available from the NCBI Sequence Read Archive under the accession number [PRJNA291219][sra].
The `GetTranscriptomeSRA.make` script automates the process of downloading these data files and converting them from SRA format to Fastq format.
This script in turn depends on the `fastq-dump` command included in the [SRA toolkit][sratk] binary distribution.

```bash
./GetTranscriptomeSRA.make
```

## Procedure

### Short read quality control

First, we designated the number of processors available, and provided the path of the `trimmomatic-0.22.jar` file distributed with the Trimmomatic source code distribution.

```bash
NumProcs=16
TrimJar=/usr/local/src/Trimmomatic-0.22/trimmomatic-0.22.jar
```

We then applied the following filters to each read pair (see `run-trim.sh` for details).

  - remove adapter contamination
  - remove any nucleotides at either end of the read whose quality score is below 3
  - trim the read once the average quality in a 5bp sliding window falls below 20
  - discard any reads which, after processing, fall below the 40bp length threshold

```bash
for caste in q w
do
  for rep in {1..6}
  do
    sample=${caste}${rep}
    ./run-trim.sh $sample $TrimJar $NumProcs
  done
done
```

### Assembly with Trinity

Trinity requires a pair of input files for paired-end data.

```bash
cat pdom-rnaseq-*-trim-1.fq > pdom-rnaseq-all-trim-1.fq
cat pdom-rnaseq-*-trim-2.fq > pdom-rnaseq-all-trim-2.fq
```

We then executed the Trinity assember using the `--CuffFly` algorithm.

```bash
Trinity.pl --seqType fq \
           --JM 100G \
           --bflyHeapSpaceMax 50G \
           --output pdom-trinity \
           --CPU $NumProcs \
           --left pdom-rnaseq-all-trim-1.fq \
           --right pdom-rnaseq-all-trim-2.fq \
           --full_cleanup \
           --jaccard_clip \
           --CuffFly
```

### Post-processing with mRNAmarkup

Contaminant, reference protein, and miRNA databases were collected as described in the mRNAmarkup documentation (`db/0README` and `db/0README-hy`).
The mRNAmarkup procedure was then run on the Trinity output.
Be sure to edit the `mRNAmarkup.conf` file with the correct paths to the databases.

```bash
mRNAmarkup -c mRNAmarkup.conf \
           -i pdom-trinity/Trinity.fasta \
           -o output-mRNAmarkup
```

------

[![Creative Commons License](https://i.creativecommons.org/l/by/4.0/88x31.png)][ccby4]  
This work is licensed under a [Creative Commons Attribution 4.0 International License][ccby4].


[pdomproj]: https://github.com/PdomGenomeProject
[ref]: http://dx.doi.org/10.1111/mec.13578
[Trimmomatic]: http://www.usadellab.org/cms/index.php?page=trimmomatic
[0.22]: http://www.usadellab.org/cms/uploads/supplementary/Trimmomatic/Trimmomatic-0.22.zip
[Trinity]: http://trinityrnaseq.sourceforge.net/
[r20131110]: http://downloads.sourceforge.net/project/trinityrnaseq/trinityrnaseq_r20131110.tar.gz
[mRNAmarkup]: http://brendelgroup.org/bioinformatics2go/mRNAmarkup.php
[10-3-2013]: http://www.brendelgroup.org/bioinformatics2go/Download/mRNAmarkup-10-3-2013.tar.gz
[sra]: http://www.ncbi.nlm.nih.gov/sra/?term=PRJNA291219
[sratk]: http://www.ncbi.nlm.nih.gov/Traces/sra/?view=software
[ccby4]: http://creativecommons.org/licenses/by/4.0/
