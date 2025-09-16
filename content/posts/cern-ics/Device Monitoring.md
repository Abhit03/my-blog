+++
title = "Monitoring Control Infrasturcuture at CERN"
author = "Abhit"
date = 2024-08-11
tags = ["cern", "ics"]
categories = ["projects"]
+++

At CERN, the operation of accelerators and experiments depends on thousands of technical devices working together. These range from:  
- **PLCs (programmable logic controllers):** industrial computers that run control logic,  
- **Power supplies:** that feed magnets and detectors,  
- **Front-end controllers:** that handle communication between equipment and higher-level systems.  

These devices are supplied by many different vendors, such as Siemens, Schneider, and Beckhoff, and each vendor provides its own way of reporting the device’s status or error messages.  

Until recently, operators could only view device health in flat dashboards, i.e, a long table or grid with thousands of entries. While this showed the state of each device individually, it gave no sense of hierarchy or how devices were related to each other.  

This created challenges:  
- If one device failed, it was not clear how that affected the larger subsystem it belonged to.  
- Operators had to manually scan long lists of devices to detect issues.  
- There was no intuitive way to see how local faults could escalate into larger problems.  

What was missing was a monitoring system that could show devices in their natural hierarchy from small components all the way up to major subsystems and, eventually, the entire CERN accelerator complex.

### Problem Statement
We set out to build a **hierarchical monitoring platform** where:  
- Devices can be arranged in a tree structure that reflects how CERN systems are actually organized (e.g., device → subsystem → sector → accelerator).  
- Users can define logical rules for groups, e.g., “if any child device has an error, the parent node turns red”.  
- Errors propagate upward in the tree, so operators can instantly see high-level health and then drill down to the faulty device.  
- The tree can be expanded or collapsed, letting operators start from a big-picture view and zoom in only where needed.  

---

### Application components

1. **Frontend (Web UI)**
   - **Framework:** React.  
   - **State Management:** Evaluated Redux, Zustand, and Jotai → chose **Jotai** because it naturally fits tree-like structures.  
   - **Tree Rendering:** Each node has its own state; React components recursively render the hierarchy.  
   - **Rule Editor:** Simple UI for creating logical rules (e.g., AND/OR, “at least N devices must be healthy”).  
   - **Persistence:** Uses browser local storage, so work is not lost between sessions.  

2. **Backend**
   - **Language:** Go (chosen for its simplicity and efficient concurrency).  
   - **Responsibilities:**  
     - Maintains the device tree.  
     - Collects child states, applies rules, and propagates results upward.  
     - Interfaces with existing diagnostic tools that query real devices.  
   - **Communication:** Uses gRPC for node-to-node updates.  
   - **Logic Engine:** Built in OCaml, supports expressive logical rules beyond simple AND/OR.  

3. **Central Management Server**
   - **Language:** Go.  
   - **REST API:** Provides endpoints for the frontend to deploy configurations and retrieve monitoring data.  
   - **Advantages:**  
     - Separates the deployment logic from the monitoring backend.  
     - Makes it easy to add other clients in the future (notifications, alternative dashboards, analytics).  

### Functionality
- **Design Mode:** Users build or edit the tree, assign devices, and define rules through a web interface.  
- **Deploy Mode:** The defined configuration (devices + rules) is exported and pushed to the backend system.  
- **Monitor Mode:** Operators see the live device tree in color e.g. green=OK, yellow=warning, red=error. Device states automatically flow upward to higher levels in the tree.  
- **Customization:** Import or export tree structures and device configurations via JSON files, and define how device or group states are mapped to specific colors.
- **Scalability:** The system handles thousands of nodes, and operators can easily collapse branches to focus on what matters.  

### Impact
- Operators no longer need to scan **thousands of devices one by one**.  
- A single red node at the top level immediately signals a problem, guiding the operator to investigate further.  
- Rules are customizable, so the system adapts to different use cases.  
- While built for CERN’s control devices, the same approach applies to **any large industrial setup** where device health must be monitored in layers.  
---

### Summary 

**Architecture**
- **Frontend:** React + Jotai for interactive tree visualization and rule editing.  
- **Backend:** Go server to run monitoring logic, query devices, and apply rules.  
- **Logic Engine:** OCaml engine to evaluate advanced logical rules.  
- **Device drivers:** Drivers (C++) to query diagnostic data from PLCs.  