+++
title = "Edge Computing Platform on Kubernetes"
author = "Abhit"
date = 2024-06-14
tags = ["cern", "edge-computing", "infrastructure"]
categories = ["projects", "CERN"]
+++

Imagine a factory or research laboratory filled with machines, each equipped with its own sensors and controllers. Traditionally, data from these devices is sent to central servers for processing. While effective, this approach often introduces delays and network bottlenecks. [Industrial Edge Computing](https://www.suse.com/c/edge-computing-the-key-to-smarter-industrial-automation/) addresses the problem by bringing computation closer to where data is generated. Instead of relying on distant servers, applications run directly on edge devices installed near the equipment, allowing for real-time decision-making. These industrial PCs can host applications that collect diagnostics, perform analytics, or even take direct control of equipment. For example, a ventilation system can adjust airflow more efficiently by running advanced control algorithms locally on an edge device using nearby sensor readings.
<!--more--> 

However, deploying hundreds of Edge Devices across CERN introduces a new challenge: how to coordinate and manage them effectively. This is where Industrial Edge Management (IEM) comes in. Acting as a central "control tower," IEM provides a web-based dashboard and back end that enable administrators to onboard new devices, deploy and update applications, and monitor system health, all from a single location.

At CERN, building this management platform inside the cloud meant bringing together several infrastructure layers. OpenStack provided the virtual machines to host IEM, while Kubernetes orchestrated the containers running on those machines, ensuring that its applications remained consistent and reliable.

## Deployment on Private Cloud

The task was to set up Siemens [Industrial Edge Management](https://docs.eu1.edge.siemens.cloud/get_started_and_operate/industrial_edge_management/overview.html) (IEM) on CERN's [OpenStack cloud](https://clouddocs.web.cern.ch/index.html) entirely from scratch. Doing so came with several challenges: selecting the right deployment tools and workflow, bringing team members up to speed with Kubernetes concepts and practices, creating a reliable and reproducible step-by-step method, and ensuring seamless integration with other existing services, including networking, DNS, and security certificates.

<p id="fig-iem-infra" align="center">
  <img src="/images/iem-infra.jpg" alt="IEM Infra" width="100%">
  <br>
  <em>CERN private cloud infrastructure used to deploy Kubernetes applications</em>
</p>

We carried out the deployment through a series of structured steps, moving from infrastructure setup to a fully operational platform.
1. **Provisioning an Operator VM on OpenStack**  
Using OpenStack command-line tools, we created a Linux VM, attached additional storage, and installed key utilities such as the OpenStack client, kubectl, and Helm. This machine served as the foundation for building and managing the Kubernetes environment.

2. **Setting up a Kubernetes cluster**  
With OpenStack's Magnum service, we deployed a Kubernetes cluster and verified that the controller and worker nodes were healthy. Worker nodes were assigned DNS names through CERN's network services, ensuring accessibility and stability.

3. **Certificates and Security**  
CERN's root and intermediate certificates were integrated into the cluster as Kubernetes secrets, enabling secure HTTPS communication. This step ensured compliance with CERN's strict security standards and protected sensitive operations.

4. **Installing Industrial Edge Management**  
Finally, we deployed IEM using the IE Provision CLI (a wrapper around Helm), configuring parameters such as hostname, storage classes, and certificate paths. Once verified on CERN's internal network, the local IEM was linked to Siemens' central IE Hub, enabling CERN to manage its devices locally while remaining connected to Siemens' global ecosystem.

## Laying the Groundwork

This work had an impact not only within the technical team but also on CERN's broader approach to industrial computing. It introduced Industrial Edge concepts to the [Industrial Controls Systems group](https://beams.cern/ics), opening new possibilities for managing equipment closer to where data is generated. The project also established a scalable and reproducible deployment method that other teams can adopt, while providing team members with hands-on experience with Kubernetes and Helm in an industrial context. Most importantly, it created a strong foundation for future edge computing experiments and potential use cases across CERN. Together, these outcomes went beyond solving the immediate task of deploying Industrial Edge Management, positioning CERN to explore wider opportunities in industrial edge computing.
