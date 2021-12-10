---
title: Grid Job Submission and Common Errors
teaching: 45
exercises: 0
questions:
- How to submit grid jobs?
objectives:
- Submit a job and understand what's happening behind the scenes
- Monitor the job and look at its outputs
- Review best practices for submitting jobs (including what NOT to do)
- Extension; submit a small job with POMS
keypoints:
- When in doubt, ask! Understand that policies and procedures that seem annoying, overly complicated, or unnecessary (especially when compared to running an interactive test) are there to ensure efficient operation and scalability. They are also often the result of someone breaking something in the past, or of simpler approaches not scaling well.
- Send test jobs after creating new workflows or making changes to existing ones. If things don't work, don't blindly resubmit and expect things to magically work the next time.
- Only copy what you need in input tar files. In particular, avoid copying log files, .git directories, temporary files, etc. from interactive areas.
- Take care to follow best practices when setting up input and output file locations.
- Always, always, always prestage input datasets. No exceptions.
---

<!--
## Video Session Part 1 of 2

<center>
<iframe width="560" height="315" src="https://www.youtube.com/embed/1tpUd3c3Irc" title="DUNE Computing Tutorial May 2021 Day 2 Grid Job Submission and Common Errors" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</center>-->

## Submit a job

**Note that job submission requires FNAL account but can be done from a CERN machine, or any other with CVMFS access.**

First, log in to a `dunegpvm` machine (should work from `lxplus` too with a minor extra step of getting a Fermilab Kerberos ticket on `lxplus` via `kinit`). Then you will need to set up the job submission tools (`jobsub`). If you set up `dunetpc` it will be included, but if not, you need to do

```bash
source /cvmfs/dune.opensciencegrid.org/products/dune/setup_dune.sh
setup jobsub_client
```
Having done that, let us submit a prepared script:

~~~
jobsub_submit -G dune -M -N 1 --memory=1000MB --disk=1GB --cpu=1 --expected-lifetime=1h --resource-provides=usage_model=DEDICATED,OPPORTUNISTIC,OFFSITE -l '+SingularityImage=\"/cvmfs/singularity.opensciencegrid.org/fermilab/fnal-wn-sl7:latest\"' --append_condor_requirements='(TARGET.HAS_CVMFS_dune_opensciencegrid_org==true&&TARGET.HAS_CVMFS_larsoft_opensciencegrid_org==true&&TARGET.CVMFS_dune_opensciencegrid_org_REVISION>=1105)' file:///dune/app/users/kherner/submission_test_singularity.sh
~~~
{: .source}

If all goes well you should see something like this:

~~~
/fife/local/scratch/uploads/dune/kherner/2021-01-20_120444.002077_4240
/fife/local/scratch/uploads/dune/kherner/2021-01-20_120444.002077_4240/submission_test_singularity.sh_20210120_120445_308543_0_1_.cmd
submitting....
Submitting job(s).
1 job(s) submitted to cluster 40351757.
JobsubJobId of first job: 40351757.0@jobsub01.fnal.gov
Use job id 40351757.0@jobsub01.fnal.gov to retrieve output
~~~
{: .output}

> ## Quiz 
>
> 1. What is your job ID?
>
{: .solution}

Now, let's look at some of these options in more detail.

* `-M` sends mail after the job completes whether it was successful for not. The default is email only on error. To disable all emails, use `--mail_never`.  
* `-N` controls the number of identical jobs submitted with each cluster. Also called the process ID, the number ranges from 0 to N-1 and forms the part of the job ID number after the period, e.g. 12345678.N.  
* `--memory, --disk, --cpu, --expected-lifetime` request this much memory, disk, number of cpus, and max run time.  Jobs that exceed the requested amounts will go into a held state. Defaults are 2000 MB, 10 GB, 1, and 8h, respectively. Note that jobs are charged against the DUNE FermiGrid quota according to the greater of memory/2000 MB and number of CPUs, with fractional values possible. For example, a 3000 MB request is charged 1.5 "slots", and 4000 MB would be charged 2. You are charged for the amount **requested**, not what is actually used, so you should not request any more than you actually need (your jobs will also take longer to start the more resources you request). Note also that jobs that run offsite do NOT count against the FermiGrid quota. **In general, aim for memory and run time requests that will cover 90-95% of your jobs and use the [autorelease feature][job-autorelease] to deal with the remainder**.  
* `--resource-provides=usage_model` This controls where jobs are allowed to run. DEDICATED means use the DUNE FermiGrid quota, OPPORTUNISTIC means use idle FermiGrid resources beyond the DUNE quota if they are available, and OFFSITE means use non-Fermilab resources. You can combine them in a comma-separated list. <span style="color:red"> In nearly all cases you should be setting this to DEDICATED,OPPORTUNISTIC,OFFSITE.</span> This ensures maximum resource availability and will get your jobs started the fastest. Note that because of Singularity, **there is absolutely no difference** between the environment on Fermilab worker nodes and any other place. Depending on where your input data are (if any), you might see slight differences in network latency, but that's it.  
* `-l` (or `--lines=`) allows you to pass additional arbitrary HTCondor-style `classad` variables into the job. In this case, we're specifying exactly what `Singularity` image we want to use in the job. It will be automatically set up for us when the job starts. Any other valid HTCondor `classad` is possible. In practice you don't have to do much beyond the `Singularity` image. Here, pay particular attention to the quotes and backslashes.  
* `--append_condor_requirements` allows you to pass additional `HTCondor-style` requirements to your job. This helps ensure that your jobs don't start on a worker node that might be missing something you need (a corrupt or out of date `CVMFS` repository, for example). Some checks run at startup for a variety of `CVMFS` repositories. Here, we check that Singularity invocation is working and that the `CVMFS` repos we need ( [dune.opensciencegrid.org][dune-openscience-grid-org] and [larsoft.opensciencegrid.org][larsoft-openscience-grid-org] ) are in working order. Optionally you can also place version requirements on CVMFS repos (as we did here as an example), useful in case you want to use software that was published very recently and may not have rolled out everywhere yet.  

## Job Output

This particular test writes a file to `/pnfs/dune/scratch/users/<username>/job_output_<id number>.log`.
Verify that the file exists and is non-zero size after the job completes.
You can delete it after that; it just prints out some information about the environment.

More information about `jobsub` is available [here][redmine-wiki-jobsub] and [here][redmine-wiki-using-the-client].

## Submit a job using the tarball containing custom code (left as an exercise)

First off, a very important point: for running analysis jobs, **you may not actually need to pass an input tarball**, especially if you are just using code from the base release and you don't actually modify any of it.
All you need to do is set up any required software from CVMFS (e.g. dunetpc and/or protoduneana), and you are ready to go.
If you're just modifying a fcl file, for example, but no code, it's actually more efficient to copy just the fcl(s) your changing to the scratch directory within the job, and edit them as part of your job script (copies of a fcl file in the current working directory have priority over others by default).

Sometimes, though, we need to run some custom code that isn't in a release. 
We need a way to efficiently get code into jobs without overwhelming our data transfer systems.
We have to make a few minor changes to the scripts you made in the previous tutorial section, generate a tarball, and invoke the proper jobsub options to get that into your job.
There are many ways of doing this but by far the best is to use the Rapid Code Distribution Service (RCDS), as shown in our example.  

If you have finished up the LArSoft follow-up and want to use your own code for this next attempt, feel free to tar it up (you don't need anything besides the localProducts* and work directories) and use your own tar ball in lieu of the one in this example.
You will have to change the last line with your own submit file instead of the pre-made one.

First, we should make a tarball. Here is what we can do (assuming you are starting from /dune/app/users/username/):

```bash
cp /dune/app/users/kherner/setupMay2021Tutorial-grid.sh /dune/app/users/username/
cp /dune/app/users/kherner/dec2021tutorial/localProducts_larsoft__e19_prof/setup-grid /dune/app/users/username/dec2021tutorial/localProducts_larsoft__e19_prof/setup-grid
```

Before we continue, let's examine these files a bit. We will source the first one in our job script, and it will set up the environment for us.

~~~
#!/bin/bash                                                                                                                                                                                                      

DIRECTORY=may2021tutorial
# we cannot rely on "whoami" in a grid job. We have no idea what the local username will be.
# Use the GRID_USER environment variable instead (set automatically by jobsub). 
USERNAME=${GRID_USER}

source /cvmfs/dune.opensciencegrid.org/products/dune/setup_dune.sh
export WORKDIR=${_CONDOR_JOB_IWD} # if we use the RCDS the our tarball will be placed in $INPUT_TAR_DIR_LOCAL.
if [ ! -d "$WORKDIR" ]; then
  export WORKDIR=`echo .`
fi

source ${INPUT_TAR_DIR_LOCAL}/${DIRECTORY}/localProducts*/setup-grid 
mrbslp
~~~
{: .source}


Now let's look at the difference between the setup-grid script and the plain setup script.
Assuming you are currently in the /dune/app/users/username directory:

```bash
diff may2021tutorial/localProducts_larsoft__e19_prof/setup may2021tutorial/localProducts_larsoft__e19_prof/setup-grid
```

~~~
< setenv MRB_TOP "/dune/app/users/<username>/may2021tutorial"
< setenv MRB_TOP_BUILD "/dune/app/users/<username>/may2021tutorial"
< setenv MRB_SOURCE "/dune/app/users/<username>/may2021tutorial/srcs"
< setenv MRB_INSTALL "/dune/app/users/<username>/may2021tutorial/localProducts_larsoft__e19_prof"
---
> setenv MRB_TOP "${INPUT_TAR_DIR_LOCAL}/may2021tutorial"
> setenv MRB_TOP_BUILD "${INPUT_TAR_DIR_LOCAL}/may2021tutorial"
> setenv MRB_SOURCE "${INPUT_TAR_DIR_LOCAL}/may2021tutorial/srcs"
> setenv MRB_INSTALL "${INPUT_TAR_DIR_LOCAL}/may2021tutorial/localProducts_larsoft__e19_prof"
~~~
{: . output}

As you can see, we have switched from the hard-coded directories to directories defined by environment variables; the `INPUT_TAR_DIR_LOCAL` variable will be set for us (see below).
Now, let's actually create our tar file. Again assuming you are in `/dune/app/users/kherner/may2021tutorial/`:
```bash
tar --exclude '.git' -czf may2021tutorial.tar.gz may2021tutorial/localProducts_larsoft__e19_prof may2021tutorial/work setupMay2021Tutorial-grid.sh
```
Then submit another job (in the following we keep the same submit file as above):

```bash
jobsub_submit -G dune -M -N 1 --memory=1800MB --disk=2GB --expected-lifetime=3h --cpu=1 --resource-provides=usage_model=DEDICATED,OPPORTUNISTIC,OFFSITE --tar_file_name=dropbox:///dune/app/users/<username>/dec2021tutorial.tar.gz --use-cvmfs-dropbox -l '+SingularityImage=\"/cvmfs/singularity.opensciencegrid.org/fermilab/fnal-wn-sl7:latest\"' --append_condor_requirements='(TARGET.HAS_Singularity==true&&
TARGET.HAS_CVMFS_dune_opensciencegrid_org==true&&
TARGET.HAS_CVMFS_larsoft_opensciencegrid_org==true&&
TARGET.CVMFS_dune_opensciencegrid_org_REVISION>=1105&&
TARGET.HAS_CVMFS_fifeuser1_opensciencegrid_org==true&&
TARGET.HAS_CVMFS_fifeuser2_opensciencegrid_org==true&&
TARGET.HAS_CVMFS_fifeuser3_opensciencegrid_org==true&&
TARGET.HAS_CVMFS_fifeuser4_opensciencegrid_org==true)' file:///dune/app/users/kherner/run_dec2021tutorial.sh
```

You'll see this is very similar to the previous case, but there are some new options: 

* `--tar_file_name=dropbox://` automatically copies and untars the given tarball into a directory on the worker node, accessed via the INPUT_TAR_DIR_LOCAL environment variable in the job. As of now, only one such tarball can be specified. If you need to copy additional files into your job that are not in the main tarball you can use the -f option (see the jobsub manual for details). The value of INPUT_TAR_DIR_LOCAL is by default $CONDOR_DIR_INPUT/name_of_tar_file, so if you have a tar file named e.g. may2021tutorial.tar.gz, it would be $CONDOR_DIR_INPUT/may2021tutorial.
* `--use-cvmfs-dropbox` stages that tarball through the RCDS (really a collection of special CVMFS repositories). As of April 2021, it is now the default method for tarball transfer and the --use-cvmfs-dropbox option is not needed (though it will not hurt to keep if it your submission for now).  
* Notice that the `--append_condor_requirements` line is longer now, because we also check for the fifeuser[1-4]. opensciencegrid.org CVMFS repositories.  

Now, there's a very small gotcha when using the RCDS, and that is when your job runs, the files in the unzipped tarball are actually placed in your work area as symlinks from the CVMFS version of the file (which is what you want since the whole point is not to have N different copies of everything).
The catch is that if your job script expected to be able to edit one or more of those files within the job, it won't work because the link is to a read-only area.
Fortunately there's a very simple trick you can do in your script before trying to edit any such files: 

~~~
cp ${INPUT_TAR_DIR_LOCAL}/file_I_want_to_edit mytmpfile  # do a cp, not mv
rm ${INPUT_TAR_DIR_LOCAL}file_I_want_to_edit # This really just removes the link
mv mytmpfile file_I_want_to_edit # now it's available as an editable regular file.
~~~
{: .source}

You certainly don't want to do this for every file, but for a handful of small text files this is perfectly acceptable and the overall benefits of copying in code via the RCDS far outweigh this small cost. 
This can get a little complicated when trying to do it for things several directories down, so it's easiest to have such files in the top level of your tar file. 

<!--
## Video Session Part 2 of 2

<center>
<iframe width="560" height="315" src="https://www.youtube.com/embed/L3dx-NQJvp8" title="DUNE Computing Tutorial May 2021 Day 2 Grid Job Submission and Common Errors Part 2" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</center>-->

## Monitor your jobs
For all links below, log in with your FNAL Services credentials (FNAL email, not Kerberos password).

* What DUNE is doing overall:  
[https://fifemon.fnal.gov/monitor/d/000000053/experiment-batch-details?orgId=1&var-experiment=dune](https://fifemon.fnal.gov/monitor/d/000000053/experiment-batch-details?orgId=1&var-experiment=dune)


* What's going on with only your jobs:   
Remember to change the url with your own username and adjust the time range to cover the region of interest.
[https://fifemon.fnal.gov/monitor/d/000000116/user-batch-details?orgId=1&var-cluster=fifebatch&var-user=kherner](https://fifemon.fnal.gov/monitor/d/000000116/user-batch-details?orgId=1&var-cluster=fifebatch&var-user=kherner)

* Why your jobs are held:  
Remember to choose your username in the upper left.  
[https://fifemon.fnal.gov/monitor/d/000000146/why-are-my-jobs-held?orgId=1](https://fifemon.fnal.gov/monitor/d/000000146/why-are-my-jobs-held?orgId=1)

## View the stdout/stderr of our jobs
Here's the link for the history page of the example job: [link](https://fifemon.fnal.gov/monitor/d/000000115/job-cluster-summary?orgId=1&var-cluster=40351757&var-schedd=jobsub01.fnal.gov&from=1611098894726&to=1611271694726). 

Feel free to sub in the link for your own jobs.

Once there, click "View Sandbox files (job logs)".
In general you want the .out and .err files for stdout and stderr.
The .cmd file can sometimes be useful to see exactly what got passed in to your job.

[Kibana][kibana] can also provide a lot of information.

You can also download the job logs from the command line with jobsub_fetchlog: 

```bash
jobsub_fetchlog --jobid=12345678.0@jobsub0N.fnal.gov --unzipdir=some_appropriately_named_directory
```

That will download them as a tarball and unzip it into the directory specified by the --unzipdir option.
Of course replace 12345678.0@jobsub0N.fnal.gov with your own job ID. 

> ## Quiz 
>
> Download the log of your last submission via jobsub_fetchlog or look it up on the monitoring pages. Then answer the following questions (all should be available in the .out or .err files):
> 1. On what site did your job run?
> 2. How much memory did it use?
> 3. Did it exit abnormally? If so, what was the exit code?
>
{: .solution}

## Brief review of best practices in grid jobs (and a bit on the interactive machines)

* When creating a new workflow or making changes to an existing one, <span style="color:red">**ALWAYS test with a single job first**</span>. Then go up to 10, etc. Don't submit thousands of jobs immediately and expect things to work.  
* **ALWAYS** be sure to prestage your input datasets before launching large sets of jobs.  
* **Use RCDS**; do not copy tarballs from places like scratch dCache. There's a finite amount of transfer bandwidth available from each dCache pool. If you absolutely cannot use RCDS for a given file, it's better to put it in resilient (but be sure to remove it when you're done!). The same goes for copying files from within your own job script: if you have a large number of jobs looking for a same file, get it from resilient. Remove the copy when no longer needed. Files in resilient dCache that go unaccessed for 45 days are automatically removed.  
* Be careful about placing your output files. **NEVER** place more than a few thousand files into any one directory inside dCache. That goes for all type of dCache (scratch, persistent, resilient, etc).  
* **Avoid** commands like `ifdh ls /path/with/wildcards/*/` inside grid jobs. That is a VERY expensive operation and can cause a lot of pain for many users.  
* Use xrootd when opening files interactively; this is much more stable than simply doing `root /pnfs/dune/...`
* **NEVER** copy job outputs to a directory in resilient dCache. Remember that they are replicated by a factor of 20! **Any such files are subject to deletion without warning**.  
* **NEVER** do hadd on files in `/pnfs` areas unless you're using `xrootd`. I.e. do NOT do hadd out.root `/pnfs/dune/file1 /pnfs/dune/file2 ...` This can cause severe performance degradations.  

## (Time permitting) submit with POMS

POMS is the recommended way of submitting large workflows. It offers several advantages over other systems, such as 

* Fully configurable. Any executables can be run, not necessarily only lar or art
* Automatic monitoring and campaign management options
* Multi-stage workflow dependencies, automatic dataset creation between stages
* Automated recovery options

At its core, in POMS one makes a "campaign", which has one or more "stages". In our example there is only a single stage.  

For analysis use: [main POMS page][poms-page-ana]  
An [example campaign](https://pomsgpvm01.fnal.gov/poms/campaign_stage_info/dune/analysis?campaign_stage_id=9743).

Typical POMS use centers around a configuration file (often more like a template which can be reused for many campaigns) and various campaign-specific settings for overriding the defaults in the config file.
An example config file designed to do more or less what we did in the previous submission is here: `/dune/app/users/kherner/may2021tutorial/work/pomsdemo.cfg`

You can find more about POMS here: [POMS User Documentation][poms-user-doc]  
Helpful ideas for structuring your config files are here: [Fife launch Reference][fife-launch-ref]  

When you start using POMS you must upload an x509 proxy to the sever before submitting. The best way to do that is to set up the poms_client UPS product and then use the upload_file command after you have generated your proxy:

```bash
kx509
voms-proxy-init -rfc -noregen -voms dune:/dune/Role=Analysis -valid 120:00
upload_file --experiment dune --proxy
```
Finally, here is an example of a campaign that does the same thing as the previous one, using our usual MC reco file from Prod2, but does it via making a SAM dataset using that as the input: [POMS campaign stage information](https://pomsgpvm01.fnal.gov/poms/campaign_stage_info/dune/analysis?campaign_stage_id=12559). 
Of course, before running **any** SAM project, we should prestage our input definition(s):

```bash
samweb prestage-dataset kherner-may2021tutorial-mc
```
{: .source}

replacing the above definition with your own definition as appropriate.

If you are used to using other programs for your work such as project.py, there is a helpful tool called [Project-py][project-py-guide] that you can use to convert existing xml into POMS configs, so you don't need to start from scratch! Then you can just switch to using POMS from that point forward. 

## Further Reading
Some more background material on these topics (including some examples of why certain things are bad) are on this PDF:  
[DUNE Computing Tutorial:Advanced topics and best practices][DUNE_computing_tutorial_advanced_topics_20210129]  

[The Glidein-based Workflow Management System]( https://glideinwms.fnal.gov/doc.prd/index.html )

[Introduction to Docker](https://hsf-training.github.io/hsf-training-docker/index.html)

[job-autorelease]: https://cdcvs.fnal.gov/redmine/projects/fife/wiki/Job_autorelease
[dune-openscience-grid-org]: dune.opensciencegrid.org
[larsoft-openscience-grid-org]: larsoft.opensciencegrid.org
[redmine-wiki-jobsub]: https://cdcvs.fnal.gov/redmine/projects/jobsub/wiki
[redmine-wiki-using-the-client]: https://cdcvs.fnal.gov/redmine/projects/jobsub/wiki/Using_the_Client
[fifemon-dune]: https://fifemon.fnal.gov/monitor/d/000000053/experiment-batch-details?orgId=1&var-experiment=dune
[fifemon-myjobs]: https://fifemon.fnal.gov/monitor/d/000000116/user-batch-details?orgId=1&var-cluster=fifebatch&var-user=kherner
[fifemon-whyheld]: https://fifemon.fnal.gov/monitor/d/000000146/why-are-my-jobs-held?orgId=1
[kibana]: https://fifemon.fnal.gov/kibana/goto/8f432d2e4a40cbf81d3072d9c9d688a6
[poms-page-ana]: https://pomsgpvm01.fnal.gov/poms/index/dune/analysis/
[poms-user-doc]: https://cdcvs.fnal.gov/redmine/projects/prod_mgmt_db/wiki/POMS_User_Documentation
[fife-launch-ref]: https://cdcvs.fnal.gov/redmine/projects/fife_utils/wiki/Fife_launch_Reference 
[poms-campaign-stage-info]: https://pomsgpvm01.fnal.gov/poms/campaign_stage_info/dune/analysis?campaign_stage_id=9023
[project-py-guide]: https://cdcvs.fnal.gov/redmine/projects/project-py/wiki/Project-py_guide
[DUNE_computing_tutorial_advanced_topics_20210129]: https://indico.fnal.gov/event/20144/contributions/55932/attachments/34945/42690/DUNE_computing_tutorial_advanced_topics_and_best_practices_20200129.pdf  


{%include links.md%}
