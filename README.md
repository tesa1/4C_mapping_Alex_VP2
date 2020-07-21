# 4C mapping: Alex Kojic VP2

A mapping pipeline that maps and filters 4C data.

**Note::::: Run bwa index on your fasta file first. Fasta file must have 'chr' in it.**


In a 4C experiment DNA fragments are ligated to your fragment of interest, which are amplified using an inverse PCR. These fragments need to mapped back to the reference genome. Repetitive fragments need to removed from the analysis. Note that non-covered fragment are also of interest to the analysis, since they signal no interaction at this genomic location.

The analysis pipeline consists of three steps that has the user has to run:

#### 1. Creating a fragment map (note this only needs to be done once because the same REs were used for all 3 Viewpoints)

To create a fragment map for you enzyme combination of choice please run the generate_fragment_map.pl script. For a fragment map for DpnII and Csp6I of the human genome you would use the following command:

```
## on harris:/DATA/t.severson/alex_4c
perl generate_fragment_map.pl ~/resources/hg19_sed.fa GATC CATG gatc_catc_fragment_map/
```

The fragment map will be strored in the directory `gatc_catc_fragment_map/`

#### 2. Identifying the repetitve fragments

We would like to filter the fragment map for repetitive fragments, therefore we will map all the fragments back to genome we selected them from to test whether they are unique or not. For the fragment map we just created will should run the following command:

```
mkdir 59_repeat
perl getRepeats.pl gatc_catc_fragment_map/ GATC 50 ~/resources/hg19_sed.fa 59_repeat/

## note, this will store your data into a folder called '59' in your test_repeat folder. 
## This will break the next script. Add the '59' to the next command. 
```

The results will be placed in the directory 59_repeat/. Note the 59, this is the length of the ligated fragment including the restriction site. Note that for every different sequencing length for you 4C experiment, you will need to create a new repeat map. So if you have a sequence length of 75, a primer of 20ntand a 4nt restriction site, your sequence length should be `75 - 20 + 4 = 59`. 



#### 3. Splitting FASTQ and mapping to the genome (use WZ3989-9_simple_index.txt above as a working example)

The preprocessing of the data is now finished and you can start to map your data to the genome. The only thing you need is an index file, which contains the minimal information of your 4C experiment. Add the 1st restriction enzyme onto the end of your primer sequence. The structure of this file is as follows:

|Experiment name | primer sequence + spacer + RE | path to reference genome | restriction enzyme 1 | restriction enzyme 2 | viewpoint chromosome |
|---------- | ---------- | ----------|----------|----------|----------|
|E2_1h_vp2_replicate1 | TAAAGGGAAGGACAGAATTG**GATC** | /home/t.severson/resources/hg19_sed.fa | GATC | GTAC | chr6 |

Note that the reference should also have bwa index. Also note that the second restriction enzyme is not strictly necessary, but the chromosome id should always be in the 6th column. Given the curre
nt setup it is not possible to mix restriction enzyme combination or reference genomes. If you have multiple genomes or multiple restriction enzyme combinations please create a seperate index file
for each one.

The following command is used to process and map your data with 10 threads.

```
perl mapping_pipeline.pl WZ3989-9_simple_index.txt WZ3989-9 /DATA/t.severson/alex_4c/alex_files/5986_9_WZ3989-9_GATCAG_S13_R1_001.fastq.gz 10 gatc_catc_fragment_map/ 59_repeat/59
```

More detailed information is given in the scripts themselves.

###Requirements:

To run the 4C mapping you will need the following software installed on your machine

1. bwa (bwasw)
2. bedtools (fastaFromBed)
3. Inline::C

