+++
title = "Managing Energy Demand"
author = "Abhit"
date = 2016-10-09
tags = ["energy"]
categories = ["projects", "shared-electric"]
+++

Electricity grids face a growing challenge: demand rarely matches supply. Evening peaks arrive just as solar output drops, while midday often brings excess solar generation when demand is modest. Traditionally, utilities have bridged this gap with fossil fuel plants, an approach that is both costly and carbon-intensive. As renewable energy continues to scale, a smarter path is emerging in the form of demand-side management (DSM). By [shifting](/posts/shared-electric/load-shifting)  when and how people use energy, DSM helps bring demand closer to supply. Instead of treating consumers as passive users, it engages them directly in balancing the grid.

While DSM is effective in theory, people need to be actively involved to make it work at scale. At Shared Electric, that became our starting point for building a pilot system called Engaze. It enabled utilities to request small shifts in energy usage while engaging consumers through timely nudges via app notifications. The system served as a testbed to demonstrate how behavioral demand response could work in practice.
<!--more--> 

## Implementation

Engaze brought together analytics and consumer engagement in a single system. On the utility side, it included a dashboard for operators to enter desired load changes and a backend layer to identify target households best positioned to provide flexibility. On the consumer side, it featured an app that allowed users to set personal goals and receive notifications at the right moments.

A key component of the system was an analytics engine I developed, which powered decision-making across the platform. Its capabilities included:

- **Time-series analysis**: Processing over 30 million hourly consumption data points from hundreds of installations over the course of a year to forecast usage patterns and identify flexible loads.
- **Clustering & targeting**: Grouping users by past behavior to ensure the right participants were selected for each event.
- **Engagement tracking**: Comparing actual consumption against forecasts to measure outcomes and refine future targeting strategies.

These capabilities ensured that Engaze could deliver reliable insights and guide both utilities and consumers in real time.  

<p id="fig-engaze" align="center">
  <img src="/images/engaze_report.jpg" alt="IoT components" width="100%">
  <br>
  <em>Report generated in Danish for an Engaze user</em>
</p>

## Reflections

This project showed that demand side management is not just a technical challenge but also a human one. On the technical side, we demonstrated that large-scale consumption data could be analyzed to guide decision making. On the human side, we found that simplicity and timing mattered as much as algorithms. A well-timed, easy-to-follow nudge often worked better than a complex incentive scheme. For utilities, the platform proved that behavioral demand response could be treated as a real, quantifiable resource, one that reduces costs and strengthens engagement with customers.

It offered an early glimpse of how future energy systems may evolve: digital, data-driven, and participatory, with consumers playing an active role in balancing supply and demand. Beyond the numbers, the project also revealed deeper lessons about the nature of demand-side management.

> Managing demand is no longer just a utility task; it is a shared responsibility.
