# kmeleon

A pipeline for k-mer count analysis of genomic diversity.

#### Dependencies

One of the kmeleon scripts, kmeleon_extract.py uses the Python API [pysam](http://pysam.readthedocs.io/en/latest/api.html) to read the SAM/BAM mappings file.

#### Running kmeleon

There are several steps to run a complete kmeleon analysis:
* [Extract the kmers from the mappings: kmeleon_extract.py](https://github.com/eead-csic-compbio/kmeleon#1st-extract-the-kmers-from-the-mappings)
* [Count the number of kmers at each genomic position: kmeleon_count.py](https://github.com/eead-csic-compbio/kmeleon#2nd-count-the-number-of-kmers-at-each-genomic-position)

There are also other tools to manage the results:
* [Join the kmer counts of all samples in a single table: kmeleon_table.py](https://github.com/eead-csic-compbio/kmeleon#3rd-join-the-kmer-counts-of-all-samples-in-a-single-table)
* [Translate the position-based kmer counts to intervals of constant kmer count](https://github.com/eead-csic-compbio/kmeleon#4th-translate-the-position-based-kmer-counts-to-intervals-of-constant-kmer-count)

## 1st) Extract the kmers from the mappings

This is done by running the script kmeleon_extract.py and the next parameters:

`kmeleon_extract.py target mappings kmer_size flush depths`

- target: for example the chromosome name or number.
- mappings: the file with mappings to process. A SAM or BAM file would be great.
- kmer_size: a number which indicates the size of sequences to search for. 50 for example.
- flush: a number which allows controlling how much memory is used by kmeleon while reading the SAM/BAM file. 10000 or 20000 is ok in general.
- depths: (Optional argument) if the word 'depths' is given, the depth of each kmer found will be also reported.

For example:

`kmeleon_extract.py chr1 mappings.sam 50 10000 depths`

it generates a file with 3 columns, as shown in its header:

```
@ Position	MD_Z	count
2917	51	1
2918	51	1
2919	51	1
2920	51	1
2921	51	1
2922	51	1
```

or

`kmeleon_extract.py chr1 mappings.sam 50 10000`

to obtain only the first 2 columns (and a lighter file):

```
@ Position	MD_Z
2917	51
2918	51
2919	51
2920	51
2921	51
2922	51
```

* @ is a symbol used to differentiate the header from the other rows
* Position: the position within the target (chromosome)
* MD_Z: a kmer found in such position, in SAM/BAM file MD_Z field format
* count: the times that this kmer has been observed at this position.

## 2nd) Count the number of kmers at each genomic position

This is done by running the script kmeleon_count.py and the next parameters:

`kmeleon_count.py kmers_file min_depth`

- kmers_file: a file with kmers by position (the one reported by kmeleon extract with the 'depths' option, for example). \
It could be in '.gz' extension or not.
- min_depth: a number which is the minimum depth of a kmer o be considered and counted. For instance 4.

For example:

`kmeleon_count.py kmer_chr1_sampleA 4`

it generates a file with 2 columns (without header):

```
11865	1
11866	1
11867	1
11868	1
11869	1
11870	1
```

* 1st field: the position within the target (chromosome)
* 2nd field: the number of different kmers found in that position

## 3rd) Join the kmer counts of all samples in a single table

This is done by running the script kmeleon_table.py and the next parameters:

`kmeleon_table.py samples_list_file target counts_DIR`

- samples_list_file: a file with a list of samples to be included in the output table. The format of the file is just a sample name in each row.
- target: chromosome or target name or number. For example, chr1.
- counts_DIR: a path to the directory  where the files with counts (from kmeleon count) can be found. Note that the files in this folder must be in '.gz' format, \
and will have the next filename format:

`counts_DIR/sample_name.target.counts.out.gz`

For example:

`kmeleon_table.py my_samples chr1 counts`

it generates a file with a header and n+1 columns, where n = number of samples.:

```
Pos	sample1	sample2	...	sampleN
11865   1	1	...	1
11866   1	2	...	1
11867   1	2	...	1
11868   3	2	...	1
11869   3	1	...	0
11870   3	1	...	0
```

* Pos: the position within the target (chromosome)
* sample1...sampleN: the number of different kmers found in that position for a given sample

## 4th) Translate the position-based kmer counts to intervals of constant kmer count

This is done by running the script kmeleon_intervals.sh and the next parameters:

`kmeleon_intervals.sh sample target counts_DIR minspan kmercount binary`

- sample: the sample to process.
- target: for example the chromosome name or number.
- counts_DIR: a path to the directory  where the files with counts (from kmeleon count) can be found. Note that the files in this folder must be in '.gz' format, \
and will have the next filename format:
`counts_DIR/sample_name.target.counts.out.gz`
- minspan: minimum length (consecutive genome nucleotides) of the resulting interval to be reported as such. E.g. 50
- kmercount: whether report all intervals (all), only those with kmercount=1 (uniq), or those with kmercount>1 (multiple)
- binary: whether the raw count value is used to compute intervals (no), or just differentiating kmercount=1 of kmercount>1 (yes)

For example:

`kmeleon_intervals.sh sample_name chr1H_part1 counts 50 multiple yes`

will report all the intervals (kmercount=all), joining all positions with kmercount>1 in intervals (binary=yes), and at least a length of 50 nts (minspan=50)

it generates a file in BED-like format without header and with 4 columns:

```
chr1H_part1	41868	41871	0
chr1H_part1	41911	41925	0
chr1H_part1	41977	41982	0
chr1H_part1	41985	42023	0
chr1H_part1	42054	42056	0
chr1H_part1	42169	42231	0
chr1H_part1	42274	42348	0
chr1H_part1	42349	42375	1
chr1H_part1	42376	42390	0
chr1H_part1	42391	42420	1
```

* 1st column is the target name@ is a symbol used to differentiate the header from the other rows
* 2nd column is the starting position of the interval. Note that this field is 0-based, as it follows the BED format
* 3rd column is the ending position of the interval
* 4th column shows whether kmercount=1 (0) or kmercount>1 (1)

NOTE that the values of the 4th column would show the actual kmercount with the option binary=no
