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
  * [Files](#files)
    * [PBAlign Files](#pbalign-files)
    * [GenomicConsensus Files](#genomicconsensus-files)
  * [Algorithm Modules](#algorithm-modules)
  * [Glossary](#glossary)

## Overview

![resequencing](https://cloud.githubusercontent.com/assets/12494820/11510701/e22e0c36-9819-11e5-839f-ca30394036a1.png)

Analyses are performed in two stages: PBAlign, and variantCaller. 
* __PBAlign__
  * PBAlign maps PacBio sequences to references using an algorithm selected from a
selection of supported command-line alignment algorithms. Input can be a
fasta, pls.h5, bas.h5 or ccs.h5 file or a fofn (file of file names). Output
can be in CMP.H5, SAM or BAM format. If output is BAM format, aligner can
only be blasr and QVs will be loaded automatically.

* __variantCaller__
  * variantCaller uses a user-specified algorithm from the GenomicConsensus tool to construct consensus sequences and a set of variants. 

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

     pbalign --concordant --hitPolicy=randombest --minAccuracy 70 --minLength 50 --algorithmOptions="-minMatch 12 -bestn 10 -minPctIdentity 70.0" subreads.bam reference.fasta aligned_subreads.bam

Where your reference sequence is in reference.fasta, your unaligned reads are in subreads.bam, and the file to store your aligned reads is aligned_subreads.bam. Note that you will need to have an index file for your reference.fasta. To index, use `samtools faidx reference.fasta`.

__Step 2. variantCaller__

Next, call variants from the aligned BAM using variantCaller.

     variantCaller --algorithm=quiver  -r reference.fasta --diploid --minConfidence=40 --minCoverage=5 -o variants.gff -o consensus.fasta.gz -o consensus.fastq aligned_subreads.bam

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

| Type  |  Parameter          |     Example      |  Explanation      |
| ----- | ------------------ | ---------------- | ----------------- |
| positional | Input File |  unaligned.bam | SubreadSet or unaligned .bam |
| positional | Reference File |  reference.fasta | Reference DataSet or FASTA file |
| positional | Output File |  aligned.bam | Output AlignmentSet file |
| optional | Help | -h, --help | show this help message and exit |
| optional | Version | -v, --version | show program's version number and exit |
| optional | Verbose | --verbose | No Description (greg) |
| optional | Debug | --debug | Writes the debug reporting to stdout |
| optional | Profile | --profile | Print runtime profile at exit |
| optional input | Region Table | --regionTable REGIONTABLE | Specify a region table for filtering reads. |
| optional input | Configuration File | --configFile CONFIGFILE | Specify a set of user-defined argument values. |
| optional input | Pulse File | --pulseFile PULSEFILE | When input reads are in fasta format and output is a cmp.h5 this option can specify pls.h5 or bas.h5 or FOFN files from which pulse metrics can be loaded for Quiver. |
| alignment | Algorithm | --algorithm {blasr, bowtie, gmap} | Select an aligorithm from ('blasr', 'bowtie', 'gmap'). Default algorithm is blasr. |
| alignment | Maximum Hits | --maxHits MAXHITS | The maximum number of matches of each read to the reference sequence that will be evaluated. Default value is 10. |
| alignment | Minimum Anchor Size | --minAnchorSize MINANCHORSIZE | The minimum anchor size defines the length of the read that must match against the reference sequence. Default value is 12. |
| alignment | Use CCS | --useccs {useccs, useccsall, useccsdenovo} | Map the ccsSequence to the genome first, then align subreads to the interval that the CCS reads mapped to. useccs: only maps subreads that span the length of the template. useccsall: maps all subreads. useccsdenovo: maps ccs only. |
| alignment | No Split Subreads | --noSplitSubreads | Do not split reads into subreads even if subread regions are available. Default value is False. |
| alignment | Concordant | --concordant | Map subreads of a ZMW to the same genomic location. |
| alignment | Number of Threads | --nproc NPROC | Number of threads. Default value is 8. |
| alignment | Algorithm Options | --algorithmOptions ALGORITHMOPTIONS | Pass alignment options through. |
| filter criteria | Maximum Divergence | --maxDivergence MAXDIVERGENCE | The maximum allowed percentage divergence of a read from the reference sequence. Default value is 30.0. |
| filter criteria | Minimum Accuracy | --minAccuracy MINACCURACY | The minimum percentage accuracy of alignments that will be evaluated. Default value is 70.0. |
| filter criteria | Minimum Length | --minLength MINLENGTH | The minimum aligned read length of alignments that will be evaluated. Default value is 50. |
| filter criteria | Score Cutoff | --scoreCutoff SCORECUTOFF | The worst score to output an alignment. |
| filter criteria | Hit Policy | --hitPolicy {randombest, allbest, random, all, leftmost} | Specify a policy for how to treat multiple hit random: selects a random hit. all: selects all hits. allbest: selects all the best score hits. randombest: selects a random hit from all best score hits. leftmost: selects a hit which has the best score and the smallest mapping coordinate in any reference. Default value is randombest. |
| filter criteria | Filter Adapter-Only | --filterAdapterOnly |  If specified, do not report adapter-only hits using annotations with the reference entry. |
| for cmp.h5 | For Quiver | --forQuiver | The output cmp.h5 file which will be sorted, loaded with pulse QV information, and repacked, so that it can be consumed by quiver directly. This requires the input file to be in PacBio bas/pls.h5 format, and --useccs must be None. Default value is False. |
| for cmp.h5 | Load QVs | --loadQVs | Similar to --forQuiver, the only difference is that --useccs can be specified. Default value is False. |
| for cmp.h5 | By Read | --byread | Load pulse information using -byread option instead of -bymetric. Only works when --forQuiver or --loadQVs are set. Default value is False. |
| for cmp.h5 | Metrics | --metrics METRICS | Load the specified (comma-delimited list of) metrics instead of the default metrics required by quiver. This option only works when --forQuiver  or --loadQVs are set. Default: DeletionQV, DeletionTag, InsertionQV, MergeQV, SubstitutionQV |
| miscellaneous | Seed | --seed SEED | Initialize the random number generator with a none-zero integer. Zero means that current system time is used. Default value is 1. |
| miscellaneous | Temporary Directory | --tmpDir TMPDIR | Specify a directory for saving temporary files. Default is /scratch. |

## variantCaller Options
In order to show variantCaller advanced options via command line: `variantCaller --help`.

| Type  |  Parameter          |     Example      |  Explanation      |
| ----- | ------------------ | ---------------- | ----------------- |
| optional | Help |  -h, --help | show this help message and exit |
| optional | Version |  -v, --version |  show program's version number and exit |
| optional | Emit Tool Contract |   --emit-tool-contract |  Emit Tool Contract to stdout (default: False) |
| optional | Resolved Tool Contract | --resolved-tool-contract RESOLVED_TOOL_CONTRACT | Run Tool directly from a PacBio Resolved tool contract (default: None) |
| optional | Log Level |  --log-level {DEBUG, INFO, WARNING, ERROR, CRITICAL} | Set log level (default: INFO) |
| optional | Debug Mode |  --debug | Debug to stdout (default: False) |
| basic required | Input File |  inputFilename |  The input FASTA/BAM/XML file (revised) |
| basic required | Reference File |   --referenceFilename REFERENCEFILENAME, --reference REFERENCEFILENAME, -r REFERENCEFILENAME | The filename of the reference FASTA file (default:None) |
| basic required | Output File |   -o OUTPUTFILENAMES, --outputFilename OUTPUTFILENAMES |  The output filename(s), as a comma-separated list.Valid output formats are .fa/.fasta, .fq/.fastq,.gff (default: []) |
| parallelism | Number of Jobs |  -j NUMWORKERS, --numWorkers NUMWORKERS | The number of worker processes to be used (default: 1) |
| output filtering | Minimum Confidence | --minConfidence MINCONFIDENCE, -q MINCONFIDENCE |  The minimum confidence for a variant call to be output to variants.gff (default: 40) |
| output filtering | Minimum Coverage | --minCoverage MINCOVERAGE, -x MINCOVERAGE | The minimum site coverage that must be achieved for variant calls and consensus to be calculated for a site. (default: 5) |
| output filtering | Help | --noEvidenceConsensusCall {nocall,reference,lowercasereference} | The consensus base that will be output for sites with no effective coverage. (default: lowercasereference) |
| read selection and filtering | Coverage |  --coverage COVERAGE, -X COVERAGE | A designation of the maximum coverage level to be used for analysis. Exact interpretation is algorithm-specific. (default: 100) |
| read selection and filtering | Minimum MapQV |  --minMapQV MINMAPQV, -m MINMAPQV |  The minimum MapQV for reads that will be used for analysis. (default: 10) |
| read selection and filtering | Reference Window |  --referenceWindow REFERENCEWINDOWSASSTRING, --referenceWindows REFERENCEWINDOWSASSTRING, -w REFERENCEWINDOW | The window (or multiple comma-delimited windows) of the reference to be processed, in the format refGroup:refStart-refEnd (default: entire reference).(default: None) |
| read selection and filtering | Alignment Window |  --alignmentSetRefWindows | The window (or multiple comma-delimited windows) of the reference to be processed, in the format refGroup:refStart-refEnd will be pulled from the alignment file. (default: False) |
| read selection and filtering | Reference Window File |  --referenceWindowsFile REFERENCEWINDOWSASSTRING, -W REFERENCEWINDOWSASSTRING |  A file containing reference window designations, one per line (default: None) |
| read selection and filtering | Barcode | --barcode _BARCODE | Only process reads with the given barcode name. (default: None) |
| read selection and filtering | Read Strata | --readStratum READSTRATUM |  A string of the form 'n/N', where n, and N are integers, 0 <= n < N, designating that the reads are to be deterministically split into N strata of roughly even size, and stratum n is to be used for variant and consensus calling. This is mostly useful for Quiver development. (default: None) |
| algorithm and parameter settings | Algorithm |  --algorithm ALGORITHM | No Description (greg) |
| algorithm and parameter settings | Quiver Parameter File | --parametersFile PARAMETERSFILE, -P PARAMETERSFILE |  Parameter set filename (QuiverParameters.ini), or directory D such that either  D/*/GenomicConsensus/QuiverParameters.ini, or D/GenomicConsensus/QuiverParameters.ini, is found. In the former case, the lexically largest path is chosen. (default: None) |
| algorithm and parameter settings | Parameter Specs | --parametersSpec PARAMETERSSPEC, -p PARAMETERSSPEC | Name of parameter set (chemistry.model) to select from the parameters file, or just the name of the chemistry, in which case the best available model is chosen. Default is 'auto', which selects the best parameter set from the cmp.h5 (default: auto) |
| Verbosity and debugging and profiling | Verbosity Level | --verbose | Set the verbosity level. (default: None) (greg) (what are the other levels?)|
| Verbosity and debugging and profiling | Quiet | --quiet | Turn off all logging, including warnings (default:False) |
| Verbosity and debugging and profiling | Profile | --profile | Enable Python-level profiling (using cProfile).(default: False) |
| Verbosity and debugging and profiling | Dump Evidence | --dumpEvidence [{variants,all}], -d [{variants,all}] | No Description (greg) |
| Verbosity and debugging and profiling | Evidence Directory | --evidenceDirectory EVIDENCEDIRECTORY | No Description (greg) |
| Verbosity and debugging and profiling | Annotate GFF | --annotateGFF | Augment GFF variant records with additional information (default: False) |
| Advanced configuration | Diploid | diploid | Enable detection of heterozygous variants (experimental) (default: False) |
| Advanced configuration | Queue Size | --queueSize QUEUESIZE, -Q QUEUESIZE | No Description (greg) |
| Advanced configuration | Thread | --threaded, -T | Run threads instead of processes (for debugging purposes only) (default: False) |
| Advanced configuration | Reference Chunk Size | --referenceChunkSize REFERENCECHUNKSIZE, -C REFERENCECHUNKSIZE | No Description (greg) |
| Advanced configuration | Fancy Chunking | --fancyChunking | Adaptive reference chunking designed to handle coverage cutouts better (default: True) |
| Advanced configuration | Simple Chunking | --simpleChunking | Disable adaptive reference chunking (default: True) |
| Advanced configuration | Reference Chunk Overlap | --referenceChunkOverlap REFERENCECHUNKOVERLAP | No Description (greg) |
| Advanced configuration | Auto-Disable Hdf5 Chunk Cache | --autoDisableHdf5ChunkCache AUTODISABLEHDF5CHUNKCACHE | Disable the HDF5 chunk cache when the number of datasets in the cmp.h5 exceeds the given threshold (default: 500) |
| Advanced configuration | Aligner | --aligner {affine,simple}, -a {affine,simple} | The pairwise alignment algorithm that will be used to produce variant calls from the consensus (Quiver only). (default: affine) |
| Advanced configuration | Refine Dinucleotide Repeats | --refineDinucleotideRepeats | Require quiver maximum likelihood search to try one less/more repeat copy in dinucleotide repeats, which seem to be the most frequent cause of suboptimal convergence (getting trapped in local optimum) (Quiver only) (default: True) |
| Advanced configuration | No Refine Dinucleotide Repeats | --noRefineDinucleotideRepeats | Disable dinucleotide refinement (default: True) |
| Advanced configuration | Fast | --fast | Cut some corners to run faster. Unsupported! (default: False) |
| Advanced configuration | Skip Unrecognized Contigs | --skipUnrecognizedContigs | Do not abort when told to process a reference window (via -w/--referenceWindow[s]) that has no aligned coverage. Outputs emptyish files if there are no remaining non-degenerate windows. Only intended for use by smrtpipe scatter/gather. (default: False) |

##Files
## PBAlign Files

__Configuration File__
(greg)

__Pulse File__
(greg)

__AlignmentSeq File__
(greg)

__Region Table?__
(greg)

##GenomicConsensus Files

__Reference Windows File__
(greg)

__Parameters File__
(greg)

__Annotated GFF__
(greg)

## Algorithm Modules

__BLASR__

(greg)

__Bowtie__

(greg)

__GenomicConsensus__

(dave/lance/nigel)

__GMAP__

(greg)

__PBAlign__

(yli)

## Glossary
* __Chunk Cache?__
  * (greg)
* __Confidence (variant call)__
  * (greg)
* __Dinucleotide repeat (how many repeats before it's a repeat, should this be defined?)__
  * (greg)
* __MapQV, DeletionQV, etc?__
  * (greg)
* __Pulse Metric__
  * (greg)
* __Region Table__
  * (greg)
* __Score__
  * (greg)
 
