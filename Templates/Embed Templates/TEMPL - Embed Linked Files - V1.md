<%*
/**
 * Export ALL scenes referenced via:
 *   scene:: [[Note A]] [[Note B]]
 * from the CURRENT note into the note specified in:
 *   export: [[My Export Note]]
 *
 * No dedupe (repeats kept). No per-scene headers. No cleaning.
 */

tR = "";

// 1) Gather scene:: wikilinks from current note (order + repeats)
const ed = app.workspace.activeLeaf?.view?.editor;
if (!ed) { new Notice("No editor"); return; }
const src = ed.getValue();

const sceneLines = src.split(/\r?\n/).filter(l => /(^|\s)scene\s*::/i.test(l));
if (!sceneLines.length) { new Notice("No scene:: lines"); return; }

const rawLinks = [];
for (const line of sceneLines) {
  for (const m of line.matchAll(/(^|[^!])\[\[([^[\]]+)\]\]/g)) {
    rawLinks.push(m[2]); // keep order & repeats
  }
}
if (!rawLinks.length) { new Notice("No wikilinks on scene:: lines"); return; }

// 2) Resolve to Markdown files (keep repeats; skip non-md / unresolved)
const files = [];
for (let inner of rawLinks) {
  const p = inner.indexOf("|"); if (p>=0) inner = inner.slice(0,p);
  const h = inner.indexOf("#"); if (h>=0) inner = inner.slice(0,h);
  const c = inner.indexOf("^"); if (c>=0) inner = inner.slice(0,c);
  inner = inner.trim();
  if (!inner) continue;

  const tf = app.metadataCache.getFirstLinkpathDest(inner, tp.file.path);
  if (!tf || tf.extension !== "md") continue;
  files.push(tf); // no dedupe!
}
if (!files.length) { new Notice("No resolvable scene notes"); return; }

// 3) Build compiled output (no headers, no cleaning)
const now = tp.date.now("YYYY-MM-DD HH:mm");
let compiled = ""; //`# Export from [[${tp.file.title}]] (scene:: repeats kept)\n\n_Generated: ${now}_\n\n`;

for (const f of files) {
  try {
    const raw = await app.vault.read(f);
    compiled += `${raw}\n\n`;  // <-- no "## filename" header
  } catch {
    compiled += `> ⚠️ Could not read [[${f.basename}]] (${f.path})\n\n`;
  }
}

// 4) Resolve export target from frontmatter and write
let exp = tp.frontmatter?.export;
if (Array.isArray(exp)) exp = exp[0];
if (exp == null) { new Notice("No 'export' frontmatter set"); return; }

let target = (typeof exp === "string") ? exp : String(exp);
const mExp = /^\s*\[\[([\s\S]*?)\]\]\s*$/.exec(target); if (mExp) target = mExp[1];
const p2 = target.indexOf("|"); if (p2>=0) target = target.slice(0,p2);
const h2 = target.indexOf("#"); if (h2>=0) target = target.slice(0,h2);
const c2 = target.indexOf("^"); if (c2>=0) target = target.slice(0,c2);
target = target.trim();

const exportFile = app.metadataCache.getFirstLinkpathDest(target, tp.file.path);
if (!exportFile || exportFile.extension !== "md") { new Notice("'export' not resolvable to .md"); return; }

await app.vault.modify(exportFile, compiled);
new Notice(`✅ Exported ${files.length} scene occurrence${files.length===1?"":"s"} to '${exportFile.basename}' (no headers)`);
%>
