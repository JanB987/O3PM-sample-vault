---
aliases:
- Person 2
tags:
  - object/person
company: "[[@ Sample Company]]"
phone: 
mail: 
created: 2025-05-30
---
## Object Responsibilities
```dataview
TABLE WITHOUT ID 
	file.link + " | " + "[[" + file.name + "#" + meta(T.section).subpath + "|" + meta(T.section).subpath + "]]" AS "File & Heading",
	T.text AS Text

FROM [[]]

FLATTEN file.tasks as T

WHERE (contains(T.status, "<") OR contains(T.status, "/")) AND !T.completed AND contains(T.text, this.file.name)

SORT 
	urg DESC, 
	prio DESC
```
