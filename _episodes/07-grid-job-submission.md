---
title: Grid Job Submission and Common Errors
teaching: 120
exercises: 0
questions:
- “What does it take to submit a job to the computing grid for processing?”
objectives:
- "objective 1"
- "objective 2"
keypoints:
- “keypoint 1"
- "keypoint 2"
---

## Submit a job

**Note that job submission requires FNAL account but can be done from a CERN machine, or any other with CVMFS access.**

First, log in to a `dunegpvm` machine (should work from `lxplus` too with a minor extra step of getting a Fermilab Kerberos ticket on `lxplus` via `kinit`). Then you will need to set up the job submission tools (`jobsub`). If you set up `dunetpc` it will be included, but if not, you need to do

~~~
    source /cvmfs/dune.opensciencegrid.org/products/dune/setup_dune.sh
    setup jobsub_client
~~~
{: .source}

Having done that, let us submit a prepared script:


~~~
jobsub_submit -G dune -M -N 1 --memory=1000MB --disk=1GB --cpu=1 --expected-lifetime=1h --resource-provides=usage_model=DEDICATED,OPPORTUNISTIC,OFFSITE -l '+SingularityImage=\"/cvmfs/singularity.opensciencegrid.org/fermilab/fnal-wn-sl7:latest\"' --append_condor_requirements='(TARGET.HAS_Singularity==true&&TARGET.HAS_CVMFS_dune_opensciencegrid_org==true&&TARGET.HAS_CVMFS_larsoft_opensciencegrid_org==true&&TARGET.CVMFS_dune_opensciencegrid_org_REVISION>=1105)' file:///dune/app/users/kherner/submission_test_singularity.sh
~~~
{: .source}


If all goes well you should see something like this:

~~~
/fife/local/scratch/uploads/dune/kherner/2021-01-20_120444.002077_4240
/fife/local/scratch/uploads/dune/kherner/2021-01-20_120444.002077_4240/submission_test_singularity.sh_20210120_120445_308543_0_1_.cmd
~~~
{: .source}

~~~
submitting....
Submitting job(s).
1 job(s) submitted to cluster 40351757.
JobsubJobId of first job: 40351757.0@jobsub01.fnal.gov
Use job id 40351757.0@jobsub01.fnal.gov to retrieve output
~~~
{: .output}

Now, let's look at some of these options in more detail.

* `-M` sends mail after the job completes whether it was successful for not. The default is email only on error. To disable all emails, use `--mail_never`.
* `-N` controls the number of identical jobs submitted with each cluster. Also called the process ID, the number ranges from 0 to N-1 and forms the part of the job ID number after the period, e.g. 12345678.N.
* `--memory, --disk, --cpu, --expected-lifetime` request this much memory, disk, number of cpus, and max run time.  Jobs that exceed the requested amounts will go into a held state. Defaults are 2000 MB, 10 GB, 1, and 8h, respectively.

Note that jobs are charged against the DUNE FermiGrid quota according to the greater of memory/2000 MB and number of CPUs, with fractional values possible. For example, a 3000MB request is charged 1.5 "slots", and 4000 MB would be charged 2. You are charged for the amount *requested*, not what is actually used, so you should not request any more than you actually need (your jobs will also take longer to start the more resources you request). Note also that jobs that run offsite do NOT count against the FermiGrid quota. '''In general, aim for memory and run time requests that will cover 90-95% of your jobs and use the [autorelease feature][job-autorelease] to deal with the remainder.
* `--resource-provides=usage_model` This controls where jobs are allowed to run. DEDICATED means use the DUNE FermiGrid quota, OPPORTUNISTIC means use idle FermiGrid resources beyond the DUNE quota if they are available, and OFFSITE means use non-Fermilab resources. You can combine them in a comma-separated list. <span style="color:red"> In nearly all cases you should be setting this to DEDICATED,OPPORTUNISTIC,OFFSITE.</span> This ensures maximum resource availability and will get your jobs started the fastest. Note that because of Singularity, **there is absolutely no difference** between the environment on Fermilab worker nodes and any other place. Depending on where your input data are (if any), you might see slight differences in network latency, but that's it.
* `-l` (or `--lines=`) allows you to pass additional arbitrary HTCondor-style `classad` variables into the job. In this case, we're specifying exactly what `Singularity` image we want to use in the job. It will be automatically set up for us when the job starts. Any other valid HTCondor classad is possible. In practice you don't have to do much beyond the `Singularity` image. Here, pay particular attention to the quotes and backslashes.
* `--append_condor_requirements` allows you to pass additional `HTCondor-style` requirements to your job. This helps ensure that your jobs don't start on a worker node that might be missing something you need (a corrupt or out of date `CVMFS` repository, for example). Some checks run at startup for a variety of `CVMFS` repositories. Here, we check that Singularity invocation is working and that the `CVMFS` repos we need ( [dune.opensciencegrid.org][dune-openscience-grid-org] and [larsoft.opensciencegrid.org][larsoft-openscience-grid-org] ) are in working order. Optionally you can also place version requirements on CVMFS repos (as we did here as an example), useful in case you want to use software that was published very recently and may not have rolled out everywhere yet.

## Job Output

This particular test writes a file to `/pnfs/dune/scratch/users/<username>/job_output_<id number>.log`. Verify that the file exists and is non-zero size after the job completes. You can delete it after that; it just prints out some information about the environment.

More information about `jobsub` is available [here][redmine-wiki-jobsub] and [here][redmine-wiki-using-the-client].


[job-autorelease]: https://cdcvs.fnal.gov/redmine/projects/fife/wiki/Job_autorelease
[dune-openscience-grid-org]: dune.opensciencegrid.org
[larsoft-openscience-grid-org]: larsoft.opensciencegrid.org
[redmine-wiki-jobsub]: https://cdcvs.fnal.gov/redmine/projects/jobsub/wiki
[redmine-wiki-using-the-client]: https://cdcvs.fnal.gov/redmine/projects/jobsub/wiki/Using_the_Client


{%include links.md%}
