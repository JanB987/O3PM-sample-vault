<%*
/**
 * Global cleaner: remove ALL level-1 headings from the export note.
 * Removes both forms:
 *   - ATX H1:    "# Heading"          -> (line removed)
 *   - Setext H1: "Heading\n======"    -> (both lines removed)
 *
 * Uses current note's frontmatter:
 *   export: [[Your Export Note]]
 */

tR = "";

// --- resolve export note ---
let exp = tp.frontmatter?.export; if (Array.isArray(exp)) exp = exp[0];
if (exp == null) { new Notice("⚠️ No 'export' frontmatter set"); return; }

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

// --- 1) Remove ATX H1 lines (# Heading) ---
const rxAtxH1Line = /^(?: {0,3})#(?!#)\s+.*\n?/gm;
const atxCount = (text.match(rxAtxH1Line) || []).length;
text = text.replace(rxAtxH1Line, "");

// --- 2) Remove Setext H1 blocks (Heading + underline ====) ---
const rxSetextH1 = /(^[^\n]+)\n(?: {0,3})=+\s*(?=\n|$)/gm;
const setextCount = (text.match(rxSetextH1) || []).length;
text = text.replace(rxSetextH1, "");

// --- tidy a bit ---
text = text.replace(/\n{3,}/g, "\n\n");

// --- write back ---
await app.vault.modify(exportFile, text);

const total = atxCount + setextCount;
new Notice(`🗑 Removed ${total} H1 header${total===1?"":"s"} in '${exportFile.basename}' (ATX: ${atxCount}, Setext: ${setextCount}).`);
%>
