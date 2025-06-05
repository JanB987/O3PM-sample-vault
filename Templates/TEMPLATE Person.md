---
aliases: <%* let title = tp.file.title;
	let title_new;
	if (title.startsWith('@ ')) {
		title = title.replace("@ ","");;
	}
	else {
		title_new = "@ " + title
		await tp.file.rename(title_new)
	}	
	tR += "\n - " + title + "\n"; -%>
tags:
- object/intel/person
created: <% tp.date.now('YYYY-MM-DD') %>
phone:
mail:
---


