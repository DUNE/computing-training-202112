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

## Introduction

DUNE data is stored around the world and the storage elements are not always organized in a way that they can be easily inspected. For this purpose we use the SAM web client.

### What is SAM?  
The Sequential Access Model (SAM) is a data handling system developed at Fermilab.  It is designed to tracklocations of files and other file metadata.

This lecture will show you how to access data files that have been defined to the DUNE Data Catalog. Execute the following commands after logging in to the DUNE interactive node, and sourcing the main dune setups.

Once per session:
~~~
setup sam_web_client
export SAM_EXPERIMENT=dune
~~~

### What is Rucio?
Rucio is the next-generation Data Replica service and is part of DUNE's new Distributed Data Management (DDM) system that is currently in deployment. 
Rucio has two functions:  (1) A rule-based system to get files to Rucio Storage Elements around the world and keep them there and (2) To return the
"nearest" replica of any data file for use either in interactive or batch file use.  It is expected that most DUNE users will not be regularly using
direct Rucio commands, but other wrapper scripts that calls them indirectly.  
As of the date of this May 2021 tutorial (a) the Rucio client is not installed as a part of the standard DUNE client software
and (b) most DUNE users are not yet enabled to use it.  But when we do, some of the commands will look like this:

~~~
rucio list-file-replicas protodune-sp:np04_raw_run005801_0001_dl1.root

rucio download protodune-sp:np04_raw_run005801_0001_dl1.root

rucio list-rses
~~~

## Finding data

If you know a given file and want to locate it, e.g.:
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
which is the locations of the file on disk and tape. We can use this to copy the file from tape to our local disk.

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

To see the total number of files that match a certain query expression, then add the `--summary` option to `samweb list-files`.

`samweb` allows you to select on a lot of parameters which are documented here:

* [Useful ProtoDUNE samweb parameters][useful-samweb]

* [dune-data.fnal.gov](https://dune-data.fnal.gov) lists some official dataset definitions 

## Accessing data for use in your analysis
To access data without copying it, XRootD is the tool to use. However it will work only if the file is staged to the disk.

You can stream files worldwide if you have a DUNE VO certificate as described in the preparation part of this tutorial.

### Where _is_ the file?

An example to find a given file:  
~~~
samweb get-file-access-url np04_raw_run005758_0001_dl3_reco_13600804_0_20181127T081955.root --schema=root
~~~
{: .language-bash}
~~~
root://fndca1.fnal.gov:1094/pnfs/fnal.gov/usr/dune/tape_backed/dunepro/protodune/np04/beam/output/detector/full-reconstructed/08/61/68/00/np04_raw_run005758_0001_dl3_reco_13600804_0_20181127T081955.root
~~~
{: .output}

**Resource**: [Using the SAM Data Catalog][sam-data-control].
> ## Exercise 1
> * Use the `--location` argument to show the path of the file above on either `enstore`, `castor` or `cern-eos`.
> * Use `get-metadata` to get SAM metadata for this file. Note that `--json` gives the output in json format.
{: .challenge}


When we are analyzing large numbers of files in a group of batch jobs, we use a SAM snapshot to describe the full set of files that we are going to analyze and create a SAM Project based on that. Each job will then come up and ask SAM to give it the next file in the list. SAM has some capability to grab the nearest copy of the file. For instance if you are running at CERN and analyzing this file it will automatically take it from the CERN storage space EOS.

> ## Exercise 2
> todo!
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
> todo!
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

### UPS basic commands

| Command                                        | Action                                                           |
|------------------------------------------------|------------------------------------------------------------------|
| `ups list -aK+ dunetpc`                        | List the versions and flavors of dunetpc that exist on this node |
| `ups active`                                   | Displays what has been setup                                     |
| `ups depend dunetpc v08_57_00 -q e19:prof:py2` | Displays the dependencies for this version of dunetpc            |

> ## Excercise 4
> todo!
{: .challenge}

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
> todo!
{: .challenge}


## CVMFS  
**What is CVMFS and why do we need it?**  
DUNE has a need to distribute precompiled code to many different computers that collaborators may use. Installed products are needed for four things: 
1. Running programs interactively
2. Running programs on grid nodes
3. Linking programs to installed libraries
4. Inspection of source code and data files

Results must be reproducible, so identical code and associated files must be distributed everywhere. DUNE does not own any batch resources -- we use CPU time on computers that participating institutions donate to the Open Science Grid. We are not allowed to install our software on these computers and must return them to their original state when our programs finish running so they are ready for the next job from another collaboration.

CVMFS is a perfect tool for distributing software and related files. It stands for CernVM File System (VM is Virtual Machine). Local caches are provided on each target computer, and files are accessed via the `/cvmfs` mount point. DUNE software is in the directory `/cvmfs/dune.opensciencegrid.org`, and LArSoft code is in `/cvmfs/larsoft.opensciencegrid.org`. These directories are auto-mounted and need to be visible when one executes `ls /cvmfs` for the first time.

When a file is accessed in `/cvmfs`, a daemon on the target computer wakes up and determines if the file is in the local cache, and delivers it if it is. If not, the daemon contacts the CVMFS repository server responsible for the directory, and fetches the file into local cache. In this sense, it works a lot like AFS. But it is a read-only filesystem on the target computers, and files must be published on special CVMFS publishing servers. Files may also be cached in a layer between the CVMFS host and the target node in a squid server, which helps facilities with many batch workers reduce the network load in fetching many copies of the same file, possibly over an international connection.

More information on CVMFS is available [here](https://wiki.dunescience.org/wiki/DUNE_Computing/Access_files_in_CVMFS)

> ## Exercise 6
> todo!
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
