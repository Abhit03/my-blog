+++
title = "Deploying Edge Computing Platform on K8s"
author = "Abhit"
date = 2024-06-14
tags = ["cern", "ics"]
categories = ["projects"]
+++

Imagine a factory or laboratory full of machines, each with its own sensors and controllers. Traditionally, all data from these devices is sent back to central servers for processing, but this can create delays and bottlenecks. **Industrial Edge computing** changes this by running applications directly close to the machines.  

<!--more--> 
At CERN, this means that small industrial PCs called **Edge Devices** can host apps that collect diagnostics, run analytics, or even control equipment locally. For example:  
- A ventilation system can adjust airflow in real time based on local sensor data.  
- An energy-optimization app can decide how to run motors more efficiently without waiting for instructions from central IT.  

But managing hundreds of such Edge Devices across CERN requires a central “control tower.” This is where **Industrial Edge Management (IEM)** comes in:  
- IEM is a management platform (a web-based dashboard and backend) that lets administrators onboard new devices, deploy apps to them, update software, manage certificates, and monitor their health.  
- In short, if Edge Devices are like the “workers” doing local computing, then IEM is the “manager” that coordinates them all.  

Deploying IEM inside CERN’s private cloud required combining two major infrastructure layers:  
- **OpenStack**: used to create the virtual machines where IEM runs.  
- **Kubernetes**: used to orchestrate the containers that make up the IEM platform and keep them running reliably.  

### Problem Statement
The challenge was to set up Siemens Industrial Edge Management (IEM) on CERN’s OpenStack cloud from scratch.  
This included:  
- Choosing the right deployment tools and workflow.  
- Guiding team members through key Kubernetes concepts for those unfamiliar with the platform.
- Creating a step-by-step, reproducible deployment method.  
- Integrating with CERN’s existing services (networking, DNS names, security certificates).  

---

### Implementation Steps
1. **Provisioning an Operator VM on OpenStack**  
   - Created a Linux virtual machine using the OpenStack command-line tools.  
   - Attached extra storage volumes and mounted them on the VM.  
   - Installed the required utilities such as the OpenStack client, `kubectl` to interact with Kubernetes, and `helm` for packaging apps.  

2. **Setting up a Kubernetes cluster**  
   - Used OpenStack’s Magnum service to spin up a Kubernetes cluster.  
   - Verified that master and worker nodes were healthy.  
   - Assigned DNS names to worker nodes via CERN’s network services so they could be reached easily.  

3. **Certificates and Security**  
   - Downloaded CERN’s root and intermediate security certificates.  
   - Built a certificate chain to ensure that all communication with the cluster was encrypted and trusted.  
   - Added these certificates as Kubernetes secrets, enabling secure HTTPS connections to the IEM service.  

4. **Installing Industrial Edge Management**  
   - Installed IEM using the **IE Provision CLI**, a wrapper around `helm` that simplifies setup.  
   - Configured settings such as hostname, storage classes, and certificate paths.  
   - Confirmed that the IEM dashboard was running and accessible through CERN’s internal network.  
   - Connected the local IEM with Siemens’ central cloud service (IE Hub).  
   - This allowed devices at CERN to be managed locally but also remain linked to Siemens’ ecosystem.  

### Impact
- Introduced **Industrial Edge concepts** to CERN’s industrial controls systems group.  
- Provided a **scalable and reproducible deployment method** that others can follow.  
- Served as a **practical introduction to Kubernetes and Helm** in an industrial context.  
- Created a solid foundation for future **edge computing experiments and use cases** at CERN.  

---

### Summary
- OpenStack provides flexible cloud resources inside CERN’s infrastructure.  
- Kubernetes ensures applications keep running and scale when needed.  
- Helm and IE Provision CLI simplify installation and updates.  
- Certificates and DNS integration provide production-grade security and accessibility.  

1. **Infrastructure**
   - **Platform:** CERN’s private cloud (OpenStack) hosts the virtual machines.  
   - **Storage:** Extra storage provided by Ceph volumes.  
   - **Networking:** DNS aliases assigned via CERN’s LANDB system.  

2. **Orchestration**
   - **Cluster creation:** Built with OpenStack Magnum.  
   - **Nodes:** Master and worker nodes run as virtual machines.  
   - **Resources:** Pods, Services, Deployments, Ingress (for external access), ConfigMaps, and Secrets.  
   - **Deployment:** Applications packaged and deployed with Helm charts.  

3. **Security**
   - **Certificates:** CERN’s internal certificate authority integrated into the cluster.  
   - **TLS secrets:** Stored as Kubernetes secrets for encrypted traffic.  

4. **Deployment Tools**
   - **IE Provision CLI:** Simplified installer for IEM.  
   - **IECTL:** Command-line client to connect IEM to Siemens cloud (IEHub).  
   - **kubectl + helm:** Standard Kubernetes management tools.  

**Architecture**
- **Operator VM:** Ubuntu machine running CLI tools for cluster creation and IEM install.  
- **Orchestration Layer:** OpenStack Magnum provisions K8s master and worker nodes.  
- **Cluster Layer:** Kubernetes orchestrates the IEM containers.  
- **Access Layer:** Ingress with TLS certificates provides secure access to the IEM dashboard.  


