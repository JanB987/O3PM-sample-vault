<%*
/**
 * Global cleaner: remove ALL task lines from the export note.
 * Matches list items with a checkbox:
 *   - [ ], - [x], * [/], + [-], etc.  (leading spaces allowed)
 *
 * Uses current note's frontmatter:
 *   export: [[Your Export Note]]
 */

tR = "";

// --- resolve export note ---
let exp = tp.frontmatter?.export; if (Array.isArray(exp)) exp = exp[0];
if (!exp) { new Notice("⚠️ No 'export' frontmatter set"); return; }

let target = String(exp);
const mExp = /^\s*\[\[([\s\S]*?)\]\]\s*$/.exec(target); if (mExp) target = mExp[1];
const p = target.indexOf("|"); if (p>=0) target = target.slice(0,p);
const h = target.indexOf("#"); if (h>=0) target = target.slice(0,h);
const c = target.indexOf("^"); if (c>=0) target = target.slice(0,c);
target = target.trim();

const exportFile = app.metadataCache.getFirstLinkpathDest(target, tp.file.path);
if (!exportFile || exportFile.extension !== "md") { new Notice("⚠️ 'export' not resolvable to .md"); return; }

// --- read export content ---
let text = await app.vault.read(exportFile);

// --- remove entire task lines ---
// Matches (start of line) optional spaces, -,*,+ list marker, spaces, [any single char] inside [], then rest of line
const rxTaskLine = /^(?:[ \t]{0,3})[-*+][ \t]+\[[^\]]\].*(?:\n|$)/gm;
const removed = (text.match(rxTaskLine) || []).length;
text = text.replace(rxTaskLine, "");

// tidy: collapse 3+ blank lines to 2
text = text.replace(/\n{3,}/g, "\n\n");

// --- write back ---
await app.vault.modify(exportFile, text);
new Notice(`🧹 Removed ${removed} task line${removed===1?"":"s"} in '${exportFile.basename}'.`);
%>