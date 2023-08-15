# STRcount
![](https://img.shields.io/github/license/mmisak/STRcount)
[![trcount Release](https://img.shields.io/github/v/release/mmisak/STRcount)](https://github.com//mmisak/STRcount/releases/)
![](https://img.shields.io/github/repo-size/mmisak/STRcount)

## Overview
<p align="justify">
STRcount is a read mapping-free method to detect differential short tandem repeat (STR) content between different groups of quantitative sequencing data samples. STRcount scans raw reads for STRs using Phobos, optionally filters them and then groups detected repeats and outputs their counts in a countstable, allowing for read count normalization and subsequent identification of differential tandem repeat content between groups of samples. Possible applications include the unbiased detection of STR enrichment in DNA/chromatin profiling sequencing (e.g. ChIP-seq, CUT&Run, CUT&Tag, DIP-seq) or comparison of STR content in whole genome sequencing samples.

Note: STRcount is not affiliated with Phobos or its developer Christoph Mayer.
</p>

## Features
1. STRcount offers unbiased de novo detection of differential short tandem repeats in quantitative NGS data
2. Works independent of reference genomes, raw reads are the only input data
3. Repeats can be grouped by repeat length and/or perfection prior to differential comparison
4. Output can be used to generate PCA plots, sample correlation heatmaps, volcano plots and more

## Requirements
- Linux - STRcount was tested on Ubuntu 18.04. Windows or macOS systems might work but were not tested thus far
- Phobos binary - STRcount was tested with the binary `phobos_64_libstdc++6` of the package `phobos-v3.3.12-linux`. Phobos is freely available for academic research and can be downloaded here: https://www.ruhr-uni-bochum.de/spezzoo/cm/cm_phobos_download.htm
- Python 3.2 or newer - STRcount was tested on Python 3.9.7

## Quick start
### Running STRcount processrepeats
Run STRcount's processrepeats function on all samples of your sequencing data in FASTQ(.gz) or FASTA(.gz) format:
```
python STRcount.py processrepeats SAMPLE OUTPUT_FOLDER PHOBOS_BINARY [addtional parameters]
```

**Example**:
```
python STRcount.py processrepeats sample1.fastq.gz ./strcount_out/ /home/phobos/phobos-v3.3.12-linux/bin/phobos_64_libstdc++6 --grouping 'perfection:[0,100)[100,100] length:[0,40),[40,80),[80,inf]' --processes 40
```

In this example, we are running the `processrepeats` function on a sample called sample1.fastq.gz using 40 processes and grouping the detected repeats by perfection and length. The output is written into a folder called `strcount_out` in the current working directiory, the folder will be created if not already present. Perfection grouping is done here by splitting the repeats in 2 groups, imperfect (perfection between 0 and lower than 100, as indicated by the brackets: `[0,100)`, square brackets indicate an inclusive range, while round brackets indicate an exclusive range) and perfect (`[100,100]`) as well as 3 length groups with the last group (`[80,inf]`) not having an upper length range limit.

**Important**:
- STRcount runs much faster on machines with many cores. By default, STRcount will determine the available number of cores by itself and use all of them. On a workstation PC, it can make sense to limit the number of cores by setting `--processes` to a lower number than the actually available cores.
- When using grouping, in most cases it will make sense to only use grouping ranges that a are not overlapping as illustrated in the above example (`length:[0,40),[40,80),[80,inf]`). Note the square and roud brackets indicating inclusive and exclusive ranges, respecitvely. In case of overlapping ranges (e.g. `length:[0,40],[0,80],[80,inf]`), a repeat of length 40bp or 80bp would be grouped into two diffent groups and increase the repeat count of both groups by 1.
- STRcount classifies STR units by their lexicographically minimal string rotation, e.g. the repeat unit of a CAGCAGCAGCAGCAGC repeat is AGC, the repeat unit of its reverse compliment is CTG
- Depending on the analysis, you might want to set the `--groupingmotif` parameter accordingly. In a non strand-aware sequencing (common WGS, ChIP-seq, CUT&Run, CUT&Tag, ..), it can make sense to set the parameter to `combine` and combine reverse complements of repeats (e.g. AGC/CTG) into a single group as these techniques commonly do not discriminate between strands. If you do not wish to combine reverse complements of repeats and group repeats by the repeat as it is detected in the repeat (e.g. for a forward-stranded sequencing), set the parameter to `detected`, to group by the reverse complement of the retected repeat (e.g. for a reversely-stranded sequencing), set the parameter to `rc`.
- If you wish to generate the repeatinfo.txt file to get detailled information (such as perfection, mismatches in comparison to a perfect repeat and more) on every single detected repeat, include `i` (for `.repeatinfo.txt`) or `g` (for `.repeatinfo.txt.gz`) in the `--outputtype` parameter. Warning: This file is usually relatively large. In our tests, its file size was usually comparable to the size of the raw reads in FASTQ format. 
  
### Running STRcount summarizecounts
After obtaining `.countstable.txt` files for all of our samples, we summarize the results into a count matrix that can be used for downstream analysis:
```
python STRcount.py summarizecounts SAMPLES [...] OUTPUT_FILE
```

**Example**:
```
python STRcount.py summarizecounts sample1.countstable.txt sample2.countstable.txt control1.countstable.txt control2.countstable.txt experiment.countmatrix.txt
```

### Read count normalization and differential repeat content analysis
Last, we use our read counts for a differential read count analysis. For this, analyses using edgeR, DESeq2, limma voom or Wilcoxon rank sum test are suitable.

**Example (using EdgeR)**:

## Full options
### STRcount processrepeats function
```
STRcount.py processrepeats [-h] [--outputprefix OUTPUT_PREFIX] [--outputtype OUTPUT_TYPE]
                                               [--processes PROCESSES_NUMBER] [--grouping GROUPING_SETTING]
                                               [--groupingmotif {detected,rc,combine}] [--minperfection MIN_PERFECTION]
                                               [--maxperfection MAX_PERFECTION] [--minrepeatlength MIN_REP_REGION_LENGTH]
                                               [--maxrepeatlength MAX_REP_REGION_LENGTH] [--minunitsize MIN_UNIT_SIZE]
                                               [--maxunitsize MAX_UNIT_SIZE] [--minrepeatnumber MIN_REP_NUMBER]
                                               [--maxrepeatnumber MAX_REP_NUMBER] [--multirepreads MULTI_REP_READS_SETTING]
                                               [--readwhitelist READ_WHITELIST_FILE] [--readblacklist READ_BLACKLIST_FILE]
                                               [--readblocksize READ_BLOCK_SIZE]
                                               inputpath outputdirectory phobospath

positional arguments:
  inputpath             path to sequencing data file in fasta(.gz) or fastq(.gz) format
  outputdirectory       directory where the putput will be written to
  phobospath            path to Phobos executable

optional arguments:
  -h, --help            show this help message and exit
  --outputprefix OUTPUT_PREFIX
                        prefix of output files, prefix will be taken from input file, if empty string (default: )
  --outputtype OUTPUT_TYPE
                        output to generate, countstable.txt ('c'), repeatinfo.txt ('i'), repeatinfo.txt.gz ('g'), concatenate
                        the letters for multiple outputs, e.g. 'cg' for countstable.txt and repeatinfo.txt.gz (default: c)
  --processes PROCESSES_NUMBER
                        number of parallel processes to be used, to automatically set to maximum number of available logical
                        cores, use 'auto' (default: auto)
  --grouping GROUPING_SETTING
                        repeat grouping settings, example: 'perfection:[0,100)[100,100] length:[0,30)[30,inf]', if 'None',
                        repeats will be only grouped by their motif (default: None)
  --groupingmotif {detected,rc,combine}
                        motif to report for grouping, 'detected' to report the detected motif, 'rc' for it's reverse
                        complement, 'combine' (combines forward and reverse complement), all motifs are reported as their
                        lexicographically minimal string rotation (default: detected)')
  --minperfection MIN_PERFECTION
                        minimum perfection of a repeat to be considered (default: 0)
  --maxperfection MAX_PERFECTION
                        maximum perfection of a repeat to be considered (default: 100)
  --minrepeatlength MIN_REP_REGION_LENGTH
                        minimum repeat region length for a repeat to be considered (default: 0)
  --maxrepeatlength MAX_REP_REGION_LENGTH
                        maximum repeat region length for a repeat to be considered (for infinite, set value to 'inf')
                        (default: inf)
  --minunitsize MIN_UNIT_SIZE
                        minimum repeat unit size for a repeat to be considered (default: 0)
  --maxunitsize MAX_UNIT_SIZE
                        maximum repeat unit size for a repeat to be considered (for infinite, set value to 'inf') (default:
                        inf)
  --minrepeatnumber MIN_REP_NUMBER
                        minimum number of repeat units for a repeat to be considered (default: 0)
  --maxrepeatnumber MAX_REP_NUMBER
                        maximum number of repeat units for a repeat to be considered (default: inf)
  --multirepreads MULTI_REP_READS_SETTING
                        which repeat to consider in case of reads with multiple repeats (after other filters have been
                        applied), either 'all' (consider all repeats for each read), 'none' (ignore multi repeat reads),
                        'longest' (only consider the longest repeat) or 'unique_longest' (for each unique repeat unit, only
                        consider the longest) (default: all)
  --readwhitelist READ_WHITELIST_FILE
                        path to list of readnames that will not be filtered out, the rest is filtered (default: None)
  --readblacklist READ_BLACKLIST_FILE
                        path to list of readnames that will be filtered out, the rest is kept (default: None)
  --readblocksize READ_BLOCK_SIZE
                        approximate number of lines that are analyzed at once in a (parallel) process (default: 50000)
```
### STRcount summarizecounts function
```
strcount2.6.2_rewrite.py summarizecounts [-h] [--samplenames SAMPLE_NAMES [SAMPLE_NAMES ...]]
                                                inputpaths [inputpaths ...] outputfile

positional arguments:
  inputpaths            countstable.txt files to be summarized into a count matrix
  outputfile            path to output count matrix.

optional arguments:
  -h, --help            show this help message and exit
  --samplenames SAMPLE_NAMES [SAMPLE_NAMES ...]
                        list of sample names to be used in the resulting header in the same order as input files. If not set,
                        input file names will be used (default: None)')
```
