---
aliases:
tags:
  - class
created: 2025-08-15
type:
  - "[[Class Class]]"
context:
---
# Content

## Properties

- net: Cost amount. 
- Responsible: The person responsible for the object's cost information.
- source: Source of the cost information. Can be an estimate (based on experience), an offer or an invoice. 
- status: Status of the cost.
- category: Category the cost information belongs to, e.g. external costs. 
- section: Another means to cluster cost information together. 
- status: Status of the cost information. 

## Instances
### Base
```base
views:
  - type: table
    name: Table
    filters:
      and:
        - type.contains(link("Class Cost"))
    order:
      - file.name
      - net
	  - source
	  - status

```