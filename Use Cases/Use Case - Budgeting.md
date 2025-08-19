---
aliases:
tags:
- object/info
created: 2025-06-06
context:
  - "[[PRJT - Sample Project]]"
type:
  - "[[Use Case]]"
status: "[[active]]"
---
## Description
Budgeting for projects or other purposes. 
To learn more about the module and its use you can check out the Article on [Nosy.Science](https://nosy.science/).

## Budgeting via Bases

Article: 

### Table
```base
views:
  - type: table
    name: Table
    filters:
      and:
        - context.contains(link("PRJT - Sample Project"))
        - type.contains(link("Class Cost"))
    order:
      - file.name
      - section
      - category
      - gross
      - net
      - source
      - status
    columnSize:
      file.name: 306
    rowHeight: medium

```


## Dataview

```dataview
table without ID
	section,
	status,
	file.link AS Cost,
	replace(net + " €", ".", ",") AS Amount,
	source
FROM [[PRJT - Sample Project]] AND [[Class Cost]]
SORT status DESC
```


