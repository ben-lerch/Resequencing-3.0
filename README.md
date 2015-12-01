# Resequencing: PacBio Consensus Sequence Generation and Variant Calling

The PacBio resequencing application leverages the exceptionally long read lengths generated by the SMRT Sequencing technology to more effectively resolve the true nature of complex repetitive genomic regions as well as shed light on overall structural variation. The random stochastic nature of errors in the raw polymerase reads is surmounted with sufficient coverage to generate a highly accurate consensus sequence. Any variation from the reference is captured in a Generic Feature Format container file (variants.gff) identifying variant positions in the genome.

Table of contents
=================

  * [Overview](#overview)
  * [Manual](#manual)
    * [Running with SMRTLink](#running-with-smrtanalysis)
    * [Running on the Command-Line](#running-on-the-command-line)
    * [Running on the Command-Line with pbsmrtpipe](#running-on-the-command-line-with-pbsmrtpipe)
  * [Advanced Analysis Options](#advanced-analysis-options)
    * [SMRTLink/pbsmrtpipe Resequencing Options](#smrtlinkpbsmrtpipe-resequencing-options)
    * [PBAlign Options](#pbalign-options)
    * [variantCaller Options](#variantcaller-options)
  * [Output Files](#output-files)
    * [PBAlign Output Files](#pbalign-output-files)
    * [GenomicConsensus Output Files](#genomicconsensus-output-files)
  * [Algorithm Modules](#algorithm-modules)
  * [Glossary](#glossary)

## Overview

![resequencing](https://cloud.githubusercontent.com/assets/12494820/11485866/7cd26f12-976a-11e5-8a22-fc76d70507b7.png)

Analyses are performed in two stages: PBAlign, and GenomicConsensus. 
* __PBAlign__
  * TODO
* __GenomicConensus__
  * TODO

##Manual
There are three ways to run Resequencing: Using SMRTLink, on the command line, and on the command line using pbsmrtpipe so that you can run the whole Resequencing analysis with one command given to pbsmrtpipe. 

##Running with SMRTLink

To run Isoseq using SMRTLink, follow the usual steps for analysing data on SMRTLink. TODO: Link to document explaining SMRTLink.

##Running on the Command Line

On the command line, the analysis is performed in 2 steps:

1. Align sequences in your FASTA, BAM, or XML to a reference using PBAlign, generating an aligned BAM.
2. Call variants from your aligned BAM using variantCaller (Genomic Consensus)

__Step 1. PBAlign__

First, align your sequences to your chosen reference.

     pbalign --concordant --hitpolicy=randombest --minAccuracy 70 --minLength 50 --algorithmOptions="-minMatch 12 -bestn 10 -minPctIdentity 70.0" subreads.bam reference.fasta aligned_subreads.bam

Where your reference sequence is in reference.fasta, your unaligned reads are in subreads.bam, and the file to store your aligned reads is aligned_subreads.bam.

__Step 2. GenomicConsensus__

Next, call variants from the aligned BAM using variantCaller.

     variantCaller --algorithm=quiver  -r reference.fasta --diploid=false --minConfidence=40 --minCoverage=5 -o variants.gff -o consensus.fasta.gz -o consesus.fastq aligned_subreads.bam

Where your reference sequence is in reference.fasta, your aligned reads are in aligned_subreads.bam, the variant callset will be stored in variants.gff, and the consensus sequences are stored in consensus.fastq and consensus.fastq.gz. 

##Running on the Command-Line with pbsmrtpipe

###Install pbsmrtpipe
pbsmrtpipe is a part of `smrtanalysis-3.0` package and will be installed
if `smrtanalysis-3.0` has been installed on your system. Or you can [download   pbsmrtpipe](https://github.com/PacificBiosciences/pbsmrtpipe) and [install](http://pbsmrtpipe.readthedocs.org/en/master/).
    
You can verify that pbsmrtpipe is running OK by:

    pbsmrtpipe --help

### Create a dataset
Now create an XML file from your subreads.

```
dataset create --type SubreadSet my.subreadset.xml subreads1.bam subreads2.bam ...
```
This will create a file called `my.subreadset.xml`. 


### Create and edit resequencing options and global options for `pbsmrtpipe`.
Create a global options XML file which contains SGE related, job chunking and
job distribution options that you may modify by:

```
 pbsmrtpipe show-workflow-options -o global_options.xml
```

Create a resequencing options XML file which contains resequencing-related options that 
you may modify by:
```
 pbsmrtpipe show-template-details pbsmrtpipe.pipelines.sa3_ds_resequencing -o isoseq_options.xml
```

The entries in the options XML files have the format:

```
 <option id="pbtranscript.task_options.min_seq_len">
            <value>300</value>
        </option>
```

And you can modify options using your favorite text editor, such as vim.

### Run Resequencing from pbsmrtpipe
Once you have set your options, you are ready to run resequencing via pbsmrtpipe:

```
pbsmrtpipe pipeline-id pbsmrtpipe.pipelines.sa3_ds_resequencing -e eid_subread:my.subreadset.xml --preset-xml=isoseq_options.xml --preset-xml=global_options.xml
```


## Advanced Analysis Options

## SMRTLink/pbsmrtpipe Resequencing Options

You may modify advanced analysis parameters for Resequencing as described below via SMRTLink.

| Module |           Parameter           |     Default      |  Explanation      |
| ------ | -------------------------- | --------------------------- | ----------------- |
| GenomicConsenus | Algorithm Name  | quiver  | Specifies the algorithm to be used by GenomicConsenus. Options are "quiver" and "plurality" |
| GenomicConsenus | Diploid mode (experimental)  | unchecked  | Enable detection of heterozygous variants (experimental) |
| GenomicConsenus | Minimum confidence  | 40  | The minimum confidence for a variant call to be output to variants.gff |
| GenomicConsenus | Minimum coverage  | 5  | The minimum site coverage that must be achieved for variant calls and consensus to be calculated for a site. |
| PBAlign | Algorithm options  | -minMatch 12 -bestn 10 -minPctIdentity 70.0  | List of space-separated arguments passed to BLASR |
| PBAlign | Concordant alignment  | checked  | Map subreads of a ZMW to the same genomic location |
| PBAlign | Consolidate .bam  | unchecked  | Merge chunked/gathered .bam files |
| PBAlign | Number of .bam files  | 1  | Number of .bam files to create in consolidate mode |
| GenomicConsenus | Hit policy  | 40  | Specify a policy for how to treat multiple hit random : selects a random hit. all : selects all hits. allbest : selects all the best score hits. randombest: selects a random hit from all best score hits. leftmost : selects a hit which has the best score and the smallest mapping coordinate in any reference. Default value is randombest. |
| GenomicConsenus | Min. accuracy  | 70  | Minimum required alignment accuracy (percent) |
| GenomicConsenus | Min. length  | 50  | Minimum required alignment length |


## PBAlign Options

## variantCaller Options
In order to show variantCaller advanced options via command line: `variantCaller --help`.

| Type  |  Parameter          |     Example      |  Explanation      |
| ----- | ------------------ | ---------------- | ----------------- |
| optional | Help |  -h, --help | show this help message and exit |
| optional | Version |  -v, --version |  show program's version number and exit |
| optional | Emit Tool Contract |   --emit-tool-contract |  Emit Tool Contract to stdout (default: False) |
| optional | Resolved Tool Contract | --resolved-tool-contract RESOLVED_TOOL_CONTRACT | Run Tool directly from a PacBio Resolved tool contract (default: None) |
| optional | Log Level |  --log-level {DEBUG,INFO,WARNING,ERROR,CRITICAL} | Set log level (default: INFO) |
| optional | Debug Mode |  --debug | Debug to stdout (default: False) |
| basic required | Input File |  inputFilename |  The input FASTA/BAM/XML file (revised) |
| basic required | Reference File |   --referenceFilename REFERENCEFILENAME, --reference REFERENCEFILENAME, -r REFERENCEFILENAME | The filename of the reference FASTA file (default:None) |
| basic required | Output File |   -o OUTPUTFILENAMES, --outputFilename OUTPUTFILENAMES |  The output filename(s), as a comma-separated list.Valid output formats are .fa/.fasta, .fq/.fastq,.gff (default: []) |
| parallelism | Number of Jobs |  -j NUMWORKERS, --numWorkers NUMWORKERS | The number of worker processes to be used (default: 1) |
| output filtering | Minimum Confidence | --minConfidence MINCONFIDENCE, -q MINCONFIDENCE |  The minimum confidence for a variant call to be output to variants.gff (default: 40) |
| output filtering | Minimum Coverage | --minCoverage MINCOVERAGE, -x MINCOVERAGE | The minimum site coverage that must be achieved for variant calls and consensus to be calculated for a site. (default: 5) |
| output filtering | Help | --noEvidenceConsensusCall {nocall,reference,lowercasereference} | The consensus base that will be output for sites with no effective coverage. (default: lowercasereference) |
| read selection/filtering | Coverage |  --coverage COVERAGE, -X COVERAGE | A designation of the maximum coverage level to be used for analysis. Exact interpretation is algorithm-specific. (default: 100) |
| read selection/filtering | Minimum MapQV |  --minMapQV MINMAPQV, -m MINMAPQV |  The minimum MapQV for reads that will be used for analysis. (default: 10) |
| read selection/filtering | Reference Window |  --referenceWindow REFERENCEWINDOWSASSTRING, --referenceWindows REFERENCEWINDOWSASSTRING, -w REFERENCEWINDOW | The window (or multiple comma-delimited windows) of the reference to be processed, in the format refGroup:refStart-refEnd (default: entire reference).(default: None) |
| read selection/filtering | Alignment Window |  --alignmentSetRefWindows | The window (or multiple comma-delimited windows) of the reference to be processed, in the format refGroup:refStart-refEnd will be pulled from the alignment file. (default: False) |
| read selection/filtering | Reference Window File |  --referenceWindowsFile REFERENCEWINDOWSASSTRING, -W REFERENCEWINDOWSASSTRING |  A file containing reference window designations, one per line (default: None) |
| read selection/filtering | Barcode | --barcode _BARCODE | Only process reads with the given barcode name. (default: None) |
| read selection/filtering | Read Strata | --readStratum READSTRATUM |  A string of the form 'n/N', where n, and N are integers, 0 <= n < N, designating that the reads are to be deterministically split into N strata of roughly even size, and stratum n is to be used for variant and consensus calling. This is mostly useful for Quiver development. (default: None) |
| algorithm and parameter settings | Algorithm |  --algorithm ALGORITHM | No Description (greg) |
| algorithm and parameter settings | Quiver Parameter File | --parametersFile PARAMETERSFILE, -P PARAMETERSFILE |  Parameter set filename (QuiverParameters.ini), or directory D such that either  D/*/GenomicConsensus/QuiverParameters.ini, or D/GenomicConsensus/QuiverParameters.ini, is found. In the former case, the lexically largest path is chosen. (default: None) |
| algorithm and parameter settings | Parameter Specs | --parametersSpec PARAMETERSSPEC, -p PARAMETERSSPEC | Name of parameter set (chemistry.model) to select from the parameters file, or just the name of the chemistry, in which case the best available model is chosen. Default is 'auto', which selects the best parameter set from the cmp.h5 (default: auto) |
| optional | Help |  -h, --help | show this help message and exit |
| optional | Help |  -h, --help | show this help message and exit |
| optional | Help |  -h, --help | show this help message and exit |
| optional | Help |  -h, --help | show this help message and exit |
| optional | Help |  -h, --help | show this help message and exit |
| optional | Help |  -h, --help | show this help message and exit |
| optional | Help |  -h, --help | show this help message and exit |


## Output Files
## PBAlign Output Files

##GenomicConsensus Output Files

## Algorithm Modules

__PBAlign__

__GenomicConsensus__

## Glossary

