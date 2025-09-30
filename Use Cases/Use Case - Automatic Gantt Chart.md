---
aliases:
tags:
- object/info
created: 2025-05-30
type:
  - "[[Class Gantt Chart]]"
  - "[[Class Use Case]]"
version:
context:
  - "[[PRJT - Sample Project]]"
status: "[[DONE]]"
---
# Content
## Description
The query in this file (see section [[#Gantt]]) generates a gantt chart based on the properties of the linked files under section [[#Files|Files]]. 
Please note that the query might take a while to load. 
If you want to learn more about how the query works and its backgrounds, you can read up on it in the article [Automating Gantt Charts in Obsidian with Mermaid and Dataview](https://nosy.science/2025/05/04/automating-gantt-charts-in-obsidian-with-mermaid-and-dataview/) on [Nosy.Science](https://nosy.science/).

## Files
*Objects that are added to the Gantt Chart.*

- [[AP Test 1]]
- [[AP Test 2]]
- [[AP Test 3]]

## Gantt
```dataviewjs
// Mermaid Gantt configuration
const mermaidConf = `gantt
    dateFormat  YYYY-MM-DD
    excludes    weekends
    axisFormat  %m
    tickInterval 1month
    title Project Plan
    todaymarker on`;

// Get linked pages from the current note that have timeline data in YAML
const pages = dv.current().file.outlinks
    .map(link => dv.page(link.path))
    .filter(p => p?.timeline_type === 'current' || p?.timeline_type === 'planned');

// Parse simple durations like "3w", "2m" into day/week/month/year objects
function parseDuration(str) {
    const num = parseInt(str);
    const unit = str.slice(-1);
    switch (unit) {
        case 'd': return { days: num };
        case 'w': return { weeks: num };
        case 'm': return { months: num };
        case 'y': return { years: num };
        default: return { days: 0 };
    }
}

// Parse ISO 8601 duration strings like "P1M1W"
function parseISODuration(iso) {
    const match = iso.match(/^P(?:(\d+)Y)?(?:(\d+)M)?(?:(\d+)W)?(?:(\d+)D)?$/);
    if (!match) return null;
    const [, y, m, w, d] = match.map(x => (x ? parseInt(x) : 0));
    return {
        ...(y ? { years: y } : {}),
        ...(m ? { months: m } : {}),
        ...(w ? { weeks: w } : {}),
        ...(d ? { days: d } : {})
    };
}

// Resolve a linked start or end date recursively if it points to another object
function resolveLinkedDate(value, condition, fallbackDate = dv.date('today'), visited = new Set()) {
    if (!value) return fallbackDate.toISODate();
    const preferStart = condition === 'start';
    const preferEnd = condition === 'end';

    const resolvePageName = (val) => {
        if (val?.path) return val.path;
        if (typeof val === 'string' && val.startsWith('[[')) {
            return val.replace(/\[\[|\]\]/g, '');
        }
        return val;
    };

    // Handle duration strings like "10w"
    if (typeof value === 'string' && /^[0-9]+[dwmy]$/.test(value.trim())) {
        const parsed = parseDuration(value.trim());
        return fallbackDate.plus(parsed).toISODate();
    }

    // Handle ISO 8601 duration strings like "P2M"
    if (typeof value === 'string' && /^P.*$/.test(value.trim())) {
        const parsed = parseISODuration(value.trim());
        if (parsed) return fallbackDate.plus(parsed).toISODate();
    }

    if (value?.toISODate) return value.toISODate();

    const path = resolvePageName(value);
    if (!path || visited.has(path)) return fallbackDate.toISODate();
    visited.add(path);

    const linked = dv.page(path);
    if (linked) {
        const linkedDate = preferEnd ? resolveEndInfo(linked).end : resolveStartDate(linked);
        return linkedDate;
    }

    return fallbackDate.toISODate();
}

// Resolve start date using inline YAML timeline
function resolveStartDate(task, fallback = dv.date('today')) {
    return resolveLinkedDate(task.timeline_start, task.timeline_start_condition, fallback);
}

// Resolve end date or compute duration using inline YAML timeline
function resolveEndInfo(task) {
    const raw = task.timeline_end;
    const startDate = resolveStartDate(task);

    const adjustIfEqual = (end) => end === startDate ? dv.date(end).plus({ days: 1 }).toISODate() : end;

    if (
        typeof raw === 'object' &&
        raw !== null &&
        typeof raw.toString === 'function' &&
        raw.toString().startsWith('P') &&
        !raw.toString().startsWith('PT')
    ) {
        const parsed = parseISODuration(raw.toString());
        if (parsed) {
            const end = dv.date(startDate).plus(parsed).toISODate();
            return { end: adjustIfEqual(end) };
        }
    }

    if (raw?.toISODate) {
        const end = raw.toISODate();
        return { end: adjustIfEqual(end) };
    }

    if (typeof raw === 'string' && /^[0-9]+[dwmy]$/.test(raw.trim())) {
        const parsed = parseDuration(raw.trim());
        const end = dv.date(startDate).plus(parsed).toISODate();
        return { end: adjustIfEqual(end) };
    }

    return { end: dv.date(startDate).plus({ days: 1 }).toISODate() };
}

// Build chart data
let mermaidSections = [];
let debugRows = [];

for (let group of pages.groupBy(p => p.timeline_section ?? 'Allgemein')) {
    const sortedTasks = group.rows
        .map(p => {
            const startDate = resolveStartDate(p);
            const { end } = resolveEndInfo(p);
            const raw = p.timeline_end;
            const type = typeof raw;
            let interpretation = '';

            if (raw?.toISODate) {
                interpretation = 'Date object';
            } else if (raw?.toString?.().startsWith?.('P')) {
                interpretation = 'ISO duration object: ' + raw.toString();
            } else if (typeof raw === 'string' && /^[0-9]+[dwmy]$/.test(raw.trim())) {
                interpretation = 'Duration string';
            } else if (raw?.path || (typeof raw === 'string' && raw.startsWith('[['))) {
                interpretation = 'Link to another note';
            } else {
                interpretation = 'Unrecognized';
            }

            debugRows.push({
                section: p.timeline_section ?? '',
                name: p.file.name,
                ID: p.timeline_ID ?? '',
                status: p.timeline_status ?? '',
                start: startDate,
                end: end,
                end_type: type,
                end_raw: raw,
                interpreted_as: interpretation
            });

            return `${p.file.name} : ${p.timeline_status ?? 'done'}, ${p.timeline_ID ?? 'id_' + p.file.name}, ${startDate}, ${end}`;
        })
        .filter(Boolean);

    if (sortedTasks.length > 0) {
        mermaidSections.push(`section ${group.key}\n${sortedTasks.join("\n")}`);
    }
}

// Debug table for inspecting timeline resolution
/*
if (debugRows.length > 0) {
    dv.table(
        ["Name", "ID", "Status", "Start", "End", "Section", "Type of end", "Raw end", "Interpreted as"],
        debugRows.map(r => [
            r.name,
            r.ID,
            r.status,
            r.start,
            r.end,
            r.section,
            r.end_type,
            (r.end_raw && typeof r.end_raw === 'object' && typeof r.end_raw.toString === 'function') ? r.end_raw.toString() : String(r.end_raw),
            r.interpreted_as
        ])
    );
}
*/

// Render final Mermaid diagram
const mermaidBlock = "```mermaid\n" + mermaidConf + "\n" + mermaidSections.join("\n") + "\n```";
dv.paragraph(mermaidBlock);

```
