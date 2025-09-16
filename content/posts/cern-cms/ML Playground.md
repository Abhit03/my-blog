+++
title = "Machine Learning Playground"
author = "Abhit"
date = 2022-03-04
tags = ["cern", "cms"]
categories = ["projects"]
+++

In the CMS experiment at CERN, huge amounts of collision data are collected every day. To make sure this data can be trusted for physics analyses, CMS relies on a process called **Data Quality Monitoring (DQM)**.  The DQM software looks at raw detector data and produces simple **summaries** called *monitor elements* which include:  
- Histograms of the sensor signals.  
- Basic statistics about detector performance.  
- Plots that help spot unusual behavior.  

Operators, known as **shifters**, and more experienced supervisors (shift leaders) examine these outputs using a web interface called the **DQMGUI**. Based on what they see, they make a final **certification decision** about whether a dataset is good enough for physics analysis. These results are stored in the **Run Registry**, the official database of certified data.

To make this process smoother, we developed the **Certification Helper (Certhelper)** web app. It guided shifters and shift leaders through checklists and forms to certify data at the **run level**, where a run might last from a few minutes up to a few hours.  

However, over time, the community wanted to certify data at a much finer granularity: the **lumisection level**. A lumisection is a 23-second block of data inside a run. Since a run can contain hundreds of lumisections, this dramatically increases the number of items to be checked. Manual certification at this scale quickly became impossible.

The proposed solution was to use **machine learning and statistical models** to support or automate parts of the process.  Examples include:  
- **Reference runs:** a “gold standard” dataset chosen by supervisors that represents how the detector should look when everything is working. New data can then be compared automatically against this reference.  
- **Automatic flagging:** models that suggest whether a lumisection is “GOOD” or “BAD,” reducing the manual effort needed from operators.  

### Who uses it?
- **Operators (shifters):** Instead of checking thousands of plots one by one, they can rely on model suggestions to focus only on suspicious data.  
- **Supervisors (shift leaders):** Use model outputs as decision support when selecting reference runs or validating final results.  
- **Researchers / ML developers:** Contribute by building and testing new models on past annotated data, benchmarking them to see which approaches work best.  

### What the ML Playground does
To tackle the challenge of certifying data at the much finer lumisection level, we created the **ML Playground (also called DQM Playground)**. It is designed as a **Kaggle-like platform**, but specialized for CMS data certification.  

The platform provides:  
- **Centralized datasets:** large collections of past runs and lumisections that have already been certified by humans serving as the “ground truth”.  
- **Standardized tasks:** predefined training and testing splits, so everyone evaluates their models on the same data.  
- **Model submissions:** researchers upload their predictions, which are automatically scored against the ground truth.  
- **Leaderboards:** results are ranked to allow fair, transparent comparison across methods.  
- **APIs & CLI tools:** Programmatic access ensures that experiments are reproducible and can be integrated into external workflows.  
- **Visualization:** histograms, statistics, and correlations can be explored through the web interface to better understand both data and model behavior.  

### Impact
Before the ML Playground, efforts in machine learning for DQM were scattered and inconsistent: 
- Researchers used different, sometimes arbitrary datasets.  
- There were no baseline models to serve as reference points.  
- Criteria for deciding whether a lumisection was GOOD or BAD were not standardized.  
- Models often ignored the fact that detector behavior evolves.  

The ML Playground directly addresses these issues by:  
- Standardizing data access and ensuring results are reproducible.  
- Offering baseline tasks and models so newcomers have a starting point.  
- Enabling the community to compare approaches side by side on equal footing.  
- Providing a pathway for successful models to be integrated into operator workflows, reducing manual workload and making lumisection-level certification feasible.  

---

### Technical Overview of ML Playground

1. **Web Framework**
   - **Technology:** Django (Python web framework).  
   - **Usage:** Leverages Django’s ORM and app structure to build the platform.  
   - **Models:** Represent core entities such as:  
     - Runs, lumisections, histograms (1D/2D).  
     - Tasks (training/testing datasets).  
     - Predictions (submitted model outputs).  
   - **Templates:** Power the web UI for:  
     - Run and lumisection summaries.  
     - Histogram and time-series visualizations.  
     - Task and prediction comparisons.  

2. **Database**
   - **Technology:** Relational DB (PostgreSQL).  
   - **Purpose:** Stores runs, lumisections, histograms, tasks, and prediction metadata.  
   - **Capabilities:**  
     - Provides a structured, queryable abstraction over raw DQM outputs (CSV/ROOT files).  
     - Ensures reproducibility — the same dataset split can always be regenerated for users.  

3. **APIs and CLI**
   - **REST API:** Django REST Framework exposes datasets, tasks, and predictions in JSON format.  
   - **Internal CLI:** Management commands for ETL (extract-transform-load), converting CSV/ROOT files into structured database entries.  
   - **External CLI:** Allows users to query tasks, download datasets, and submit predictions directly from scripts or notebooks.  

4. **Deployment Platform**
   - **Technology:** OpenShift (Kubernetes-based at CERN).  
   - **Packaging:** All components are containerized with Docker (web app, REST API, supporting services).  
   - **Why Kubernetes/OpenShift?**  
     - Scales as more users upload models or query data.  
     - Manages secrets and configuration securely.  
     - Provides high availability and automated restarts.  


---

### Summary
- Django provides rapid prototyping and clear structure.  
- PostgreSQL ensures consistency and scalability of stored datasets.  
- REST + CLI enable reproducibility and integration with external ML workflows.  
- OpenShift/Kubernetes ensures the platform can grow and remain reliable.  

**Architecture**
- **Frontend:** Django templates + Altair/JS for visualization.  
- **Backend:** Django apps representing runs, lumisections, tasks, predictions, histograms.  
- **Database:** PostgreSQL (persistent store for data, tasks, results).  
- **APIs:** Django REST endpoints for programmatic access.  
- **CLI:** For data ingestion and user interaction.  
- **Deployment:** Dockerized services on OpenShift/Kubernetes.  



