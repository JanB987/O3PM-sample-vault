<%*
/**
 * Remove ALL comments from the export note:
 *   - Obsidian comments: %% ... %%
 *   - HTML comments: <!-- ... -->
 *
 * The export note is taken from current note's frontmatter:
 *   export: [[Your Export Note]]
 */

tR = "";

// Resolve export note
let exp = tp.frontmatter?.export; if (Array.isArray(exp)) exp = exp[0];
if (exp == null) { new Notice("⚠️ No 'export' frontmatter set"); return; }

let target = String(exp);
const m = /^\s*\[\[([\s\S]*?)\]\]\s*$/.exec(target); if (m) target = m[1];
const p = target.indexOf("|"); if (p>=0) target = target.slice(0,p);
const h = target.indexOf("#"); if (h>=0) target = target.slice(0,h);
const c = target.indexOf("^"); if (c>=0) target = target.slice(0,c);
target = target.trim();

const exportFile = app.metadataCache.getFirstLinkpathDest(target, tp.file.path);
if (!exportFile || exportFile.extension !== "md") { new Notice("⚠️ 'export' not resolvable to .md"); return; }

// Read
let text = await app.vault.read(exportFile);

// Count matches before
const rxObsidian = /%%[\s\S]*?%%/g;
const rxHtml     = /<!--[\s\S]*?-->/g;
const countOb = (text.match(rxObsidian) || []).length;
const countHtml = (text.match(rxHtml) || []).length;

// Strip
text = text.replace(rxObsidian, "").replace(rxHtml, "");

// Write back
await app.vault.modify(exportFile, text);

// Report
new Notice(`🧼 Removed ${countOb} Obsidian and ${countHtml} HTML comment block${(countOb+countHtml===1)?"":"s"} from '${exportFile.basename}'.`);
%>