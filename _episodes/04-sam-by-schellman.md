---
title: SAM by Schellman
teaching: 5
exercises: 0
questions:  
- What event information can be queried for a given data file?
objectives:  
- Learn about the utility of SAM
- Practice selected SAM commands
keypoints:
- SAM is a data catalog originally designed for the D0 and CDF experiments at FNAL and is now used widely by HEP experiments.
---

## Notes on the SAM data catalog system

These notes are provided as an ancilliary resource on the topic of DUNE data mananagement by Dr. Heidi Schellman 1-28-2020, and updated Dec. 2021
 
## Introduction

SAM is a data catalog originally designed for the D0 and CDF high energy physics experiments at Fermilab.  It is now used by most of the Intensity Frontier experiments at Fermilab. 
 
The most important objects cataloged in SAM are individual **files** and collections of files called **datasets**.
 
Data files themselves are not stored in SAM, their metadata is, and that metadata allows you to search for and find the actual physical files.  SAM also provides mechanisms for initiating and tracking file delivery through **projects**. 
 
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
 
**samweb**  – `samweb` is the command line and python API that allows queries of the SAM metadata, creation of datasets and tools to track and deliver information to batch jobs.

samweb can be acquired from ups via 
~~~
setup samweb_client
~~~
{: .language-bash}

Or installed locally via 

```bash
git clone http://cdcvs.fnal.gov/projects/sam-web-client
``` 

You then need to do something like:

~~~
export  PATH=$HOME/sam-web-client/bin:${PATH}
export  PYTHONPATH=$HOME/sam-web-client/python:${PYTHONPATH}
export SAM_EXPERIMENT=dune
~~~
{: .language-bash}
 
## Projects
 
SAM also supports access tracking mechanisms called projects and consumers.  These are generally implemented for you by grid processing scripts. Your job is to choose a dataset and then ask the processing system to launch a project for that dataset.
 
A project is effectively a processing campaign across a dataset which is owned by the SAM system. At launch a snapshot is generated and then the files in the snapshot are delivered to a set of consumers.  The project maintains an internal record of the status of the files and consumers. Each grid process can instantiate a consumer which is attached to the project.  Those consumers then request “files” from the project and, when done processing, tell the project of their status.  
 
The original SAM implementation actually delivered the files to local hard drives.  Modern SAM delivers the location information and expects the consumer to find the optimal delivery method. This is a pull model, where the consuming process requests the next file rather than having the file assigned to it.  This makes the system more robust on distributed systems. 
 
See running projects [here](http://samweb.fnal.gov:8480/station_monitor/dune/stations/dune/projects).
 
## Accessing the database in read mode

Checking the database does not require special privileges but storing files and running projects modifies the database and requires authentication to the right experimental group.   `kx509` authentication and membership in the experiment VO are needed. 
 
Administrative actions like adding new values are restricted to a small set of superusers for each experiment. 
 
### Suggestions for configuring SAM (for admins)
 
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
 
~~~
$ samweb get-file-access-url np04_raw_run005141_0015_dl10_reco_12736632_0_20181028T182951.root
~~~
{: .language-bash}

~~~
gsiftp://eospublicftp.cern.ch/eos/experiment/neutplatform/protodune/rawdata/np04/output/detector/full-reconstructed/07/35/27/71/np04_raw_run005141_0015_dl10_reco_12736632_0_20181028T182951.root
gsiftp://fndca1.fnal.gov:2811/pnfs/fnal.gov/usr/dune/tape_backed/dunepro/protodune/np04/beam/output/detector/full-reconstructed/07/35/27/71/np04_raw_run005141_0015_dl10_reco_12736632_0_20181028T182951.root
~~~
{: .output}

```bash 
$ samweb file-lineage children np04_raw_run005141_0015_dl10.root 
``` 
~~~
np04_raw_run005141_0015_dl10_reco_12736632_0_20181028T182951.root
~~~
{: .output}

```bash
$ samweb file-lineage parents
``` 
~~~
np04_raw_run005141_0015_dl10_reco_12736632_0_20181028T182951.root 
np04_raw_run005141_0015_dl10.root
~~~
{: .output}
 
## Merging and splitting (for experts)
 
Parentage works pretty well if one is merging files but splitting them can become problematic as it makes the parentage structure pretty complex.
SAM will let you merge files with different attributes if you don’t check carefully. Generally, it is a good idea not to merge files from different data tiers and certainly not from different data_types. Merging across major processing versions should also be avoided.
 
### Example: Execute samweb Commands
 
There is documentation at [here](https://cdcvs.fnal.gov/redmine/projects/sam/wiki/User_Guide_for_SAM) and [here](https://cdcvs.fnal.gov/redmine/projects/sam-main/wiki/Updated_dimensions_syntax).

This exercise will start you accessing data files that have been defined to the DUNE Data Catalog. Execute the following commands after logging in to the DUNE interactive node, creating the directories above - Once per session

```bash
setup sam_web_client #(or set up your standalone version)
``` 
~~~
export SAM_EXPERIMENT=dune
~~~
{: .output}

Then if curious about a file: 

```bash 
samweb locate-file np04_raw_run005141_0001_dl7.root
``` 
this will give you output that looks like 
~~~
rucio:protodune-sp
enstore:/pnfs/dune/tape_backed/dunepro/protodune/np04/beam/detector/None/raw/06/60/59/05(596@vr0072m8)
castor:/neutplatform/protodune/rawdata/np04/detector/None/raw/06/60/59/05
cern-eos:/eos/experiment/neutplatform/protodune/rawdata/np04/detector/None/raw/06/60/59/05
~~~
{: .output}

which are the locations of the file on disk and tape. We can use this to copy the file from tape to our local disk. 
Better yet, you can use xrootd to access the file without copying it if it is staged to disk.
Find the xrootd uri via 
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

```bash
samweb get-file-access-url np04_raw_run005141_0001_dl7.root --schema=root --location=cern-eos
``` 
~~~
root://eospublic.cern.ch//eos/experiment/neutplatform/protodune/rawdata/np04/detector/None/raw/06/60/59/05/np04_raw_run005141_0001_dl7.root 
~~~
{: .output}

To get SAM metadata for a file for which you know the name: 

```bash 
samweb  get-metadata np04_raw_run005141_0001_dl7.root 
``` 

add the `--json` option to get output in json format

To list raw data files for a given run: 

```bash
samweb list-files "run_number 5141 and run_type protodune-sp and data_tier raw"
``` 
What about a reconstructed version?  

```bash
samweb list-files "run_number 5141 and run_type protodune-sp and data_tier full-reconstructed and version (v07_08_00_03,v07_08_00_04)"
``` 

Gives a list of files  from the first production like 
 
~~~
np04_raw_run005141_0001_dl7_reco_12736115_0_20181028T165152.root
~~~
{: .output}

We also group reconstruction versions into Campaigns like PDSPProf4

```bash 
samweb list-files "run_number 5141 and run_type protodune-sp and data_tier full-reconstructed and DUNE.campaign PDSPProd4"
``` 

Gives more recent files like:

~~~
np04_raw_run005141_0009_dl1_reco1_18126423_0_20210318T102429Z.root
~~~
{: .output}

samweb allows you to select on a lot of parameters 

Useful ProtoDUNE samweb parameters can be found  [here](https://dune-data.fnal.gov) and [here](https://wiki.dunescience.org/wiki/ProtoDUNE-SP_datasets);
these list some official dataset definitions.

You can make your own samweb dataset definitions: First, make certain a definition does not already exist that satisfies your needs by checking the official pages above. 

Then check to see what you will get:
```bash
samweb list-files "data_tier full-reconstructed and DUNE.campaign PDSPProd4  and data_stream cosmics and run_type protodune-sp and detector.hv_value 180" –summary
``` 
```
samweb create-definition $USER-PDSPProd4_good_cosmics "data_tier full-reconstructed and DUNE.campaign PDSPProd4  and data_stream cosmics and run_type protodune-sp and detector.hv_value 180"
``` 

Note that the `username` appears in the definition name - to prevent users from getting confused with official samples, your user name is required in the definition name. 
prestaging

At CERN files are either on eos or castor. At FNAL they can be on tape_backed `dcache` which may mean they are on tape and may need to be prestaged to disk before access.
setup fife_utils  # a new version we requested

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

Oops - this one is not on disk 

~~~
returns ONLINE_AND_NEARLINE: 1 if available on disk 
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

No `ONLINE_NEARLINE` means you need to prestage that file. Unfortunately, prestaging requires a definition. Let's find some for run 5141.  Your physics group should already have some defined.

The official Protodune dataset definitions are [here](https://wiki.dunescience.org/wiki/ProtoDUNE-SP_datasets).

```bash
samweb describe-definition PDSPProd4a_MC_1GeV_reco1_sce_datadriven_v1_00
```

Is simulation for 10% of the total sample

Gives this description:

```bash
samweb describe-definition PDSPProd4a_MC_1GeV_reco1_sce_datadriven_v1_00
```
~~~
Definition Name: PDSPProd4a_MC_1GeV_reco1_sce_datadriven_v1_00
  Definition Id: 635109
  Creation Date: 2021-08-02T16:57:20+00:00
       Username: dunepro
          Group: dune
     Dimensions: run_type 'protodune-sp' and file_type mc and data_tier 'full-reconstructed' and dune.campaign PDSPProd4a and dune_mc.beam_energy 1 and
dune_mc.space_charge yes and dune_mc.generators beam_cosmics and version v09_17_01 and run_number in 18800650,.....
~~~
{: .output}

```bash
samweb list-files "defname:PDSPProd4a_MC_1GeV_reco1_sce_datadriven_v1_00
``` 
~~~
> " --summary
File count:	5025
Total size:	9683195368818
Event count:	50250
~~~
{: .output}

```bash
samweb prestage-dataset --def=PDSPProd4a_MC_1GeV_reco1_sce_datadriven_v1_00 --parallel=10
``` 

would prestage all of the reconstructed data for run 5141 and you can check on the status by going [here](http://samweb.fnal.gov:8480/station_monitor/dune/stations/dune/projects) and scrolling down to see your prestage link. 

At CERN

You can find local copies of files at CERN for interactive use. 

```bash
samweb list-file-locations --defname=runset-5141-raw-180kV-7GeV-v0 --schema=root --filter_path=castor 
``` 
gives you:

~~~
root://castorpublic.cern.ch//castor/cern.ch/neutplatform/protodune/rawdata/np04/detector/None/raw/06/60/74/16/np04_raw_run005141_0015_dl3.root	castor:/neutplatform/protodune/rawdata/np04/detector/None/raw/06/60/74/16	np04_raw_run005141_0015_dl3.root	8289321123
root://castorpublic.cern.ch//castor/cern.ch/neutplatform/protodune/rawdata/np04/detector/None/raw/06/60/74/17/np04_raw_run005141_0015_dl10.root	castor:/neutplatform/protodune/rawdata/np04/detector/None/raw/06/60/74/17	np04_raw_run005141_0015_dl10.root	8276312581
~~~
{: .output}


{%include links.md%} 
