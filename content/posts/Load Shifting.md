+++
title = "Load Shifting Calculator"
author = "Abhit"
date = 2025-09-01
tags = ["energy"]
categories = ["projects", "case-studies"]
+++

# Interactive Energy Shifting Calculator to optimize costs and CO₂ savings

One of the biggest challenges for renewable energy adoption is that **demand doesn’t always match supply**.  
- Solar peaks at midday, but residential consumption peaks in the evening.  
- Wind can be abundant at night, but industry runs during the day.  

Utilities and customers both struggle to see *how much money or CO₂ could be saved* by simply shifting energy usage from one time to another.  

We wanted to solve this problem with a simple, visual tool:  
- That would instantly calculate **optimal costs and CO₂ savings** if energy consumption were shifted.  
- And something that would make a technical concept — demand-side response — accessible to a **non-technical audience**.  
---

## Modeling the Problem

Before building the tool, we needed a clear **model of the problem**.  

- **Load Profile**: A 24-hour vector showing how much electricity is consumed each hour.  
- **Flexibility Profile**: How much consumption can be shifted up or down at each hour. Some hours may have zero flexibility, others might allow 10–20%.  
- **Marginal Cost**: The price of generating 1 extra kWh in a given hour
- **Marginal CO₂**: The emissions from generating that same extra kWh, depending on which power plants are online.  

The challenge:  
- Given these profiles, what is the **Change Profile** (when to reduce, when to increase consumption) that results in the lowest overall cost, emissions, or a balance of both?  
- And how can we compute this quickly enough to show results *live* in a browser?  

---

## The Brick & Wall Analogy

To explain this problem in a way that anyone could understand, we developed the **“brick and wall” analogy**.  

- Think of daily electricity demand as a **wall with 24 columns** — one for each hour of the day.  
- Each column must be filled up to a certain height (the load).  
- To fill the wall, we have a set of **bricks** — each brick representing a power plant.  
  - The **size** of the brick = how much the plant can generate.  
  - The **color** = which plant it comes from (nuclear, coal, solar, gas, etc.).  
  - The **weight** (or cost per unit) = how expensive it is to use.  
  - The **quality** = its CO₂ emissions per unit.  

The optimization problem becomes:  
- How do we fill the wall with the available bricks, while respecting minimum/maximum limits, flexibility rules, and costs?  
- Which bricks do we use first if we want to minimize cost?  
- Which bricks do we use if we want to minimize CO₂?  
- And how does the result change if we consider a trade-off between the two?  

This analogy made a complex optimization problem **visual and intuitive**.  
It was also the basis for the **interactive tool**, where users could actually *see* the effect of shifting consumption as colored bars in a chart.  


## The Algorithm: Three-Stage Optimization

Once the model was clear, we needed an algorithm that could compute the optimal allocation of demand quickly and transparently.  
We designed a **three-stage optimization process**:

1. **Stage 1 – Minimum requirements**  
   - Every power plant has a minimum output (they can’t simply turn off).  
   - We first filled each hour’s load with the mandatory minimums.

2. **Stage 2 – Flexibility constraints**  
   - We then allocated demand up to the **flexibility limits** for each hour.  
   - This respected the user-defined flexibility profile (how much load can be shifted up or down).  

3. **Stage 3 – Cost- and CO₂-based allocation**  
   - For the remaining demand, we selected the **most cost-effective** or **lowest-emission** bricks first.  
   - The algorithm could optimize for **cost**, **CO₂**, or a **weighted balance of both**, depending on user input.  

### JavaScript Implementation

The algorithm was implemented in **JavaScript** so it could run entirely in the browser.  

```js
function optimize() {
  optimize1(); // Stage 1: mandatory minimums
  optimize2(); // Stage 2: fill up to flexibility profile
  optimize3(); // Stage 3: allocate cheapest/cleanest remaining
  updateResults(); // prepare stacked bar data for visualization
}
```

## Building the Interactive Tool

We developed an **interactive user interface** where clients could play with inputs and instantly see results.  

- Implemented entirely in **JavaScript + web technologies**, so it could run in any browser without setup.  
- Input fields allowed users to provide the data, such as  Load profile, Flexibility profile and costs
- The tool highlighted the **recommended shifts**:  
  - Hours where usage should be **reduced** (most expensive).  
  - Hours where usage should be **increased** (cheapest/cleanest).  
  - 
This visualization made it clear, at a glance, where the biggest savings opportunities were.

---

## Results

The results consistently showed that even modest demand shifts could lead to significant benefits:

- **10–20% potential cost savings** in typical case studies.  
- Noticeable reductions in **CO₂ emissions**, especially when shifting demand into hours with high renewable availability.  
- A more intuitive understanding for clients of how **flexibility translates to value**.  

This made the business case for demand-side management **tangible, interactive, and persuasive**.  


## Reflections

Looking back, the Energy Shifting Calculator was more than just a coding exercise — it was about making **complex energy concepts simple and actionable**.  

A few key takeaways stood out:

- **Invent & Simplify**: By turning a dense optimization problem into the “brick & wall” analogy, we created a model that anyone, from engineers to clients, could understand.  
- **Visualization matters**: A single interactive chart explained savings in seconds, far better than spreadsheets or equations ever could.  
- **Bridging two worlds**: This project showed how **software engineering and energy systems** can complement each other. Clean, modular code powered a tool that directly influenced how customers thought about renewable integration and flexibility.  

Most importantly, this project taught me that **impact doesn’t always require massive infrastructure** — sometimes, a simple interactive tool can shift perspectives, drive decisions, and pave the way for smarter, greener energy systems.  
