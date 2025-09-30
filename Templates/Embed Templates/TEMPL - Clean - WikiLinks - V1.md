<%*
/**
 * Global cleaner: replace ALL plain wikilinks [[...]] in the export note
 * with readable text.
 *
 * Rules:
 *  - [[Target|Alias]] -> "Alias"
 *  - [[Note#Heading]] -> "Heading"
 *  - [[Folder/Note]]  -> "Note"
 *  - Skips embeds: ![[...]] are left as-is
 *
 * Use after your embed step in the master chain.
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

// --- read export text ---
let src = await app.vault.read(exportFile);

// --- replace all plain [[...]] with display text (alias > heading > basename) ---
const re = /(^|[^!])\[\[([^[\]]+)\]\]/g;
let out = "";
let last = 0;
let count = 0;

const displayFromInner = (inner) => {
  // Alias?
  const pipe = inner.indexOf("|");
  const alias = pipe >= 0 ? inner.slice(pipe + 1).trim() : null;
  let target = pipe >= 0 ? inner.slice(0, pipe) : inner;

  // Prefer alias if present
  if (alias && alias.length) return alias;

  // If heading present, prefer heading part
  const hash = target.indexOf("#");
  if (hash >= 0) {
    let heading = target.slice(hash + 1);
    const caretInHeading = heading.indexOf("^");
    if (caretInHeading >= 0) heading = heading.slice(0, caretInHeading);
    heading = heading.trim();
    if (heading) return heading;
    target = target.slice(0, hash);
  }

  // Drop block ref if any
  const caret = target.indexOf("^");
  if (caret >= 0) target = target.slice(0, caret);

  // Use basename (after last '/')
  target = target.trim();
  const parts = target.split("/");
  return (parts.pop() || target).trim();
};

for (let m; (m = re.exec(src)); ) {
  const pre = m[1] ?? "";
  const inner = m[2];

  // text before match
  out += src.slice(last, m.index) + pre;

  const display = displayFromInner(inner);
  out += display;

  last = m.index + pre.length + ("[[" + inner + "]]").length;
  count++;
}

out += src.slice(last);

// --- write back if changed ---
if (count > 0 && out !== src) {
  await app.vault.modify(exportFile, out);
}
new Notice(`🔗 Stripped ${count} wikilink${count===1?"":"s"} in '${exportFile.basename}'.`);
%>