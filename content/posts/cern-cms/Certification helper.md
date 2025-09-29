+++
title = "Ensuring Quality in CMS Tracker Data"
author = "Abhit"
date = 2022-01-22
tags = ["cern", "monitoring", "web-app"]
categories = ["projects", "CERN"]
+++

Every second, the CMS tracker detector records millions of particle trajectories. But without careful quality checks, this flood of data risks being unusable for physics discoveries. To guard against this, CMS relies on a dedicated Data Quality Monitoring (DQM) system. The tracker, functioning like a giant digital camera, captures the paths of charged particles produced during collisions. Because it generates such an enormous volume of data, ensuring quality is essential for 
reliable physics analyses.

<!--more--> 
## Certifying the Data

The DQM software processes raw detector output and produces simplified summaries, known as monitor elements. These include histograms of sensor signals, basic statistics on detector performance, and plots that highlight irregular or unexpected behavior. Physicists examine these summaries through a web interface called the DQMGUI, where the monitoring team decides whether a dataset is suitable for physics analysis. Their decisions are then recorded in the Run Registry, the official database of certified data. In this process, data certification acts as a quality assurance stamp, ensuring that only trustworthy datasets are released for physics studies.

While the DQM system generates automated summaries, the final step of certification remains a human-driven process organized in shifts. Operators, or shifters, are on duty during data taking. They monitor the tracker in real time, follow detailed checklists, and confirm the detector's status. Their work is then reviewed by experienced supervisors (shift leaders), who validate certifications, ensure consistency, and prepare official summary reports.

## Streamlining Certification

To make this process more efficient, I helped develop the CertHelper web application. The tool streamlines certification by guiding operators through structured forms, with many fields automatically filled from other CMS services. This reduces manual entry and minimizes the risk of errors.

CertHelper also enforces checklists to ensure that no critical step is overlooked and stores all certifications in a centralized database. Supervisors can review or edit entries and designate certain runs as reference runs, which then serve as trusted baselines for future comparisons.

Beyond data entry and validation, the application provides visualization and reporting features. It generates maps and plots that illustrate detector health and automatically produces daily and weekly reports, giving the monitoring team a clear and consistent view of performance over time.


## Impact

Before CertHelper, certification was manual and error-prone, with operators copying information between systems and supervisors tied up in routine checks. With CertHelper, much of this work is automated. The tool saves time, enforces consistency across shifts, and builds confidence in the results, ensuring that the discoveries made at CMS are based on data that is both reliable and of high quality. 


---

## Appendix: Architecture

```mermaid
flowchart LR
  U[Users] -->|HTTPS| R[OpenShift Ingress]

  subgraph OS[Kubernetes / OpenShift]
    R --> A[Django App]
    A -->|ORM| DB[(PostgreSQL)]
  end

  SSO["CERN SSO (OIDC)"] -- Auth --> A
  OMS[OMS API<br>Metadata] --> A
  RR[Run Registry API] --> A
```

1. **Web Framework**
   - **Technology:** Django (Python web framework).  
   - **Features provided by Django:**  
     - **Models:** Represent entities like certified runs, users and checklists.  
     - **Forms & Views:** Handle the certification process by allowing operators to fill in forms, ensuring the data is validated, and then storing it.  
     - **Admin Interface:** Enable administrators to configure checklists and roles.  
   - Why Django?  
     - Mature and well-supported.  
     - Batteries included (ORM, authentication, admin UI).  
     - Easy integration with APIs and CERN’s Single Sign-On user authentication.  

2. **Database**
   - **Technology:** Relational DB (PostgreSQL).  
   - **Purpose:** Stores certifications, user information, checklists, and reports.  
   - **Django ORM:** Maps Python classes (`TrackerCertification`, `User`) to database tables.  
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
