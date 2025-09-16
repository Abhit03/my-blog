+++
title = "Computing for Industrial Control Systems"
author = "Abhit"
date = 2025-03-25
tags = ["cern", "ics"]
categories = ["projects"]
+++

### The challenge in CERN’s technical infrastructure
CERN operates thousands of devices like programmable logic controllers (PLCs), power supplies, cooling and ventilation units, and other control equipment that keep the accelerator complex running safely and efficiently.  As the number and diversity of devices grew, so did the challenges:  
- How can operators monitor such a large, heterogeneous infrastructure intuitively?  
- How can advanced computing approaches like edge computing be integrated into this environment?  

My work in the Industrial Control Systems group within the beams department at CERN addressed these questions across two fronts:  
1. **Infrastructure monitoring at scale** - Building a new way to visualize and supervise thousands of devices.  
2. **Industrial Edge computing** - Deploying Siemens’ Industrial Edge Management and running advanced algorithms like Model Predictive Control on edge devices.  

---

## Part 1 – Device Monitoring Platform

The first challenge was monitoring. Existing dashboards showed devices in flat lists or grids, making it hard to see how failures in one part of the system affected the bigger picture.  

I worked on a **hierarchical monitoring platform** that represented devices in a **tree structure** — from individual devices up through subsystems and sectors. This allowed operators to:  
- See high-level system health at a glance.  
- Define **rules** (e.g., “if any child fails, mark the parent as failed”) so errors propagate automatically.  
- Drill down interactively from a red node at the top level to the exact failing device.  
- Scale up to hundreds or thousands of devices without overwhelming the view.  

**Impact:** Operators no longer had to manually scan endless lists. Instead, they could spot issues intuitively, follow error propagation through the hierarchy, and resolve problems faster.  

---

## Part 2 – Industrial Edge Computing at CERN

With monitoring addressed, the next focus was **bringing more intelligence to control systems** through edge computing. In collaboration with Siemens, I led work in two key areas: deploying the management platform and testing a real edge application.

### A. Deploying Industrial Edge Management
Industrial Edge Management (IEM) is Siemens’ platform for onboarding, updating, and supervising fleets of Industrial Edge Devices (industrial PCs). IEM acts as the “control tower,” while the edge devices are the “workers” running applications close to the machines.  

I deployed IEM **from scratch on CERN’s private cloud**, combining:  
- **OpenStack** to provision Linux virtual machines.  
- **Kubernetes (via OpenStack Magnum)** to orchestrate the containerized IEM services.  
- CERN’s own **certificate authority and DNS services** for secure, production-ready deployment.  

This provided a **scalable, reproducible setup** for edge management and gave the team hands-on exposure to Kubernetes in an industrial controls context.  

---

### B. Running Model Predictive Control on Edge Devices
Once IEM was in place, we needed to prove that **real applications** could run on Industrial Edge Devices. The use case was **Model Predictive Control (MPC)** for an air handling unit — a smarter alternative to simple PLC feedback rules.  

The challenge: MPC requires solving heavy optimization problems in real time, far beyond what a PLC can do.  

My contributions included:  
- Packaging the MPC algorithm as a deployable **edge app** using Siemens’ Template Framework.  
- Setting up data pipelines with **S7 Connector**, **MQTT Databus**, and dashboards built in **Node-RED/Grafana**.  
- Optimizing performance by compiling the CasADi library for the device’s CPU, testing different **numerical solvers** (IPOPT, CPLEX, Gurobi), and applying warm-start strategies.  
- Validating runtime with ~28,000 test cases on CERN’s HPC cluster.  

**Impact:** The project demonstrated that advanced optimization can run reliably on industrial PCs at the edge, enabling more energy-efficient control strategies while keeping decisions close to the process.  

---

## In summary
- **Device Monitoring Platform:** delivered a scalable, intuitive way for operators to monitor thousands of devices hierarchically.  
- **Industrial Edge Management Deployment:** established the foundation for managing edge devices and apps at CERN using OpenStack and Kubernetes.  
- **MPC on Edge Devices:** proved that heavy algorithms like Model Predictive Control can run on industrial PCs, bridging the gap between traditional PLCs and modern optimization.  

Together, these projects trace a progression:  
- From **better visibility** (infrastructure monitoring),  
- To **better management** (edge orchestration),  
- To **better intelligence** (advanced control at the edge).  

This reflects the group’s shift toward **scalable monitoring, edge computing, and intelligent automation** — ensuring CERN’s technical backbone evolves to meet future demands.  
