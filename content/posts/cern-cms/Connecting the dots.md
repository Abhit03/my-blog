+++
title = "Data Certification: Human Workflows to Automation"
author = "Abhit"
date = 2022-03-04
tags = ["cern", "cms"]
categories = ["projects"]
+++

At CERN’s CMS experiment, one of the largest particle physics experiments in the world, the **tracker detector** records the paths of particles created during proton collisions. To make sure the data is reliable for physics studies, CMS uses a process called **Data Quality Monitoring (DQM)** followed by **certification**.  

<!--more--> 
### The challenge of data certification in CMS
Certification has traditionally been done by:  
- **Operators (shifters):** people on shift who look at plots and fill in forms to mark data as good or bad.  
- **Supervisors (shift leaders):** more experienced team members who double-check the work and assign **reference runs** which are examples of data taken when the detector was working perfectly. These reference runs serve as a baseline to compare new data against.  

For many years, this process was manageable because certification was done at the **run level **, where one run might last anywhere from a few minutes to a couple of hours.  
But the community wanted finer granularity: the **lumisection level** (about 23 seconds of data). Since each run contains hundreds of lumisections, the amount of data to check exploded, and it became clear that manual workflows alone would not scale.

---

### Project 1: Certhelper – streamlining human workflows
The first step was to improve the process for humans.  We developed **Certification Helper (Certhelper)**, a web application built with Django that provided a structured, user-friendly environment for certification.  

With Certhelper, operators could:  
- Fill in certification forms that were auto-completed with metadata from CERN databases.  
- Follow checklists to make sure no steps are forgotten.  
- Save everything into a central PostgreSQL database, ensuring traceability.  

Shift leaders could:  
- Review the certifications.
- Assign reference runs.
- Generate automated daily and weekly reports.

Certhelper also integrated with CERN’s **Online Monitoring System (OMS)** and the **Run Registry**, reducing duplication of effort. The application was deployed on **OpenShift (Kubernetes)**, which made it scalable and reliable.  Overall, Certhelper turned a manual, error-prone workflow into one that was **consistent, auditable, and much less time-consuming**.

---

### Project 2: ML Playground – preparing for automation
Certhelper solved the immediate need, but it couldn’t solve the looming challenge: At the lumisection level, humans would need to check **thousands of data blocks per run**, which is simply too much.  

To address this, I initiated the **Machine Learning Playground (ML Playground)**, a platform inspired by Kaggle competitions. The idea was to let researchers develop and test **machine learning and statistical models** that could help automate certification.  

The ML Playground provided:  
- **Datasets of past runs and lumisections**, already labeled by humans, to serve as training and testing material.  
- **Standardized tasks and benchmarks**, so all models were compared on the same ground.  
- **Submissions and leaderboards**, where researchers could upload predictions and see how they ranked against others.  
- **Visualization tools**, to better understand both the data and the model performance.  
- **APIs and CLI tools**, so experiments were reproducible and easy to integrate into research workflows.  

It was deployed in Docker containers on OpenShift, ensuring the platform could scale as more users participated. Where Certhelper supported human operators, the ML Playground was designed to augment them with algorithms, providing a pathway toward automation.

---

### Summary
- **Certhelper**: addressed the present problem by making certification at the run level efficient, reliable, and structured for humans.  
- **ML Playground**: anticipated the future problem by enabling the community to build and compare machine learning models for lumisection-level certification.  

Both projects build on top of the **CMS Data Quality Monitoring system**, which produces the monitoring plots and statistics. Together, they represent a natural progression: from *human-driven certification at a coarse level* to *machine-assisted certification at a fine level*.  

