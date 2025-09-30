<%*
/**
 * Global cleaner: remove ALL markdown strikethroughs (~~…~~)
 * from the export note, including the text inside.
 *
 * Example:
 *   "This is ~~bad~~ text."  ->  "This is  text."
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

// --- remove strikethroughs completely: ~~text~~ -> "" ---
const rxStrike = /~~[^~\n]+~~/g;
const strikeCount = (text.match(rxStrike) || []).length;
text = text.replace(rxStrike, "");

// --- write back ---
await app.vault.modify(exportFile, text);

new Notice(`✂️ Removed ${strikeCount} strikethrough${strikeCount===1?"":"s"} (including content) in '${exportFile.basename}'.`);
%>
