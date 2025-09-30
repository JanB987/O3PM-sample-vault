---
aliases:
tags:
  - object/entity
created: 2025-09-02
type:
  - "[[Use Case]]"
context:
status: "[[DONE]]"
---
# Content
Article: https://nosy.science/2025/09/30/longformstemplater/
## Use Case
- Create a single longform note from the content of multiple notes based on links in a master note into a single linked note in the order they are linked in. 
- Clean up the resulting accumulated content to prepare it for exporting by removing
	- wikilinks
	- frontmatter
	- headers
	- comments
	- strikethroughed text blocks
- allow the (de)activation of any cleanup step

## File setup
Frontmatter:
export: "[[Test Export Note]]"
	"Test Export Note" is the target note for the MD-embeds.
	"Test Export Note" has to exist already before running the master template.
	
pandoc_reference: "[[00. SYSTEM/Pandoc Templates/MW1-reference.docx]]"
	Reference File for the pandoc export to DocX
pandoc_export: "Test File"
	Name of the target file for the pandoc export.



## Templates
### Base file
- [[TEMPL - Master - Embed OR Combine OR Merge Linked Files and clean them - V1]]
	- Master file that links all scripts
- [[TEMPL - Embed Linked Files - V1]]
	- Embeds the linked files
- [[TEMPL - Clean - WikiLinks - V1]]
	- Removes wikilinks
- [[TEMPL - Remove - Frontmatter - V1]]
	- Removes Frontmatter by looking for instances of "---" with properties in between them. Can therefore run after the files have been embedded. 
- [[TEMPL - Remove - Comments - V1]]
	- Removes all instances of "%% Comment %%"
- [[TEMPL - Remove - StrikeThrough - V1]]
	- Removes all instances of "~~Strikethrough~~"
- [[TEMPL - Remove - Tasks- V1TEMPL - Remove - StrikeThrough - V1 1]]
### Header removal scrips
The following scripts remove all headings of their specific level. 
- [[TEMPL - Remove - Heading 1 - V1]]
- [[TEMPL - Remove - Heading 2 - V1]]
- [[TEMPL - Remove - Heading 3 - V1]]
- [[TEMPL - Remove - Heading 4 - V1]]
- [[TEMPL - Remove - Heading 5 - V1]]
- [[TEMPL - Remove - Heading 6 - V1]]

### Export Files
- [[TEMPL - Pandoc - Export to Word - V1]]
- 


## Motivation 
- Better extendability through templates
- Just build on templates that can be accessed and worked on from anywhere. (?Not sure about this one)
- I understand them
- Low friction to work with and modify
- Fits better into my workflow
- Fits better into obsidian's interface without extra forms
- Doesn't clutter the frontmatter.
- Scenes can be (de)activated with a simple addition or removal of a property
- Scenes can be commented on
- Works easy and equally well on desktop and mobile
- Direct Export via Pandoc possible. 

