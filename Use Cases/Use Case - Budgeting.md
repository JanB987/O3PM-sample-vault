---
aliases:
tags:
- object/info
created: 2025-06-06
context:
  - "[[PRJT - Sample Project]]"
type:
  - "[[Class Use Case]]"
status: "[[active]]"
icon: 📊
---
# Info
# Content

### Relational Table
```dataview
table without ID
	cost_info.section AS Section,
	cost_info.status AS Status,
	file.link AS "Cost Carrier",
	replace(cost_info.net + " €", ".", ",") AS Amount,
	short as "Descr.",
	cost_info AS "Cost Info",
	cost_info.source as Source,
	cost_info.account AS Account
FROM [[PRJT - Sample Project]]
WHERE cost_info
```

## Description
Budgeting for projects or other purposes. 
To learn more about the module and its use you can check out the Article on [Nosy.Science](https://nosy.science/).



## Budgeting via Bases

Article: https://nosy.science/2025/08/19/budget-with-bases/

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

Article: https://nosy.science/2025/08/25/budget-with-dataview/


### Table
```dataview
table without ID
	section,
	status,
	file.link AS "Cost Carrier",
	replace(net + " €", ".", ",") AS Amount,
	source AS Source,
	account AS Account
FROM [[PRJT - Sample Project]] AND [[Class Cost]]
SORT status DESC
```

