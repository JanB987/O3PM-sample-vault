---
aliases:
tags:
  - object/entity
created: 2025-09-04
type:
  - "[[Use Case]]"
context:
  - "[[Obsidian MD]]"
  - "[[Plugin]]"
status: "[[DONE]]"
---

## Use Case Description
When I train, I document my progress through workout files of the type [[TEMPL - Workout - V2]]. Lately, instead of creating new files I create copies of previous workout files such as the following one:
[[2025-09-02 Chin Ups - Home]]

But before I use it I have to update many entries, like the date, the title and the status. I want to automate these steps with another master template that calls other templates like the following: 
[[TEMPL COM update meeting]]

The modifications are the following:
1. Duplicate and open the current file
	- [[TEMPL COMP - Duplicate and Open File - V1]]
2. Change the date to today's date. 
	- [[TEMPL COM - Set Date To Today - V1]]
3. Change the date in the title to today's date. 
	- [[TEMPL COM update meeting]]. 
4. Update the status to "[[ToDo]]"

I will call that master template from the workout note that I want to modify. 






- Changes date to today
- Updates title
- changes status to todo
# Content
## Templates

- [[TEMPL - Master - Clean Up Workout Note - V1]]
- [[TEMPL COMP - Duplicate and Open File - V1]]
- [[TEMPL COM - Set Date To Today - V1]]
- [[TEMPL COM update meeting]]
- [[TEMPL Set Action to ToDo]]
