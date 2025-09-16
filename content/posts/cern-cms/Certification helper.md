+++
title = "Certifying Data from Tracker Detector"
author = "Abhit"
date = 2022-01-22
tags = ["cern", "cms"]
categories = ["projects"]
+++

At CERN, the CMS experiment has a detector made up of many sub-detectors. One of the most important is the **tracker**, which works like a giant digital camera that records the paths of charged particles created in collisions. The tracker produces an enormous amount of data, and for physics analyses to be reliable, this data must be checked carefully for quality.  

This is where the **Data Quality Monitoring (DQM)** system comes in. The DQM software looks at the raw detector output and generates simplified summaries, known as *monitor elements*. These include:  
- **Histograms** of sensor signals.  
- **Basic statistics** about detector performance.  
- **Plots** that highlight unusual behavior.  

These summaries are examined through a web interface called the **DQMGUI**. Based on what they see, the monitoring team decides whether a dataset is good enough for physics analysis. Their decisions are stored in the **Run Registry**, the official database of certified data. In this context, **data certification** means reviewing each run of data and confirming its quality. It is essentially a **quality assurance stamp** before the data is released for physics use.  

### Who does the monitoring
Certification is a human-driven process carried out in shifts. **Operators (shifters)** are on duty during data taking: they monitor the tracker in real time, follow checklists, and confirm the detector status. **Supervisors (shift leaders)**, who are more experienced, review these certifications, validate the results, and prepare the summary reports.  

### What Certhelper does
The **Certhelper web application** was created to support this workflow. It guides operators through certification forms with many fields automatically filled from other CMS services. It enforces checklists so no critical step is missed, and stores certifications in a central database. Supervisors can review or edit the results and designate certain runs as *reference runs*, which are trusted and serve as baselines for future comparisons. The app also generates maps and plots to visualize detector health and produces daily and weekly reports automatically.  

### Impact
Before Certhelper, certification was a manual, repetitive, and error-prone task. Operators had to copy information between systems, and supervisors spent time on routine checks. With Certhelper, much of this process is automated. The tool saves time for both operators and supervisors, ensures consistency across hundreds of shifts, and guarantees that the physics results from CMS are based on high-quality, well-understood data.  

---

## Technical Overview

1. **Web Framework**
   - **Technology:** Django (Python web framework).  
   - **Features provided by Django:**  
     - **Models:** Represent entities like certified runs, datasets, and checklists.  
     - **Forms & Views:** Handle the certification process by allowing operators to fill in forms, ensuring the data is validated, and then storing it.  
     - **Admin Interface:** Enable administrators to configure checklists and roles.  
   - Why Django?  
     - Mature and well-supported.  
     - Batteries included (ORM, authentication, admin UI).  
     - Easy integration with APIs and CERN’s Single Sign-On user authentication.  
     - Clear modular structure: each feature lives in its own “app” (e.g., `certifier`, `checklists`, `shiftleader`).  

2. **Database**
   - **Technology:** Relational DB (PostgreSQL).  
   - **Purpose:** Stores certifications, user information, checklists, and reports.  
   - **Django ORM:** Maps Python classes (`TrackerCertification`, `Dataset`, `User`) to database tables.  
   - **PostgreSQL:** Chosen for scalability, durability, and smooth integration with CERN infrastructure.  
   - Why a database?  
     - Certification data must be reliable and queryable, including details on who certified it, when it was certified, and with what status.  
     - Relationships matter, i.e., each certification links to a run, dataset, user, and possibly a “reference run”.  
     - Supports consistency and auditing by ensuring that no data is lost, and supervisors can restore deleted entries.  


3. **Deployment Platform**
   - **Technology:** OpenShift (Kubernetes at CERN).  
   - **App Packaging:** Containerized as Docker images with code, dependencies, and runtime environment.  
   - **Static Files:** Served via WhiteNoise within the container.  
   - Why OpenShift/Kubernetes?
     - Provides scalability by running multiple pods to support concurrent users.
     - Manages secrets/configuration, e.g., DB credentials, API tokens, SSO secrets.  
     - Offers routing/ingress for secure HTTPS access.  
     - Ensures high availability by automatically restarting failed pods.  

4. **Integrations & External Systems**
   - **Authentication:** CERN Single Sign-On (SSO), integrated via OpenID Connect (OIDC) with Django AllAuth. Roles (operator/shift leader/admin) mapped from CERN groups.  
   - **External APIs:**  
     - **OMS (Online Monitoring System):** Supplies run metadata and auto-fills certification forms.  
     - **Run Registry:** Provides the official certification database for cross-checks.  
   - **Remote Execution:** Tracker map generation offloaded to CERN servers; triggered by Certhelper via SSH. Logs and updates are streamed back to users using Django Channels + Redis (WebSockets).  

---

## Summary 
- Django speeds up development and enforces a clean structure.  
- PostgreSQL guarantees data consistency and durability.  
- OpenShift/Kubernetes ensures scalability, security, and reliability.  
- Integrations tie Certhelper seamlessly into CERN’s ecosystem.

**Architecture**
- **Frontend:** Django templates + Bootstrap, with custom JavaScript for interactivity.  
- **Backend:** Django apps, each responsible for part of the workflow (certification form, checklists, shift-leader tools, reporting, etc.).  
- **Database:** PostgreSQL (persistent storage of certifications and related data).  
- **Deployment:** Containerized app running on OpenShift/Kubernetes with secrets, routing, and monitoring.  
- **Integrations:** CERN SSO, and remote tracker-maps generation.  


