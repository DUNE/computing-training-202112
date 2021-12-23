---
title: Data Management
teaching: 45
exercises: 0
questions:
- What are the data management tools and software for DUNE?
- How are different software versions handled?
- What are the best data management practices?

objectives:   
- Learn how to access data from DUNE Data Catalog
- Understand the roles of the tools UPS, mrb and CVMFS 

keypoints:
- SAM and Rucio are data handling systems used by the DUNE collaboration to retrieve data.
- Staging is a necessary step to make sure files are on disk in dCache (as opposed to only on tape).
- Xrootd allows user to stream data file. 
- The Unix Product Setup (UPS) is a tool to ensure consistency between different software versions and reproducibility.
- The multi-repository build (mrb) tool allows code modification in multiple repositories, which is relevant for a large project like LArSoft with different cases (end user and developers) demanding consistency between the builds.
- CVMFS distributes software and related files without installing them on the target computer (using a VM, Virtual Machine).
---

## Session Video

The data management portion from the December 2021 training was captured on video, and is provided here.

<center>
<iframe width="560" height="315" src="https://www.youtube.com/embed/qmTjfBGHSps" title="DUNE Computing Tutorial December 2021 Data Management" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</center>

## Introduction

DUNE data is stored around the world and the storage elements are not always organized in a way that they can be easily inspected. For this purpose we use the SAM web client.

### What is SAM?  

Sequential Access via Metadata (SAM) is a data catalog originally designed for the D0 and CDF high energy physics experiments at Fermilab.  It is now used by most of the Intensity Frontier experiments at Fermilab. The most important objects cataloged in SAM are individual **files** and collections of files called **datasets**.
 
Data files themselves are not stored in SAM, their metadata and physical locations are, and via metadata, you can search for and locate collections of files.  SAM also provides mechanisms for initiating and tracking file delivery through **projects**. 

This lecture will show you how to access data files that have been defined to the DUNE Data Catalog. Execute the following commands after logging in to the DUNE interactive node, and sourcing the main dune setups.

### What is Rucio?
Rucio is the next-generation Data Replica service and is part of DUNE's new Distributed Data Management (DDM) system that is currently in deployment. 
Rucio has two functions:
1. A rule-based system to get files to Rucio Storage Elements (RSEs) around the world and keep them there for the lifeimte of the file.
2. To return the "nearest" replica of any data file for use either in interactive or batch file use.  It is expected that most DUNE users will not be regularly using direct Rucio commands, but other wrapper scripts that call them indirectly.

As of the date of this Dec 2021 tutorial:
- The Rucio client is not installed as a part of the standard DUNE client software
- Most DUNE users are not yet enabled to use it.  But when we do, some of the commands will look like this:

~~~
rucio list-file-replicas protodune-sp:np04_raw_run005801_0001_dl1.root

rucio download protodune-sp:np04_raw_run005801_0001_dl1.root

rucio list-rses
~~~

## Back to SAM

### General considerations
 
SAM was designed to ensure that large scale data-processing was done completely and accurately which leads to some features not always present in a generic catalog but very desirable if one wishes high standards of reproducibility and documentation in data analysis.
 
For example, at the time of the original design, the main storage medium was 8mm tapes using consumer-grade drives.  Drive and tape failure rates were > 1%.  Several SAM design concepts, notably luminosity blocks and parentage tracking, were introduced to allow accurate tracking of files and their associated normalization in a high error-rate environment. 


-     Description of the contents of data collections to allow later retrieval 
-     Tracking of object and collection parentage and description of processing transformations to document the full provenance of any data **object** and ensure accurate normalization
-     Grouping of objects and collection into larger “**datasets**” based on their characteristics
-     Storing physical location of objects
-     Tracking of the processing of collections to allow reprocessing on failure and avoid double processing.
-     Methods (“projects”) for delivering and tracking collections in multi-processing jobs
-     Preservation of data about processing/storage for debugging/reporting


The first 3 goals relate to content and characteristics while the last 3 relate to data storage and processing tools.  
 
### Specifics

1.     The current SAM implementation uses the file as the basic unit of information.   **Metadata** is associated with the file name.  Filenames must be unique in the system. This prevents duplication of data in a sample, as a second copy cannot be cataloged. This makes renaming a file very unwise. A very common practice is to include some of the metadata in the filename, both to make it easier to identify and to ensure uniqueness.    
 
2.     **Metadata** for a file can include file locations but does not have to. A file can have no location at all, or many. When you move or remove a file with an associated SAM location, you need to update the location information.  

3.     SAM does not move files..  It provides location information for a process to use in streaming or copying a file using its own methods. Temporary locations (such as on a grid node) need not be reported to SAM. Permanently storing or removing files requires both moving/removing the file itself and updating its metadata to reflect that location and is generally left up to special packages such as the Fermilab FTS (File Transfer Service) and SAM projects. 
  
4.     Files which are stored on disk or tape are expected to have appropriate file sizes and checksums. One can have duplicate instances of a file in different locations, but they must all be identical.  If one reprocesses an input file, the output may well be subtly different (for example dates stored in the file itself can change the checksum). SAM should force you to choose which version, old or new, is acceptable.  It will not let you catalog both with the same filename. As a result, if you get a named file out of SAM, you can be reasonably certain you got the right copy. 
 
5.     files with duplicate content but different names can be problematic. The reprocessed file mentioned in part 4, if renamed, could cause significant problems if it were allowed into the data sample along with the originals as a job processing all files might get both copies. This is one of the major reasons for the checksums and unique filenames. There is a temptation to put, for example, timestamps, in filenames to generate unique names but that removes a protection against duplication. 
  
6.     Files can have **parents** and **children** and, effectively, birth certificates that can tell you how they were made.  An example would be a set of raw data files RAWnnn processed with code X to produce a single reconstructed file RECO.  One can tell SAM that RAWnnn are the parents of RECO processed with version x of code X. If one later finds another RAWnnn file that was missed in processing, SAM can tell you it has not been processed yet with X (i.e., it has no children associated with version x of X) and you can then choose to process that file. This use case often occurs when a production job fails without reporting back or succeeds but the copy back or catalog action fails. 
Note: The D0 experiment required that all official processing going into SAM be done with tagged releases and fixed parameter sets to increase reproducibility and the tags for that information were included in the metadata. Calibration databases were harder to timestamp so some variability was still possible if calibrations were updated.
 
7.  SAM supports several types of data description fields:
 
Values are standard across all implementations like run_type, file_size …
 
Parameters are defined by the experiment for example MC.Genieversion
 
Values are common to almost all  HEP experiments and are optimized for efficient queries.  SAM also allows definition of “parameters” (by administrators) as they are needed.  This allows the schema to be modified easily as needs arrive.
 
8.  Metadata can also contain “spill” or luminosity block information that allows a file to point to specific data taking periods with smaller granularity than a run or subrun. When files are merged, this spill information is also merged.
 
9. SAM currently does not contain a means of easily determining which file a given event is in.  If a daq system is writing multiple streams, an event from a given subrun could be in any stream.  Adding an event database would be a useful feature.

All of these features are intended to assure that your data are well described and can be found. As SAM stores full location information, this means any SAM-visible location. In addition, if parentage information is provided, you can determine and reproduce the full provenance of any file.
 
## Datasets and projects
 
### Datasets
 
In addition to the files themselves, SAM allows you to define datasets.
 
A *SAM* dataset is not a fixed list of files but a query against the SAM database. An example would be “data_tier reconstructed and run_number 2001 and version v10” which would be all files from run 2001 that are reconstructed data produced by version v10. This dataset is dynamic. If one finds a missing file from run 2001 and reconstructs it with v10, the dataset will grow. There are also dataset snapshots that are derived from datasets and capture the exact files in the dataset when the snapshot was made.
Note: most other data catalogs assume a “dataset” is a fixed list of files.  This is a “snapshot” in SAM. 

### Projects
 
SAM also supports access tracking mechanisms called projects and consumers.  These are generally implemented for you by grid processing scripts. Your job is to choose a dataset and then ask the processing system to launch a project for that dataset.
 
A project is effectively a processing campaign across a dataset which is owned by the SAM system. At launch a snapshot is generated and then the files in the snapshot are delivered to a set of consumers.  The project maintains an internal record of the status of the files and consumers. Each grid process can instantiate a consumer which is attached to the project.  Those consumers then request “files” from the project and, when done processing, tell the project of their status.  
 
The original SAM implementation actually delivered the files to local hard drives.  Modern SAM delivers the location information and expects the consumer to find the optimal delivery method. This is a pull model, where the consuming process requests the next file rather than having the file assigned to it.  This makes the system more robust on distributed systems. 
 
See running projects [here](http://samweb.fnal.gov:8480/station_monitor/dune/stations/dune/projects).

## Accessing the database in read mode

Checking the database does not require special privileges but storing files and running projects modifies the database and requires authentication to the right experimental group.   `kx509` authentication and membership in the experiment VO are needed. 
 
Administrative actions like adding new values are restricted to a small set of superusers for each experiment. 
 
## Suggestions for configuring SAM (for admins)
 
First of all, it really is nice to have filenames and dataset names that tell you what’s in the box, although not required. The D0 and MINERvA conventions have been to use “_” underscores between useful key strings. As a result, D0 and MINERvA tried not to use “\_” in metadata entries to allow cleaner parsing. “-“ is used if needed in the metadata.
 
D0 also appended processing information to filenames as they moved through the system to assure that files run through different sequences had unique identifiers.
 
Example: A Monte Carlo simulation file generated with version v3 and then reconstructed with v5 might look like

~~~
SIM_MC_020000_0000_simv3.root would be a parent of RECO_MC_020000_0000_simv3_recov5.root
~~~
{: .output}

Data files are all children of the raw data while simulation files sometimes have more complicated ancestry, with both unique generated events and overlay events from data as parents.
 
## Setting up SAM metadata (For admins)
 
This needs to be done once, and very carefully, early in the experiment. It can grow but thinking hard at the beginning saves a lot of pain later.
 
You need to define data_tiers. These represent the different types of data that you produced through your processing chain. Examples would be `raw`, `pedsup`, `calibrated`, `reconstructed`, `thumbnail`, `mc-generated`, `mc-geant`, `mc-overlaid`.
 
`run_type` can be used to support multiple DAQ instances. 
 
`data_stream` is often used for trigger subsamples that you may wish to split data into (for example pedestal vs data runs).
 
Generally, you want to store data from a given data_tier with other data from that tier to facilitate fast sequential access. 
 
## Applications
 
It is useful, but not required to also define applications which are triads of “appfamily”, “appname” and “version”. Those are used to figure out what changed X to Y. There are also places to store the machine the application ran on and the start and end time for the job.

The query:
~~~
samweb list-files "data_tier raw and not isparentof: (data_tier reconstructed and appname reco and version 7)"
~~~
{: .language-bash}

Should, in principle, list raw data files not yet processed by version 7 of reco to produce files of tier reconstructed. You would use this to find lost files in your reconstruction after a power outage.
 
It is good practice to also store the name of the head application configuration file for processing but this does not have a standard “value.”

## samweb client

**samweb**  – `samweb` is the command line and python API that allows queries of the SAM metadata, creation of datasets and tools to track and deliver information to batch jobs.

samweb can be acquired from ups via:

~~~
source /cvmfs/dune.opensciencegrid.org/products/dune/setup_dune.sh
setup dunetpc $DUNETPC_VERSION -q e19:prof #use DUNETPC_VERSION v09_22_02
setup_fnal_security
~~~
{: .language-bash}

`samweb` allows you to select on a lot of parameters which are documented here:

* [SAM User Guide](https://cdcvs.fnal.gov/redmine/projects/sam/wiki/User_Guide_for_SAM)
 
* [SAM Query Dimension syntax](https://cdcvs.fnal.gov/redmine/projects/sam-main/wiki/Updated_dimensions_syntax).

* [Useful ProtoDUNE samweb parameters][useful-samweb]

* [dune-data.fnal.gov](https://dune-data.fnal.gov) lists some official dataset definitions 

This exercise will start you accessing data files that have been defined to the DUNE Data Catalog.

### Example metadata from DUNE

Here are some examples of querying sam to get file information
 
~~~
$ samweb get-metadata np04_raw_run005141_0015_dl10_reco_12736632_0_20181028T182951.root –json
~~~
{: .language-bash}
~~~
{
 "file_name": "np04_raw_run005141_0015_dl10_reco_12736632_0_20181028T182951.root", 
 "file_id": 7352771, 
 "create_date": "2018-10-29T14:59:42+00:00", 
 "user": "dunepro", 
 "update_date": "2018-11-28T17:07:30+00:00", 
 "update_user": "schellma",
 "file_size": 14264091111, 
 "checksum": [
  "enstore:1390300706", 
  "adler32:e8bf4e23"
 ], 
 "content_status": "good", 
 "file_type": "detector", 
 "file_format": "artroot", 
 "data_tier": "full-reconstructed", 
 "application": {
  "family": "art", 
  "name": "reco", 
  "version": "v07_08_00_03"
 }, 
 "event_count": 108, 
 "first_event": 21391, 
 "last_event": 22802, 
 "start_time": "2018-10-28T17:34:58+00:00", 
 "end_time": "2018-10-29T14:55:42+00:00", 
 "data_stream": "physics", 
 "beam.momentum": 7.0, 
 "data_quality.online_good_run_list": 1, 
 "detector.hv_value": 180, 
 "DUNE_data.acCouple": 0, 
 "DUNE_data.calibpulsemode": 0, 
 "DUNE_data.DAQConfigName": "np04_WibsReal_Ssps_BeamTrig_00021", 
 "DUNE_data.detector_config": "cob2_rce01:cob2_rce02:cob2.. 4 more lines of text", 
 "DUNE_data.febaselineHigh": 2, 
 "DUNE_data.fegain": 2, 
 "DUNE_data.feleak10x": 0, 
 "DUNE_data.feleakHigh": 1, 
 "DUNE_data.feshapingtime": 2, 
 "DUNE_data.inconsistent_hw_config": 0, 
 "DUNE_data.is_fake_data": 0, 
 "runs": [
  [
   5141, 
   1, 
   "protodune-sp"
  ]
 ], 
 "parents": [
  {
   "file_name": "np04_raw_run005141_0015_dl10.root", 
   "file_id": 6607417
  }
 ]
}
~~~
{: .output}

To find the files produced from this file (children):

```bash 
$ samweb file-lineage children np04_raw_run005141_0015_dl10.root 
``` 
~~~
np04_raw_run005141_0015_dl10_reco_12736632_0_20181028T182951.root
~~~
{: .output}

To find files used to produce this file (parents):

```bash
$ samweb file-lineage parents np04_raw_run005141_0015_dl10_reco_12736632_0_20181028T182951.root
``` 
~~~ 
np04_raw_run005141_0015_dl10.root
~~~
{: .output}

## Locating data

If you know the full filename and want to locate it, e.g.:
~~~
samweb locate-file np04_raw_run005758_0001_dl3.root
~~~
{: .language-bash}

This will give you output that looks like:
~~~
rucio:protodune-sp
cern-eos:/eos/experiment/neutplatform/protodune/rawdata/np04/detector/None/raw/07/42/28/49
castor:/neutplatform/protodune/rawdata/np04/detector/None/raw/07/42/28/49
enstore:/pnfs/dune/tape_backed/dunepro/protodune/np04/beam/detector/None/raw/07/42/28/49(597@vr0337m8)
~~~
{: .output}
which is the locations of the file on disk and tape. But if we want to copy the file from tape to our local disk, then we need the file access URIs:

~~~
$ samweb get-file-access-url np04_raw_run005141_0015_dl10_reco_12736632_0_20181028T182951.root
~~~
{: .language-bash}

~~~
gsiftp://eospublicftp.cern.ch/eos/experiment/neutplatform/protodune/rawdata/np04/output/detector/full-reconstructed/07/35/27/71/np04_raw_run005141_0015_dl10_reco_12736632_0_20181028T182951.root
gsiftp://fndca1.fnal.gov:2811/pnfs/fnal.gov/usr/dune/tape_backed/dunepro/protodune/np04/beam/output/detector/full-reconstructed/07/35/27/71/np04_raw_run005141_0015_dl10_reco_12736632_0_20181028T182951.root
~~~
{: .output}

Here we have shown the gridftp transfer URIs, but in general it is better to stream data with xrootd and so you should add "--schema=root" to the samweb command. This is shown in the next section.

## Accessing data for use in your analysis
To access data without copying it, XRootD is the tool to use. However it will work only if the file is staged to the disk.

You can stream files worldwide if you have a DUNE VO certificate as described in the preparation part of this tutorial.
Better yet, you can use xrootd to access the file without copying it if it is staged to disk. Find the xrootd uri via 

```bash
samweb get-file-access-url np04_raw_run005141_0001_dl7.root --schema=root
``` 
~~~
root://fndca1.fnal.gov:1094/pnfs/fnal.gov/usr/dune/tape_backed/dunepro/protodune/np04/beam/detector/None/raw/06/60/59/05 /np04_raw_run005141_0001_dl7.root
root://castorpublic.cern.ch//castor/cern.ch/neutplatform/protodune/rawdata/np04/detector/None/raw/06/60/59/05/np04_raw_run005141_0001_dl7.root
root://eospublic.cern.ch//eos/experiment/neutplatform/protodune/rawdata/np04/detector/None/raw/06/60/59/05/np04_raw_run005141_0001_dl7.root
~~~
{: .output}

You can localize your file with the `--location` argument (enstore, castor, cern-eos) 

```bash
samweb get-file-access-url np04_raw_run005141_0001_dl7.root --schema=root --location=enstore
``` 
~~~
root://fndca1.fnal.gov:1094/pnfs/fnal.gov/usr/dune/tape_backed/dunepro/protodune/np04/beam/detector/None/raw/06/60/59/05 /np04_raw_run005141_0001_dl7.root
~~~
{: .output}

## Querying SAM Metadata catalog for files

To list raw data files for a given run:
~~~
samweb list-files "run_number 5758 and run_type protodune-sp and data_tier raw"
~~~
{: .language-bash}

~~~
np04_raw_run005758_0001_dl3.root
np04_raw_run005758_0002_dl2.root
...
np04_raw_run005758_0065_dl10.root
np04_raw_run005758_0065_dl4.root
~~~
{: .output}

What about a reconstructed version?
~~~ 
samweb list-files "run_number 5758 and run_type protodune-sp and data_tier full-reconstructed and version (v07_08_00_03,v07_08_00_04)"
~~~
{: .language-bash}

~~~
np04_raw_run005758_0053_dl7_reco_12891068_0_20181101T222620.root
np04_raw_run005758_0025_dl11_reco_12769309_0_20181101T213029.root
np04_raw_run005758_0053_dl2_reco_12891066_0_20181101T222620.root
...
np04_raw_run005758_0061_dl8_reco_14670148_0_20190105T175536.root
np04_raw_run005758_0044_dl6_reco_14669100_0_20190105T172046.root
~~~
{: .output}
The above is truncated output to show us the one reconstructed file that is the child of the raw data file above.

We also group reconstruction versions into Campaigns like PDSPProf4

```bash 
samweb list-files "run_number 5141 and run_type protodune-sp and data_tier full-reconstructed and DUNE.campaign PDSPProd4"
``` 
Gives more recent files like:
~~~
np04_raw_run005141_0009_dl1_reco1_18126423_0_20210318T102429Z.root
~~~
{: .output}

To see the total number of files that match a certain query expression, then add the `--summary` option to `samweb list-files`.

## Creating a dataset

You can make your own samweb dataset definitions: First, make certain a definition does not already exist that satisfies your needs by checking the official pages above. 

Then check to see what you will get:
```bash
samweb list-files --summary "data_tier full-reconstructed and DUNE.campaign PDSPProd4  and data_stream physics and run_type protodune-sp and detector.hv_value 180" –summary
``` 
```
samweb create-definition $USER-PDSPProd4_good_physics "data_tier full-reconstructed and DUNE.campaign PDSPProd4  and data_stream physics and run_type protodune-sp and detector.hv_value 180"
``` 

Note that the `username` appears in the definition name - to prevent users from getting confused with official samples, your user name is required in the definition name. 

### check to see if a file is on tape or disk

```bash
sam_validate_dataset  --locality  --file np04_raw_run005141_0015_dl10.root --location=/pnfs/ --stage_status
``` 
~~~
         Staging status for: file np04_raw_run005141_0015_dl10.root
                Total Files: 1
              Tapes spanned: 1
      Percent files on disk: 0%
Percent bytes online DCache: 0%
locality counts:
ONLINE: 0
NEARLINE: 1
NEARLINE_size: 8276312581
~~~
{: .output}

Oops - this one is not on disk since ONLINE is not 1. ONLINE: 1 if available on disk 

~~~
sam_validate_dataset  --locality --name=schellma-1GeVMC-test --stage_status --location=/pnfs/
         Staging status for: defname:schellma-1GeVMC-test
                Total Files: 140
              Tapes spanned: 10
      Percent files on disk: 100%
Percent bytes online DCache: 100%
locality counts:
ONLINE: 0
ONLINE_AND_NEARLINE: 140
ONLINE_AND_NEARLINE_size: 270720752891
~~~
{: .output}

No `ONLINE_AND_NEARLINE` means you need to prestage that file. Unfortunately, prestaging requires a definition.

The official Protodune dataset definitions are [here](https://wiki.dunescience.org/wiki/ProtoDUNE-SP_datasets).

**Resource**: [Using the SAM Data Catalog][sam-data-control].
> ## Exercise 1
> * Use the `--location` argument to show the path of the file above on either `enstore`, `castor` or `cern-eos`.
> * Use `get-metadata` to get SAM metadata for this file. Note that `--json` gives the output in json format.
{: .challenge}


When we are analyzing large numbers of files in a group of batch jobs, we use a SAM snapshot to describe the full set of files that we are going to analyze and create a SAM Project based on that. Each job will then come up and ask SAM to give it the next file in the list. SAM has some capability to grab the nearest copy of the file. For instance if you are running at CERN and analyzing this file it will automatically take it from the CERN storage space EOS.

> ## Exercise 2
> * use the samweb describe-definition command to see the dimensions of data set PDSPProd4_MC_1GeV_reco1_sce_datadriven_v1
> * use the samweb list-definition-files command with the --summary option to see the total size of PDSPProd4_MC_1GeV_reco1_sce_datadriven_v1
> * use the samweb take-snapshot command to make a snapshot of PDSPProd4_MC_1GeV_reco1_sce_datadriven_v1
{: .challenge}

## What is UPS and why do we need it?

An important requirement for making valid physics results is computational reproducibility. You need to be able to repeat the same calculations on the data and MC and get the same answers every time. You may be asked to produce a slightly different version of a plot for example, and the data that goes into it has to be the same every time you run the program. 

This requirement is in tension with a rapidly-developing software environment, where many collaborators are constantly improving software and adding new features. We therefore require strict version control; the workflows must be stable and not constantly changing due to updates. 

DUNE must provide installed binaries and associated files for every version of the software that anyone could be using. Users must then specify which version they want to run before they run it. All software dependencies must be set up with consistent versions in order for the whole stack to run and run reproducibly.

The Unix Product Setup (UPS) is a tool to handle the software product setup operation. 

UPS is set up when you setup DUNE:
~~~
 source /cvmfs/dune.opensciencegrid.org/products/dune/setup_dune.sh
~~~
{: .language-bash}

This sourcing defines the UPS `setup` command. Now to get DUNE's LArSoft-based software, this is done through:
~~~
 setup dunetpc $DUNETPC_VERSION -q e19:prof
~~~
{: .language-bash}


`dunetpc`: product name <br>
`$DUNETPC_VERSION` version tag <br>
`e19:prof` are "qualifiers". Qualifiers are separated with colons and may be specified in any order. The "e19" qualifier refers to a specific version of the gcc compiler suite, and "prof" means select the installed product that has been compiled with optimizations turned on. An alternative to "prof" is the "debug" qualifier. All builds of LArSoft and dunetpc are compiled with debug symbols turned on, but the "debug" builds are made with optimizations turned off. Both kinds of software can be debugged, but it is easier to debug the debug builds (code executes in the proper order and variables aren't optimized away so they can be inspected).

Another specifier of a product install is the "flavor". This refers to the operating system the program was compiled for. These days we only support SL7, but in the past we used to also support SL6 and various versions of macOS. The flavor is automatically selected when you set up a product using setup (unless you override it which is usually a bad idea). Some product are "unflavored" because they do not contain anything that depends on the operating system. Examples are products that only contain data files or text files.

Setting up a UPS product defines many environment variables. Most products have an environment variable of the form `<productname>_DIR`, where `<productname>` is the name of the UPS product in all capital letters. This is the top-level directory and can be used when searching for installed source code or fcl files for example. `<productname>_FQ_DIR` is the one that specifies a particular qualifier and flavor.

> ## Exercise 3
> * show all the versions of dunetpc that are currently available by using the "ups list -aK+ dunetpc" command
> * pick one version and substitute that for DUNETPC_VERSION above and set up dunetpc
{: .challenge}

Many products modify the following search path variables, prepending their pieces when set up. These search paths are needed by _art_ jobs.

`PATH`: colon-separated list of directories the shell uses when searching for programs to execute when you type their names at the command line. The command "which" tells you which version of a program is found first in the PATH search list. Example:
~~~
which lar
~~~
{: .language-bash}
will tell you where the lar command you would execute is if you were to type "lar" at the command prompt. 
The other paths are needed by _art_ for finding plug-in libraries, fcl files, and other components, like gdml files.  
`CET_PLUGIN_PATH`  
`LD_LIBRARY_PATH`  
`FHICL_FILE_PATH`  
`FW_SEARCH_PATH`  

Also the PYTHONPATH describes where Python modules will be loaded from.

### UPS basic commands

| Command                                        | Action                                                           |
|------------------------------------------------|------------------------------------------------------------------|
| `ups list -aK+ dunetpc`                        | List the versions and flavors of dunetpc that exist on this node |
| `ups active`                                   | Displays what has been setup                                     |
| `ups depend dunetpc v08_57_00 -q e19:prof:py2` | Displays the dependencies for this version of dunetpc            |

> ## Exercise 4
> * show all the dependencies of dunetpc by using "ups depend dunetpc $DUNETPC_VERSION -q e19:prof"
{: .challenge}

>## UPS Docmentation Links

> * [UPS reference manual](http://www.fnal.gov/docs/products/ups/ReferenceManual/)
> * [UPS documentation](https://cdcvs.fnal.gov/redmine/projects/ups/wiki)
> * [UPS qualifiers](https://cdcvs.fnal.gov/redmine/projects/cet-is-public/wiki/AboutQualifiers)

## mrb
**What is mrb and why do we need it?**  
Early on, the LArSoft team chose git and cmake as the software version manager and the build language, respectively, to keep up with industry standards and to take advantage of their new features. When we clone a git repository to a local copy and check out the code, we end up building it all. We would like LArSoft and DUNE code to be more modular, or at least the builds should reflect some of the inherent modularity of the code.

Ideally, we would like to only have to recompile a fraction of the software stack when we make a change. The granularity of the build in LArSoft and other art-based projects is the repository. So LArSoft and DUNE have divided code up into multiple repositories (DUNE ought to divide more than it has, but there are a few repositories already with different purposes). Sometimes one needs to modify code in multiple repositories at the same time for a particular project. This is where mrb comes in. 

**mrb** stands for "multi-repository build". mrb has features for cloning git repositories, setting up build and local products environments, building code, and checking for consistency (i.e. there are not two modules with the same name or two fcl files with the same name). mrb builds UPS products -- when it installs the built code into the localProducts directory, it also makes the necessasry UPS table files and .version directories. mrb also has a tool for making a tarball of a build product for distribution to the grid. The software build example later in this tutorial exercises some of the features of mrb. 

| Command                  | Action                                              |
|--------------------------|-----------------------------------------------------|
| `mrb --help`             | prints list of all commands with brief descriptions |
| `mrb \<command\> --help` | displays help for that command                      |
| `mrb gitCheckout`        | clone a repository into working area                |
| `mrbsetenv`              | set up build environment                            |
| `mrb build -jN`          | builds local code with N cores                      |
| `mrb b -jN`              | same as above                                       |
| `mrb install -jN`        | installs local code with N cores                    |
| `mrb i -jN`              | same as above (this will do a build also)           |
| `mrbslp`                 | set up all products in localProducts...             |
| `mrb z`                  | get rid of everything in build area                 |

Link to the [mrb reference guide](https://cdcvs.fnal.gov/redmine/projects/mrb/wiki/MrbRefereceGuide)

> ## Exercise 5
> There is no exercise 5. mrb example exercises will be covered in Friday morning's session as any useful exercise with mrb takes more than 30 minutes on its own. Everyone gets 100% credit for this exercise!
{: .challenge}


## CVMFS  
**What is CVMFS and why do we need it?**  
DUNE has a need to distribute precompiled code to many different computers that collaborators may use. Installed products are needed for four things: 
1. Running programs interactively
2. Running programs on grid nodes
3. Linking programs to installed libraries
4. Inspection of source code and data files

Results must be reproducible, so identical code and associated files must be distributed everywhere. DUNE does not own any batch resources -- we use CPU time on computers that participating institutions donate to the Open Science Grid. We are not allowed to install our software on these computers and must return them to their original state when our programs finish running so they are ready for the next job from another collaboration.

CVMFS is a perfect tool for distributing software and related files. It stands for CernVM File System (VM is Virtual Machine). Local caches are provided on each target computer, and files are accessed via the `/cvmfs` mount point. DUNE software is in the directory `/cvmfs/dune.opensciencegrid.org`, and LArSoft code is in `/cvmfs/larsoft.opensciencegrid.org`. These directories are auto-mounted and need to be visible when one executes `ls /cvmfs` for the first time.  Some software is also in /cvmfs/fermilab.opensciencegrid.org.

CVMFS also provides a de-duplication feature.  If a given file is the same in all 100 releases of dunetpc, it is only cached and transmitted once, not independently for every release.  So it considerably decreases the size of code that has to be transferred.

When a file is accessed in `/cvmfs`, a daemon on the target computer wakes up and determines if the file is in the local cache, and delivers it if it is. If not, the daemon contacts the CVMFS repository server responsible for the directory, and fetches the file into local cache. In this sense, it works a lot like AFS. But it is a read-only filesystem on the target computers, and files must be published on special CVMFS publishing servers. Files may also be cached in a layer between the CVMFS host and the target node in a squid server, which helps facilities with many batch workers reduce the network load in fetching many copies of the same file, possibly over an international connection.

CVMFS also has a feature known as "Stashcache" or "xCache".  Files that are in /cvmfs/dune.osgstorage.org are not actually transmitted 
in their entirety, only pointers to them are, and then they are fetched from one of several regional cache servers or in the case of DUNE from Fermilab dCache directly.  DUNE uses this to distribute photon library files, for instance.  

CVMFS is by its nature read-all so code is readable by anyone in the world with a CVMFS client.  CVMFS clients are available for download to desktops or laptops.  Sensitive code can not be stored in CVMFS.

More information on CVMFS is available [here](https://wiki.dunescience.org/wiki/DUNE_Computing/Access_files_in_CVMFS)

> ## Exercise 6
> * cd /cvmfs and do an ls at top level
> * What do you see--do you see the four subdirectories (dune.opensciencegrid.org, larsoft.opensciencegrid.org, fermilab.opensciencegrid.org, and dune.osgstorage.org)
> * cd dune.osgstorage.org/pnfs/fnal.gov/usr/dune/persistent/stash/PhotonPropagation/LibraryData
{: .challenge}

## Useful links to bookmark
* Official dataset definitions: [dune-data.fnal.gov](https://dune-data.fnal.gov)
* [UPS reference manual](http://www.fnal.gov/docs/products/ups/ReferenceManual/)
* [UPS documentation (redmine)](https://cdcvs.fnal.gov/redmine/projects/ups/wiki)
* UPS qualifiers: [About Qualifiers (redmine)](https://cdcvs.fnal.gov/redmine/projects/cet-is-public/wiki/AboutQualifiers)
* [mrb reference guide (redmine)](https://cdcvs.fnal.gov/redmine/projects/mrb/wiki/MrbRefereceGuide)
* CVMFS on DUNE wiki: [Access files in CVMFS](https://wiki.dunescience.org/wiki/DUNE_Computing/Access_files_in_CVMFS)


[Ifdh_commands]: https://cdcvs.fnal.gov/redmine/projects/ifdhc/wiki/Ifdh_commands
[xrootd-man-pages]: https://xrootd.slac.stanford.edu/docs.html
[Understanding-storage]: https://cdcvs.fnal.gov/redmine/projects/fife/wiki/Understanding_storage_volumes
[useful-samweb]: https://wiki.dunescience.org/wiki/Useful_ProtoDUNE_samweb_parameters
[dune-data-fnal]: https://dune-data.fnal.gov/
[dune-data-fnal-how-works]: https://dune-data.fnal.gov/tutorial/howitworks.pdf
[sam-data-control]: https://wiki.dunescience.org/wiki/Using_the_SAM_Data_Catalog_to_find_data
