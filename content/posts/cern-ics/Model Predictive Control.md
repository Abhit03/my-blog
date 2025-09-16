+++
title = "Control Algorithm on Edge"
author = "Abhit"
date = 2023-10-07
tags = ["cern", "ics"]
categories = ["projects"]
+++

At CERN, large technical systems such as cooling plants, ventilation systems, and safety installations must operate continuously around the clock. These systems are controlled by programmable logic controllers (PLCs). PLCs are specialized industrial computers designed to execute the same instructions continuously and reliably, often for years without interruption. Built with safety and stability in mind, they serve as the backbone of industrial automation. <!--more--> 
For example, they can:
- Control pumps that circulate cooling water for accelerators.
- Adjust ventilation fans to ensure underground tunnels are safe to enter.
- Activate interlocks that immediately shut down equipment if unsafe conditions are detected.

The control logic inside a PLC is usually based on **simple feedback rules**. A common example is:  *“If the temperature goes above a threshold, increase the fan speed; if it drops too low, reduce the fan speed.”* These rules, often implemented as PID controllers, work well for straightforward, repetitive tasks. But they fall short when situations become more complex, such as when:  
- Multiple variables like temperature, humidity, and energy cost interact 
- Future conditions need to be anticipated
- There are competing goals, such as comfort vs. energy efficiency.  

### A more advanced method: Model Predictive Control
To handle such complexity, control engineers use **Model Predictive Control (MPC)**.  
- MPC builds a **mathematical model** of the system (e.g., how temperature will change if fan speed is adjusted).  
- It **predicts future behavior** of the system over the next minutes or hours.  
- It then **chooses the optimal control actions** that minimize cost like energy consumption, while still respecting limits such as safety margins or comfort requirements.  

This makes MPC significantly smarter and more efficient than traditional methods. For instance, an MPC controller managing a ventilation system can proactively increase fan speed before a large group enters a hall, instead of waiting for CO₂ sensors to signal a problem.

### Why not just run MPC on PLCs?
The drawback is that MPC is **computationally demanding**. Solving optimization problems in real time requires far more computing power than a PLC can provide. This is where **Industrial Edge Devices** come in. These small industrial PCs are located near the equipment and connect to the same network as the PLCs. They are powerful enough to run complex algorithms like MPC. Because they operate at the 'edge' (directly beside the machines), they can make quick decisions without sending all raw data to distant central servers.

In other words:  
- **PLCs** remain the reliable backbone for safety and routine control.  
- **Edge Devices** provide the intelligence needed for advanced optimization, bridging the gap between central computing and field-level control.  

### Problem Statement
- **Goal:** Execute and test an **MPC algorithm** for an Air Handling Unit (part of CERN’s HVAC system) directly on an Industrial Edge Device.  
- **Challenge:** While theoretically effective, the algorithm was too resource-intensive to run in real time on a small industrial PC without optimization.
- **Task:** Redesign the deployment and optimize performance so the algorithm can run reliably within the device’s limited computing resources.  

---

### Implementation Steps
1. **Setting up the Edge Device**  
   - Installed Siemens apps to handle communication:  
     - **S7 Connector**: talks to PLCs.  
     - **Databus (MQTT broker)**: lets different apps on the device exchange messages.  
   - Added operator tools:  
     - **Node-RED** for building and displaying simple dashboards.  
     - **InfluxDB + Grafana** for time-series data logging and visualization.  

2. **Packaging the Algorithm**  
   - Used Siemens’ **Template Framework** to wrap the MPC code written in Python into an app that could run on the Edge Device.  
   - This involved:  
     - Writing functions in Python and annotating them as different steps in a process, for example, a **training step** where the model learns, or a **prediction step** where the model makes decisions.  
     - Using the **Builder**tool to package everything into a zip file.  
     - Uploading the package via FTP to the device.  
     - Triggering the functions via MQTT messages.  
   - Operators could view the results through Node-RED dashboards.  

3. **Optimizing for Edge Hardware**  
   - Compared performance on a developer laptop vs. the small Edge Device (with an Intel Celeron CPU).  
   - Improvements included:  
     - Compiling the mathematical optimization library **CasADi**  specifically for the device’s CPU.  
     - Trying different **numerical solvers** (such as IPOPT, CPLEX, and Gurobi), which are specialized software libraries for solving optimization problems.
     - Using techniques like **warm start** (reusing previous results to speed up the next calculation).  
   - Achieved execution times of under ~17 seconds for 95% of cases — fast enough for practical use.  

4. **Validation at Scale**  
   - Tested the system with ~28,000 input scenarios on CERN’s HPC cluster to mimic real workloads using the SLURM framework.  
   - Identified rare cases where the solver failed or was too slow and proposed fallbacks (e.g., temporarily handing control back to the PLC).  
   - Results were published as a **peer-reviewed paper and poster** at an international conference.  

### Impact
- Showed that **advanced algorithms** can be deployed on industrial PCs at the edge — not just in research code or central servers.  
- Enabled **energy savings** in HVAC operations, which is critical given the scale of CERN’s infrastructure.  
- Brought together two communities:  
  - **Control engineers**, who work with PLCs and industrial reliability.  
  - **Data scientists**, who use modern Python libraries and optimization frameworks.  
- Created a **repeatable process (DevOps for Edge)** for packaging, deploying, and monitoring the functions in a production-like environment.  

---

### Summary
- Edge devices bring computation closer to the process, reducing latency and dependence on central servers.  
- Containerization ensures apps can be redeployed and scaled consistently.  
- Siemens’ Template Framework standardizes packaging and makes lifecycle management easier.  
- Optimization + validation ensured the algorithm wasn’t just theoretically correct but **practically usable on small industrial hardware**.  

1. **Edge Device**
	- **Hardware:** Siemens Industrial PC (small, rugged, factory-ready computer).  
	- **OS & Runtime:** Debian-based Linux with Siemens Industrial Edge Runtime.  
	- **Apps:** MPC algorithm, PLC connectors, dashboards — all deployed as Docker containers.  

2. **Communication**
	- **Between PLC and Edge:** S7 Connector + MQTT broker.  
	- **Between Edge and User:** Node-RED dashboards for quick visualization.  
	- **Data Logging:** InfluxDB database with Grafana dashboards for monitoring.  

3. **Deployment Pipeline**
	- **Template Framework:**  
		  - SDK to mark stages in Python notebooks.  
		  - Builder to turn them into packaged apps.  
		  - Runtime on the device to execute them.  
	- **FTP:** Used to upload packaged apps.  
	- **MQTT:** Used to invoke the functions.  
	- **Web UI:** Operator dashboards for monitoring.  

4. Optimization
	- **Libraries:** CasADi (math library for optimization).  
	- **Solvers:** IPOPT (default nonlinear solver), with options to test CPLEX and Gurobi (commercial solvers).  
	- **Techniques:** Warm start, parallelization, and CPU-specific builds.  
	- **Validation:** Tested with thousands of scenarios on HPC clusters before deployment.  

**Architecture**
- **Field layer:** Sensors, actuators, PLCs.  
- **Edge layer:** Siemens Edge Device running the MPC algorithm and connector apps.  
- **Management layer:** Industrial Edge Management for managing the Edge devices and apps.  
- **Visualization layer:** Grafana dashboards and Node-RED panels.  
- **Validation layer:** HPC cluster for large-scale testing before going live.  


