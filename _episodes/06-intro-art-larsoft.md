---
title: Introduction to art and LArSoft
teaching: 90
exercises: 0
questions:
- “The question to answer"
objectives:  
- "objective 1"
- "objective 2"
keypoints:
- “keypoint 1"
- "keypoint 2"
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

### *Art* command-line tools

Configuration information for a file can be printed with

~~~
config_dumper -P <artrootfile>
~~~
{: .source}

and you can also parse a FCL file with

~~~
fhicl-dump <fclfile>
~~~
{:. source}

You can get a peek at what's inside an artroot file with

~~~
product_sizes_dumper -f 0 <artrootfile>
~~~
{:. source}

Or, you can open up an artoot file with `ROOT` and browse the `TTrees` in it with a `TBrowser`. The most interesting piece is the `Events TTree` inside of it. Not all `TBranches` and leaves can be inspected easily this way, but enough can that it can save a lot of time programming if you just want to know something simple about a file such as whether it contains a particular data product and how many there are. *Art* is not constrained to using `ROOT` files -- some effort has already been underway to use HDF5-formatted files for some purposes.

The *art* main executable program is a very short stub that interprets command-line options, reads in the configuration document (a FHiCL file which usually includes other FHiCL files), and loads shared libraries, initializes software components, and schedules execution of modules. Most code we are interested in is in the form of art plug-ins -- modules, services, and tools. The generic executable for invoking *art* is called *art*, but a LArSoft-customized one is called lar. No additional customization has yet been applied so in fact, the lar executable has identical functionality to the art executable.

There is online help:

~~~
lar --help
~~~
{:. source}

All programs in the art suite have a --help command-line option.

Most *art* job invocations take the form

~~~
lar -n <nevents> -c fclfile.fcl artrootfile.root
~~~
{:. source}

where the input file specification is just on the command line without a command-line option. The `-n <nevents>` is optional -- it specifies the number of events to process. If omitted, or if <nevents> is bigger than the number of events in the input file, the job processes all of the events in the input file. `-n <nevents>` is important for the generator stage. There's also a handy `--nskip <nevents_to_skip>` argument if you'd like the job to start processing partway through the input file. You can steer the output with

~~~
lar -c fclfile.fcl artrootfile.root -o outputartrootfile.root -T outputhistofile.root
~~~
{:. source}

The outputhistofile.root file contains ROOT objects that have been declared with the TFileService service in user-supplied art plug-in code (i.e. your code).

### Job configuration with FHiCL

The Fermilab Hierarchical Configuration Language, FHiCL is described  here [https://cdcvs.fnal.gov/redmine/documents/327][fhicl-described].

It is not a Turing-complete language: you cannot write an executable program in it. It is meant to declare values for named parameters to steer job execution and adjust algorithm parameters (such as the electron lifetime in the simulation and reconstruction). Look at .fcl files in installed job directories, like `$DUNETPC_DIR/job` for examples. Fcl files are sought in the directory seach path FHICL_FILE_PATH when art starts up and when #include statements are processed. A fully-expanded fcl file with all the #include statements executed is referred to as a fhicl "document".

Parameters may be defined more than once. The last instance of a parameter definition wins out over previous ones. This makes for a common idiom in changing one or two parameters in a fhicl document. The generic pattern for making a short fcl file that modifies a parameter is:

~~~
#include "fcl_file_that_does_almost_what_I_want.fcl"
block.subblock.parameter: new_value
~~~
{:. source}

To see what block and subblock a parameter is in, use `fhcl-dump on` the parent fcl file and look for the curly brackets. You can also use

~~~
lar -c fclfile.fcl --debug-config tmp.txt --annotate
~~~
{:. source}

which is equivalent to `fhicl-dump` with the --annotate option and piping the output to tmp.txt.

Entire blocks of parameters can be substituted in using @local and @table idioms. See the examples and documentation for guidance on how to use these. Generally they are defined in the PROLOG sections of fcl files. PROLOGs must precede all non-PROLOG definitions and if their symbols are not subsequently used they do not get put in the final job configuration document (that gets stored with the data and thus may bloat it). This is useful if there are many alternate configurations for some module and only one is chosen at a time.

### Types of Plug-Ins

Plug-ins each have their own .so library which gets dynamically loaded by art when referenced by name in the fcl configuration.

Producer Modules A producer module is a software component that writes data products to the event memory. It is characterized by produces<> and consumes<> statements in the class constructor, and `art::Event::put()` calls in the `produces()` method. A producer must produce the data product collection it says it produces, even if it is empty, or art will throw an exception. `art::Event::put()` transfers ownership of memory (use std::move so as not to copy the data) from the module to the art event memory. Data in the art event memory will be written to the output file unless output commands in the fcl file tell art not to do that. Documentation on output commands can be found in the LArSoft wiki here: [https://cdcvs.fnal.gov/redmine/projects/larsoft/wiki/Rerun_part_of_all_a_job_on_an_output_file_of_that_job][larsoft-rerun-part-job] Producer modules have methods that are called on begin job, begin run, begin subrun, and on each event, as well as at the end of processing, so you can initialize counters or histograms, and finish up summaries at the end. Source code must be in files of the form: <modulename>_module.cc, where modulename does not have any underscores in it.

Analyzer Modules Analyzer modules read data product from the event memory and produce histograms or TTrees, or other output. They are typically scheduled after the producer modules have been run. Producer modules have methods that are called on begin job, begin run, begin subrun, and on each event, as well as at the end of processing, so you can initialize counters or histograms, and finish up summaries at the end. Source code must be in files of the form: <modulename>_module.cc, where modulename does not have any underscores in it.

Source Modules Source modules read data from input files and reformat it as need be, in order to put the data in art event data store. Most jobs use the art-provided RootInput source module which reads in art-formatted ROOT files. RootInput interacts well with the rest of the framework in that it provides lazy reading of TTree branches. Only when GetByLabel or GetValidHandle or other product get methods are called is the data actually fetched from the input file. This is useful for art jobs that only read a subset of the TBranches in an input file. Source code must be in files of the form: <modulename>_source.cc, where modulename does not have any underscores in it.

Services These are singleton classes that are globally visible within an art job. They can be FHiCL configured like modules, and they can schedule methods to be called on begin job, begin run, begin event, etc. They are meant to help supply configuration parameters like the drift velocity, or more complicated things like geometry functions, to modules that need them. Please do not use services as a back door for storing event data outside of the art event store. Source code must be in files of the form: `<modulename>_service.cc`, where servicename does not have any underscores in it.

Tools Tools are FHiCL-configurable software components that are not singletons, like services. They are meant to be swappable by FHiCL parameters which tell art which .so libraries to load up, configure, and call from user code. See the [Art Wiki Page][art-wiki-redmine] for more information on tools and other plug-ins.

You can use cetskelgen to make empty skeletons of art plug-ins. See the art wiki for documentation, or use

~~~
cetskelgen --help
~~~
{:. source}

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

## Introductory Documentation

Using LArSoft: [https://cdcvs.fnal.gov/redmine/projects/larsoft/wiki/Using_LArSoft][using-larsoft].

## Software structure

The LArSoft toolkit is a set of software components that simulate and reconstruct LArTPC data, and also it provides tools for accessing raw data from the experiments. LArSoft contains an interface to GEANT4 (art does not list GEANT4 as a dependency) and the GENIE generator. It contains geometry tools that are adapted for wire-based LArTPC detectors.

A recent graph of the UPS products in a full stack staring with protduneana is available here: [https://wiki.dunescience.org/w/img_auth.php/f/fd/Protoduneana_v09_12_00_e19-prof_graph.pdf][protoduneana] You can see the LArSoft pieces under dunetpc, as well as GEANT4, GENIE, ROOT, and a few others.

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
{:. source}

Some of these may require some configuration of input module labels so they can find the data products of interest.

### Examples and current workflows

The page with instructions on how to find and look at ProtoDUNE data has links to standard fcl configurations for simulating and reconstructing ProtoDUNE data: [https://wiki.dunescience.org/wiki/Look_at_ProtoDUNE_SP_data][look-at-protodune].

Try it yourself! The workflow for ProtoDUNE-SP MC is given in the Simulation Task Force web page. [https://wiki.dunescience.org/wiki/ProtoDUNE-SP_Simulation_Task_Force][protodune-sim-task-force] Here are instructions starting from a clean login on a dunegpvm machine:

~~~
 export USER=`whoami`
 mkdir -p /dune/data/users/$USER/tutorialtest
 cd /dune/data/users/$USER/tutorialtest
 source /cvmfs/dune.opensciencegrid.org/products/dune/setup_dune.sh
 setup dunetpc v09_13_00 -q e19:prof
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
{:. source}

The last command, which runs the event display, You can also browse the root files with a TBrowser or run other dumper fcl files on them. The dump example commands above redirect their outputs to text files which you can edit with a text editor or run grep on to look for things.

You can run the event display with

~~~
lar -c evd_protoDUNE.fcl reco_stage1.root
~~~
{:. source}

but it will run very slowly over a tunneled X connection. A VNC session will be much faster. Tips: select the "Reconstructed" radio button at the bottom and click on "Unzoom Interest" on the left to see the reconstructed objects in the three views.

To run the same exercise on a lxplus machine (though you still need to be in the Fermilab VO and be able to get a Fermilab Kerberos ticket), add these commands after setting up dunetpc and before running the job. Warning -- this involves running kinit on a lxplus machine which is not recommended, as your password goes over the network this way.

~~~
 kinit <user>@FNAL.GOV
 setup kx509
 setup_fnal_security
~~~
{:. source}

You will also have to use storage you can write to instead of `/dune/data`. This was tested with a personal afs work area, since eos required a CERN kerberos ticket to write to.

### DUNE software documentation and how-to's

This wiki page provides a lot of information on how to check out, build, and contribute to dune-specific larsoft plug-in code.

[https://cdcvs.fnal.gov/redmine/projects/dunetpc/wiki][dunetpc-wiki]

The third part of the tutorial gives hands-on exercises for doing these things.

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
{:. source}

## GArSoft

GArSoft is another art-based software package, designed to simulate the ND-GAr near detector. Many components were copied from LArSoft and modified for the pixel-based TPC with an ECAL. You can find installed versions in CVMFS with the following command:

~~~
ups list -aK+ garsoft
~~~
{:. source}

and you can check out the source and build it by following the instructions on the GArSoft wiki: [https://cdcvs.fnal.gov/redmine/projects/garsoft/wiki][garsoft-wiki]

{%include links.md%} 

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