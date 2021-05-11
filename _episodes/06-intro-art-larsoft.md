---
title: Introduction to art and LArSoft
teaching: 90
exercises: 0
questions:
- Why do we need a complicated software framework? Can't I just write standalone code?
objectives:  
- Learn what services the art framework provides.
- Learn how the LArSoft tookit is organized and how to use it.
keypoints:
- Art provides the tools physicists in a large collaboration need in order to contribute software to a large, shared effort without getting in each others' way.
- Art helps us keep track of our data and job configuration, reducing the chances of producing mystery data that no one knows where it came from.
- LArSoft is a set of simulation and reconstruction tools shared among the liquid-argon TPC collaborations.
---

## Introduction to *art*

*Art* is the framework used for the offline software used to process LArTPC data from the far detector and the ProtoDUNEs. It was chosen not only because of the features it provides, but also because it allows DUNE to use and share algorithms developed for other LArTPC experiments, such as ArgoNeuT, MicroBooNE and ICARUS. The section below describes LArSoft, a shared software toolkit. Art is also used by the NOvA and mu2e experiments. The primary language for *art* and experiment-specific plug-ins is C++.

The *art* wiki page is here: [https://cdcvs.fnal.gov/redmine/projects/art/wiki][art-wiki]. It contains important information on command-line utilities, how to configure an art job, how to define, read in and write out data products, how and when to use art modules, services, and tools.

*Art* features:

1. Defines the event loop
2. Manages event data storage memory and prevents unintended overwrites
3. Input file interface -- allows ganging together input files
4. Schedules module execution
5. Defines a standard way to store data products in art-formatted ROOT files
6. Defines a format for associations between data products (for example, tracks have hits, and associations between  tracks and hits can be made via art's association mechanism.
7. Provides a uniform job configuration interface
8. Stores job configuration information in art-formatted root files.
9. Output file control -- lets you define output filenames based on parts of the input filename.
10. Message handling
11. Random number control
12. Exception handling

The configuration storage is particularly useful if you receive a data file from a colleague, or find one in a data repository and you want to know more about how it was produced, with what settings.

### Getting set up to try the tools

Log in to a `dunegpvm*.fnal.gov` machine and set up your environment:

```bash
source ~/.dune_presetup_202105.sh
dune_setup
setup dunetpc $DUNETPC_VERSION -q e19:prof
setup_fnal_security
```


The examples below will refer to files in `dCache` at Fermilab which can best be accessed via `xrootd`. 

**For those with no access to Fermilab computing resources but with a CERN account:**  
Copies are stored in `/afs/cern.ch/work/t/tjunk/public/may2021tutorialfiles/`.

The follow-up of this tutorial provides help on how to find data and MC files in storage.

You can list available versions of `dunetpc` installed in `CVMFS` with this command:

```bash
ups list -aK+ dunetpc
```

The output is not sorted, although portions of it may look sorted. Do not depend on it being sorted. The string indicating the version is called the version tag (v09_22_02 here). The qualifiers are e19 and prof. Qualifiers can be entered in any order and are separated by colons. "e19" refers to a specific version of the gnu compiler collection (v8_2_0). We will soon be moving everything to "e20" which corresponds to `gcc v9_3_0`. We also compile with `clang` -- the compiler qualifier for that is "c7".

"prof" means "compiled with optimizations turned on." "debug" means "compiled with optimizations turned off". More information on qualifiers is [here][about-qualifiers].

In addition to the version and qualifiers, `UPS` products have "flavors". This refers to the operating system and version. Older versions of `dunetpc` and `larsoft` supported `SL6` and some versions of macOS. Currently only SL7 and the compatible CentOS 7 are supported. The flavor of a product is automatically selected to match your current operating system when you set up a product. If a flavor is missing, you will get an error message.

There is a setup command provided by the operating system -- you usually don't want to use it (at least not when developing DUNE software). If you haven't sourced the `setup_dune.sh` script in `CVMFS` above and type `setup xyz`, you will get the system setup command, which will ask you for the root password. Just `control-C` out of it, source the `setup_dune.sh` script, and try again.

UPS's setup command (find out where it lives with this command):

```bash
type setup
```

will not only set up the product you specify (in the instructions above, dunetpc), but also all dependent products with corresponding versions so that you get a consistent software environment. You can get a list of everything that's set up with this command

```bash
 ups active
```

It is often useful to pipe the output through grep to find a particular product.

```bash
 ups active | grep geant4
```

for example, to see what version of geant4 you have set up.

### *Art* command-line tools

All of these command-line tools have online help. Invoke the help feature with the `--help` command-line option. Example:

```bash
config_dumper --help
 ```

Docmentation on art command-line tools is available on the [art wiki page][art-wiki].

#### config_dumper

Configuration information for a file can be printed with config_dumper. 

```bash
config_dumper -P <artrootfile>
```

Try it out:
```bash
config_dumper -P root://fndca1.fnal.gov:1094/pnfs/fnal.gov/usr/dune/tape_backed/dunepro/protodune-sp/full-reconstructed/2021/mc/out1/PDSPProd4/40/57/23/91/PDSPProd4_protoDUNE_sp_reco_stage1_p1GeV_35ms_sce_datadriven_41094796_0_20210121T214555Z.root
```

The output is an executable `fcl` file, sent to stdout. We recommend redirecting the output to a file that you can look at in a text editor:

Try it out:
```bash
config_dumper -P root://fndca1.fnal.gov:1094/pnfs/fnal.gov/usr/dune/tape_backed/dunepro/protodune-sp/full-reconstructed/2021/mc/out1/PDSPProd4/40/57/23/91/PDSPProd4_protoDUNE_sp_reco_stage1_p1GeV_35ms_sce_datadriven_41094796_0_20210121T214555Z.root > tmp.fcl
```

Your shell may be configured with `noclobber`, meaning that if you already have a file called `tmp.fcl`, the shell will refuse to overwrite it. Just `rm tmp.fcl` and try again.

The `-P` option to `config_dumper` is needed to tell `config_dumper` to print out all processing configuration `fcl` parameters. The default behavior of `config_dumper` prints out only a subset of the configuration parameters, and is most notably missing art services configuration.


> ## Quiz 
>
> Quiz questions from the output of the above run of `config_dumper`:
>
>  1.  What generators were used?  What physics processes are simulated in this file?
>  2.  What geometry is used?  (hint:  look for "GDML" or "gdml")
>  3.  What electron lifetime was assumed?
>  4.  What is the readout window size? 
> 
{: .solution}

 
#### fhicl-dump

You can parse a `FCL` file with `fhicl-dump`. 

Try it out:
```bash
fhicl-dump protoDUNE_refactored_g4_stage2.fcl
```

See the section below on `FCL` files for more information on what you're looking at.

#### count_events

Try it out:
```bash
>count_events root://fndca1.fnal.gov:1094/pnfs/fnal.gov/usr/dune/tape_backed/dunepro/protodune-sp/full-reconstructed/2021/mc/out1/PDSPProd4/40/57/23/91/PDSPProd4_protoDUNE_sp_reco_stage1_p1GeV_35ms_sce_datadriven_41094796_0_20210121T214555Z.root
```

#### product_sizes_dumper

You can get a peek at what's inside an artroot file with `product_sizes_dumper`.

Try it out:
```bash
product_sizes_dumper -f 0 root://fndca1.fnal.gov:1094/pnfs/fnal.gov/usr/dune/tape_backed/dunepro/protodune-sp full-reconstructed/2021/mc/out1/PDSPProd4/40/57/23/91/PDSPProd4_protoDUNE_sp_reco_stage1_p1GeV_35ms_sce_datadriven_41094796_0_20210121T214555Z.root
```

It is also useful to redirect the output of this command to a file so you can look at it with a text editor and search for items of interest. This command lists the sizes of the `TBranches` in the `Events TTree` in the `artroot` file. There is one `TBranch` per data product, and the name of the `TBranch` is the data product name, an "s" is appended (even if the plural of the data product name doesn't make sense with just an "s" on the end), an underscore, then the module label that made the data product, an underscore, the instance name, an underscore, and the process name and a period.


Quiz questions, looking at the output from above.

> ## Quiz
> Questions:
> 1.  What is the name of the data product that takes up the most space in the file?
> 2.  What the module label for this data product?
> 3.  What is the module instance name for this data product?   (tricky.  You have to count underscores here).
> 4.  How many different modules produced simb::MCTruth data products?  What are their module labels?
> 5.  How many different modules produced recob::Hit data products?  What are their module labels?
{: .solution}

You can open up an artoot file with `ROOT` and browse the `TTrees` in it with a `TBrowser`. Not all `TBranches` and leaves can be inspected easily this way, but enough can that it can save a lot of time programming if you just want to know something simple about a file such as whether it contains a particular data product and how many there are. 

Try it out
```bash
root root://fndca1.fnal.gov:1094/pnfs/fnal.gov/usr/dune/tape_backed/dunepro/protodune-sp/full-reconstructed/2021/mc/out1/PDSPProd4/40/57/23/91/PDSPProd4_protoDUNE_sp_reco_stage1_p1GeV_35ms_sce_datadriven_41094796_0_20210121T214555Z.root
```
then at the `root` prompt, type:
```bash
new TBrowser
``` 

This will be faster with `VNC`. Navigate to the `Events TTree` in the file that is automatically opened, navigate to the `TBranch` with the Argon 39 MCTruths (it's near the bottom), click on the branch icon `simb::MCTruths_ar39__SinglesGen.obj`, and click on the `NParticles()` leaf (It's near the bottom. Yes, it has a red exclamation point on it, but go ahead and click on it). How many events are there? How many 39Ar decays are there per event on average?

`Art` is not constrained to using `ROOT` files -- some effort has already been underway to use HDF5-formatted files for some purposes.

The `art` main executable program is a very short stub that interprets command-line options, reads in the configuration document (a `FHiCL` file which usually includes other `FHiCL` files), and loads shared libraries, initializes software components, and schedules execution of modules. Most code we are interested in is in the form of art plug-ins -- modules, services, and tools. The generic executable for invoking art is called `art`, but a LArSoft-customized one is called lar. No additional customization has yet been applied so in fact, the lar executable has identical functionality to the `art` executable.

There is online help:

```bash
 lar --help
 ```

All programs in the art suite have a `--help` command-line option.

Most `art` job invocations take the form 

```bash
lar -n <nevents> -c fclfile.fcl artrootfile.root
``` 

where the input file specification is just on the command line without a command-line option. Explicit examples follow below. The `-n <nevents>` is optional -- it specifies the number of events to process. If omitted, or if `<nevents>` is bigger than the number of events in the input file, the job processes all of the events in the input file. `-n <nevents>` is important for the generator stage. There's also a handy `--nskip <nevents_to_skip>` argument if you'd like the job to start processing partway through the input file. You can steer the output with

```bash
lar -c fclfile.fcl artrootfile.root -o outputartrootfile.root -T outputhistofile.root
```


The `outputhistofile.root` file contains `ROOT` objects that have been declared with the `TFileService` service in user-supplied art plug-in code (i.e. your code).

### Job configuration with FHiCL

The Fermilab Hierarchical Configuration Language, FHiCL is described  here [https://cdcvs.fnal.gov/redmine/documents/327][fhicl-described].

It is **not** a Turing-complete language: you cannot write an executable program in it. It is meant to declare values for named parameters to steer job execution and adjust algorithm parameters (such as the electron lifetime in the simulation and reconstruction). Look at `.fcl` files in installed job directories, like `$DUNETPC_DIR/job` for examples. `Fcl` files are sought in the directory seach path `FHICL_FILE_PATH` when art starts up and when `#include` statements are processed. A fully-expanded `fcl` file with all the #include statements executed is referred to as a fhicl "document".

Parameters may be defined more than once. The last instance of a parameter definition wins out over previous ones. This makes for a common idiom in changing one or two parameters in a fhicl document. The generic pattern for making a short fcl file that modifies a parameter is:

~~~
#include "fcl_file_that_does_almost_what_I_want.fcl"
block.subblock.parameter: new_value
~~~

To see what block and subblock a parameter is in, use `fhcl-dump on` the parent fcl file and look for the curly brackets. You can also use

```bash
lar -c fclfile.fcl --debug-config tmp.txt --annotate
``` 

which is equivalent to `fhicl-dump` with the --annotate option and piping the output to tmp.txt.

Entire blocks of parameters can be substituted in using `@local` and `@table` idioms. See the examples and documentation for guidance on how to use these. Generally they are defined in the PROLOG sections of fcl files. PROLOGs must precede all non-PROLOG definitions and if their symbols are not subsequently used they do not get put in the final job configuration document (that gets stored with the data and thus may bloat it). This is useful if there are many alternate configurations for some module and only one is chosen at a time.


Try it out:
```bash
fhicl-dump protoDUNE_refactored_g4_stage2.fcl > tmp.txt
``` 

Look for the parameter `ModBoxA`. It is one of the Modified Box Model ionization parameters. See what block it is in. Here are the contents of a modified g4 stage 2 fcl file that modifies just that parameter:

~~~
#include "protoDUNE_refactored_g4_stage2.fcl"
services.LArG4Parameters.ModBoxA: 7.7E-1
~~~
{: .source}

> ## Exercise
> Do a similar thing -- modify the stage 2 g4 fcl configuration to change the drift field from 486.7 V/cm to 500 V/cm. Hint -- you will find the drift field in an array of fields which also has the fields between wire planes listed.
{: .challenge}


### Types of Plug-Ins

Plug-ins each have their own .so library which gets dynamically loaded by art when referenced by name in the fcl configuration.

**Producer Modules**  
A producer module is a software component that writes data products to the event memory. It is characterized by produces<> and consumes<> statements in the class constructor, and `art::Event::put()` calls in the `produces()` method. A producer must produce the data product collection it says it produces, even if it is empty, or art will throw an exception. `art::Event::put()` transfers ownership of memory (use std::move so as not to copy the data) from the module to the art event memory. Data in the art event memory will be written to the output file unless output commands in the fcl file tell art not to do that. Documentation on output commands can be found in the LArSoft wiki here: [https://cdcvs.fnal.gov/redmine/projects/larsoft/wiki/Rerun_part_of_all_a_job_on_an_output_file_of_that_job][larsoft-rerun-part-job] Producer modules have methods that are called on begin job, begin run, begin subrun, and on each event, as well as at the end of processing, so you can initialize counters or histograms, and finish up summaries at the end. Source code must be in files of the form: <modulename>_module.cc, where modulename does not have any underscores in it.

**Analyzer Modules**  
Analyzer modules read data product from the event memory and produce histograms or TTrees, or other output. They are typically scheduled after the producer modules have been run. Producer modules have methods that are called on begin job, begin run, begin subrun, and on each event, as well as at the end of processing, so you can initialize counters or histograms, and finish up summaries at the end. Source code must be in files of the form: <modulename>_module.cc, where modulename does not have any underscores in it.

**Source Modules**  
Source modules read data from input files and reformat it as need be, in order to put the data in art event data store. Most jobs use the art-provided RootInput source module which reads in art-formatted ROOT files. RootInput interacts well with the rest of the framework in that it provides lazy reading of TTree branches. Only when GetByLabel or GetValidHandle or other product get methods are called is the data actually fetched from the input file. This is useful for art jobs that only read a subset of the TBranches in an input file. Source code must be in files of the form: <modulename>_source.cc, where modulename does not have any underscores in it.

**Services**  
These are singleton classes that are globally visible within an art job. They can be FHiCL configured like modules, and they can schedule methods to be called on begin job, begin run, begin event, etc. They are meant to help supply configuration parameters like the drift velocity, or more complicated things like geometry functions, to modules that need them. Please do not use services as a back door for storing event data outside of the art event store. Source code must be in files of the form: `<modulename>_service.cc`, where servicename does not have any underscores in it.

**Tools**  
Tools are FHiCL-configurable software components that are not singletons, like services. They are meant to be swappable by FHiCL parameters which tell art which .so libraries to load up, configure, and call from user code. See the [Art Wiki Page][art-wiki-redmine] for more information on tools and other plug-ins.

You can use cetskelgen to make empty skeletons of art plug-ins. See the art wiki for documentation, or use

```bash
cetskelgen --help
``` 

for instructions on how to invoke it.

### Non-Plug-In Code

You are welcome to write standard C++ code -- classes and C-style functions are no problem. In fact, to enhance the portability of code, the art team encourages the separation of algorithm code into non-framework-specific source files, and to call these functions or class methods from the art plug-ins. Typically, source files for standalone algorithm code have the extension .cxx while art plug-ins have .cc extensions. Most directories have a CMakeLists.txt file which has instructions for building the plug-ins, each of which is built into a .so library, and all other code gets built and put in a separate .so library.

### Retrieving Data Products

In a producer or analyzer module, data products can be retrieved from the art event store with `getByLabel()` or `getValidHandle()` calls, or more rarely `getManyByType` or other calls. The arguments to these calls specify the module label and the instance of the data product. A typical `TBranch` name in the Events tree in an artroot file is

~~~
simb::MCParticles_largeant__G4Stage1.
~~~
{:. source}

here, `simb::MCParticle` is the name of the class that defines the data product. The "s" after the data product name is added by *art* -- you have no choice in this even if the plural of your noun ought not to just add an "s". The underscore separates the data product name from the module name, "largeant". Another underscore separates the module name and the instance name, which in this example is the empty string -- there are two underscores together there. The last string is the process name and usually is not needed to be specified in data product retrieval. You can find the `TBranch` names by browsing an artroot file with `ROOT` and using a `TBrowser`, or by using `product_sizes_dumper -f 0`.

### Art documentation

There is a mailing list -- `art-users@fnal.gov` where users can ask questions and get help.

There is a workbook for art available at [https://art.fnal.gov/art-workbook/][art-workbook] Look for the "versions" link in the menu on the left for the actual document. It is a few years old and is missing some pieces like how to write a producer module, but it does answer some questions. I recommend keeping a copy of it on your computer and using it to search for answers.

There was an [art/LArSoft course in 2015][art-LArSoft-2015]. While it, too is a few years old, the examples are quite good and it serves as a useful reference.

## Gallery

Gallery is a lightweight tool that lets users read art-formatted root files and make plots without having to write and build art modules. It works well with interpreted and compiled ROOT macros, and is thus ideally suited for data exploration and fast turnaround of making plots. It lacks the ability to use art services, however, though some LArSoft services have been split into services and service providers. The service provider code is intended to be able to run outside of the art framework and linked into separate programs.

Gallery also lacks the ability to write data products to an output file. You are of course free to open and write files of your own devising in your gallery programs. There are example gallery ROOT scripts in dunetpc/dune/GalleryScripts. They are only in the git repository but do not get installed in the UPS product.

More documentation: [https://art.fnal.gov/gallery/][art-more-documentation]

## LArSoft

### Introductory Documentation

Using LArSoft: [https://cdcvs.fnal.gov/redmine/projects/larsoft/wiki/Using_LArSoft][using-larsoft].

### Software structure

The LArSoft toolkit is a set of software components that simulate and reconstruct LArTPC data, and also it provides tools for accessing raw data from the experiments. LArSoft contains an interface to GEANT4 (art does not list GEANT4 as a dependency) and the GENIE generator. It contains geometry tools that are adapted for wire-based LArTPC detectors.

A recent graph of the UPS products in a full stack staring with protduneana is available here: [https://wiki.dunescience.org/w/img_auth.php/8/87/Protoduneana_v09_22_02_e19_prof_graph.pdf][protoduneana] You can see the LArSoft pieces under dunetpc, as well as GEANT4, GENIE, ROOT, and a few others.

### LArSoft Data Products

A very good introduction to data products such as raw digits, calibrated waveforms, hits and tracks, that are created and used by LArSoft modules and usable by analyzers was given by Tingjun Yang at the 2019 ProtoDUNE analysis workshop: [https://indico.fnal.gov/event/19133/contributions/50492/attachments/31462/38611/dataproducts.pdf][larsoft-data-products].

There are a number of data product dumper fcl files. A non-exhaustive list of useful examples is given below:

~~~
 dump_mctruth.fcl
 dump_mcparticles.fcl
 dump_simenergydeposits.fcl
 dump_simchannels.fcl
 dump_simphotons.fcl
 dump_rawdigits.fcl
 dump_wires.fcl
 dump_hits.fcl
 dump_clusters.fcl
 dump_tracks.fcl
 dump_pfparticles.fcl
 eventdump.fcl
 dump_lartpcdetector_channelmap.fcl
 dump_lartpcdetector_geometry.fcl
~~~
{: .source}

Some of these may require some configuration of input module labels so they can find the data products of interest.

Some of these may require some configuration of input module labels so they can find the data products of interest. Try one of these yourself:

```bash 
lar -n 1 -c dump_mctruth.fcl root://fndca1.fnal.gov:1094/pnfs/fnal.gov/usr/dune/tape_backed/dunepro/protodune-sp/full-reconstructed/2021/mc/out1/PDSPProd4/40/57/23/91/PDSPProd4_protoDUNE_sp_reco_stage1_p1GeV_35ms_sce_datadriven_41094796_0_20210121T214555Z.root
``` 

This command will make a file called `DumpMCTruth.log` which you can open in a text editor. Reminder: `MCTruth` are particles made by the generator(s), and MCParticles are those made by GEANT4, except for those owned by the `MCTruth` data products. Due to the showering nature of LArTPCs, there are usually many more MCParticles than MCTruths.

### Examples and current workflows

The page with instructions on how to find and look at ProtoDUNE data has links to standard fcl configurations for simulating and reconstructing ProtoDUNE data: [https://wiki.dunescience.org/wiki/Look_at_ProtoDUNE_SP_data][look-at-protodune].

Try it yourself! The workflow for ProtoDUNE-SP MC is given in the Simulation Task Force web page. [https://wiki.dunescience.org/wiki/ProtoDUNE-SP_Simulation_Task_Force][protodune-sim-task-force] 


#### Running on a dunegpvm machine at Fermilab

~~~
export USER=`whoami`
 mkdir -p /dune/data/users/$USER/tutorialtest
 cd /dune/data/users/$USER/tutorialtest
 source /cvmfs/dune.opensciencegrid.org/products/dune/setup_dune.sh
 setup dunetpc v09_22_02 -q e19:prof
 lar -n 1 -c mcc12_gen_protoDune_beam_cosmics_p1GeV.fcl -o gen.root
 lar -n 1 -c protoDUNE_refactored_g4_stage1.fcl gen.root -o g4_stage1.root
 lar -n 1 -c protoDUNE_refactored_g4_stage2_sce_datadriven.fcl g4_stage1.root -o g4_stage2.root
 lar -n 1 -c protoDUNE_refactored_detsim_stage1.fcl g4_stage2.root -o detsim_stage1.root
 lar -n 1 -c protoDUNE_refactored_detsim_stage2.fcl detsim_stage1.root -o detsim_stage2.root
 lar -n 1 -c protoDUNE_refactored_reco_35ms_sce_datadriven_stage1.fcl detsim_stage2.root -o reco_stage1.root
 lar -c eventdump.fcl reco_stage1.root >& eventdump_output.txt
 config_dumper -P reco_stage1.root >& config_output.txt
 product_sizes_dumper -f 0 reco_stage1.root >& productsizes.txt

~~~
{: .source}

#### Running at CERN

This example puts all files in a subdirectory of your home directory. There is an input file for the ProtoDUNE-SP beamline simulation that is copied over and you need to point the generation job at it. The above sequence of commands will work at CERN if you have a Fermilab grid proxy, but not everyone signed up for the tutorial can get one of these yet, so we copied the necessary file over and adjusted a fcl file to point at it. It also runs faster with the local copy of the input file than the above workflow which copies it.

```bash 
 cd ~
 mkdir May2021Tutorial
 cd May2021Tutorial
 source /cvmfs/dune.opensciencegrid.org/products/dune/setup_dune.sh
 setup dunetpc v09_22_02 -q e19:prof
 cat > tmpgen.fcl << EOF
 #include "mcc12_gen_protoDune_beam_cosmics_p1GeV.fcl"
 physics.producers.generator.FileName: "/afs/cern.ch/work/t/tjunk/public/may2021tutorialfiles/H4_v34b_1GeV_-27.7_10M_1.root"
 EOF
 lar -n 1 -c tmpgen.fcl -o gen.root
 lar -n 1 -c protoDUNE_refactored_g4_stage1.fcl gen.root -o g4_stage1.root
 lar -n 1 -c protoDUNE_refactored_g4_stage2_sce_datadriven.fcl g4_stage1.root -o g4_stage2.root
 lar -n 1 -c protoDUNE_refactored_detsim_stage1.fcl g4_stage2.root -o detsim_stage1.root
 lar -n 1 -c protoDUNE_refactored_detsim_stage2.fcl detsim_stage1.root -o detsim_stage2.root
 lar -n 1 -c protoDUNE_refactored_reco_35ms_sce_datadriven_stage1.fcl detsim_stage2.root -o reco_stage1.root
 lar -c eventdump.fcl reco_stage1.root >& eventdump_output.txt
 config_dumper -P reco_stage1.root >& config_output.txt
 product_sizes_dumper -f 0 reco_stage1.root >& productsizes.txt
 ``` 

You can also browse the root files with a TBrowser or run other dumper fcl files on them. The dump example commands above redirect their outputs to text files which you can edit with a text editor or run grep on to look for things.

You can run the event display with

```bash 
lar -c evd_protoDUNE.fcl reco_stage1.root
``` 

but it will run very slowly over a tunneled X connection. A VNC session will be much faster. Tips: select the "Reconstructed" radio button at the bottom and click on "Unzoom Interest" on the left to see the reconstructed objects in the three views.


### DUNE software documentation and how-to's

This wiki page provides a lot of information on how to check out, build, and contribute to dune-specific larsoft plug-in code.

[https://cdcvs.fnal.gov/redmine/projects/dunetpc/wiki][dunetpc-wiki]

The follow-up part of this tutorial gives hands-on exercises for doing these things.

### Contributing to LArSoft

Unlike dunetpc, protoduneana and garsoft, which are hosted in Fermilab's Redmine repository, the LArSoft git repositories are hosted on GitHub and use a pull-request model. LArSoft's github link is [https://github.com/larsoft][github-link]

See the documentation at this link: [https://cdcvs.fnal.gov/redmine/projects/larsoft/wiki/Developing_With_LArSoft][developing-with-larsoft]

There are bi-weekly LArSoft coordination meetings [https://indico.fnal.gov/category/405/][larsoft-meetings] at which stakeholders, managers, and users discuss upcoming releases, plans, and new features to be added to LArSoft.

### Useful tip: check out an inspection copy of larsoft

A good old-fashioned `grep -r` or a find command can be effective if you are looking for an example of how to call something but I do not know where such an example might live. The copies of LArSoft source in CVMFS lack the CMakeLists.txt files and if that's what you're looking for to find examples, it's good to have a copy checked out. Here's a script that checks out all the LArSoft source and DUNE LArSoft code but does not compile it. Warning: it deletes a directory called "inspect" in your app area. Make sure `/dune/app/users/<yourusername>` exists first:

~~~
 #!/bin/sh
 USERNAME=`whoami`
 LARSOFT_VERSION=v09_13_00
 COMPILER=e19
 source /cvmfs/dune.opensciencegrid.org/products/dune/setup_dune.sh
 cd /dune/app/users/${USERNAME}
 rm -rf inspect
 mkdir inspect
 cd inspect
 setup larsoft ${LARSOFT_VERSION} -q debug:${COMPILER}
 mrb newDev
 source /dune/app/users/${USERNAME}/inspect/localProducts_larsoft_${LARSOFT_VERSION}_debug_${COMPILER}/setup
 cd srcs
 mrb g larsoft_suite
 mrb g larsoftobj_suite
 mrb g larutils
 mrb g larbatch
 mrb g dunetpc
 mrb g duneutil
 mrb g -d dune_raw_data dune-raw-data
~~~
{: .language-bash}

Putting it to use: A very common workflow in developing software is to look for an example of how to do something similar to what you want to do. Let's say you want to find some examples of how to use `FindManyP` -- it's an *art* method for retrieving associations between data products, and the art documentation isn't anywhere near as good as the examples for learning how to use it. You can use a recursive grep through your checked-out version, or you can even look through the installed source in CVMFS:

```bash 
 cd $DUNETPC_DIR/source/dune
 grep -r -i findmanyp *
 ``` 
 
It is good to use the `-i` option to grep which tells it to ignore the difference between uppercase and lowercase string matches, in case you misremembered the case of what you are looking for. The list of matches is quite long -- you may want to pipe the output of that grep into another grep

```bash 
 grep -r -i findmanyp * | grep recob::Hit
``` 
 
The checked-out versions of the software have the advantage of providing some files that don't get installed in CVMFS, notably CMakeLists.txt files and the UPS product_deps files, which you may want to examine when looking for examples of how to do things.

## GArSoft

GArSoft is another art-based software package, designed to simulate the ND-GAr near detector. Many components were copied from LArSoft and modified for the pixel-based TPC with an ECAL. You can find installed versions in CVMFS with the following command:

~~~
ups list -aK+ garsoft
~~~
{:. source}

and you can check out the source and build it by following the instructions on the GArSoft wiki: [https://cdcvs.fnal.gov/redmine/projects/garsoft/wiki][garsoft-wiki]

{%include links.md%} 


[about-qualifiers]: https://cdcvs.fnal.gov/redmine/projects/cet-is-public/wiki/AboutQualifiers
[art-wiki]: https://cdcvs.fnal.gov/redmine/projects/art/wiki
[larsoft-rerun-part-job]: https://cdcvs.fnal.gov/redmine/projects/larsoft/wiki/Rerun_part_of_all_a_job_on_an_output_file_of_that_job
[github-link]: https://github.com/larsoft
[protodune-sim-task-force]: https://wiki.dunescience.org/wiki/ProtoDUNE-SP_Simulalation_Task_Force
[larsoft-meetings]: https://indico.fnal.gov/category/405/][larsoft-meetings
[developing-with-larsoft]: https://cdcvs.fnal.gov/redmine/projects/larsoft/wiki/Developing_With_LArSoft
[fhicl-described]: https://cdcvs.fnal.gov/redmine/documents/327
[garsoft-wiki]: https://cdcvs.fnal.gov/redmine/projects/garsoft/wiki
[art-wiki-redmine]: https://cdcvs.fnal.gov/redmine/projects/art/wiki#How-to-use-the-modularity-of-art
[art-more-documentation]: https://art.fnal.gov/gallery/][art-more-documentation
[using-larsoft]: https://cdcvs.fnal.gov/redmine/projects/larsoft/wiki/Using_LArSoft
[protoduneana]:https://wiki.dunescience.org/w/img_auth.php/f/fd/Protoduneana_v09_12_00_e19-prof_graph.pdf
[larsoft-data-products]: https://indico.fnal.gov/event/19133/contributions/50492/attachments/31462/38611/dataproducts.pdf
[dunetpc-wiki]: https://cdcvs.fnal.gov/redmine/projects/dunetpc/wiki
[look-at-protodune]: https://wiki.dunescience.org/wiki/Look_at_ProtoDUNE_SP_data
[art-LArSoft-2015]: https://indico.fnal.gov/event/9928/timetable/?view=standard
[art-workbook]: https://art.fnal.gov/art-workbook/
