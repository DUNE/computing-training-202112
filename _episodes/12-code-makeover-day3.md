---
title: Code-makeover - Submit with POMS
teaching: 60
exercises: 0
questions:
- How to submit grid jobs with POMS?
objectives:  
- Demonstrate use of POMS for job submission.
keypoints:
- Always, always, always prestage input datasets. No exceptions.
---

## Submit with POMS

This lesson extends from earlier work: [Grid Job Submission and Common Errors]({{ site.baseurl }}/07-grid-job-submission/index.html) 

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

When you start using POMS you must upload an x509 proxy to the sever before submitting (you can just scp your proxy file from a dunegpvm machine) and it must be named x509up_voms_dune_Analysis_yourusername when you upload it.
To upload, look for the User Data item in the left-hand menu on the POMS site, choose Uploaded Files, and follow the instructions.

Finally, here is an example of a campaign that does the same thing as the previous one, using our usual MC reco file from Prod2, but does it via making a SAM dataset using that as the input: [POMS campaign stage information](https://pomsgpvm01.fnal.gov/poms/campaign_stage_info/dune/analysis?campaign_stage_id=9753). 
Of course, before running **any** SAM project, we should prestage our input definition(s):

```bash
samweb prestage-dataset kherner-may2021tutorial-mc
```

replacing the above definition with your own definition as appropriate.

If you are used to using other programs for your work such as project.py, there is a helpful tool called [Project-py][project-py-guide] that you can use to convert existing xml into POMS configs, so you don't need to start from scratch! Then you can just switch to using POMS from that point forward. 

{%include links.md%} 

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


