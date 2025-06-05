---
aliases:
tags:
- object/info
created: 2025-05-30
type:
  - "[[Class Gantt Chart]]"
context:
  - "[[PRJT - Sample Project]]"
status: "[[DONE]]"
---
# Content
## Description
The mermaid code in this file (see section [[#Gantt]]) generates a gantt chart. 
If you want to learn more about how the diagram works you can read up on it on the [official Mermaid site](https://mermaid.js.org/syntax/gantt.html). Or check out the article [Gantt-charts in Obsidian through Mermaid](https://nosy.science/2025/04/29/gantt-charts-in-obsidian-through-mermaid/) on [Nosy.Science](https://nosy.science/) to learn more about the use of gantt charts in [O3PM](https://nosy.science/2025/05/10/object-oriented-management-in-obsidian-o3pm/).

## Gantt

```mermaid
gantt
    dateFormat   YYYY-MM-DD
    axisFormat   %m
    tickInterval 1month
    title        Project Plan

section Something
AP Test 1 : crit, done, AP1, 2024-04-17, 10w
AP2 : crit, active, AP2, after AP1, 8w
AP3 : crit, AP3, after AP2, 12w

```