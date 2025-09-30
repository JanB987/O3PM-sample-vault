<%*
/** Remove ALL level-5 headings (##### ...) from the export note. */
tR = "";
let exp = tp.frontmatter?.export; if (Array.isArray(exp)) exp = exp[0];
if (!exp) { new Notice("⚠️ No 'export' frontmatter set"); return; }
let target = String(exp); { const m=/^\s*\[\[([\s\S]*?)\]\]\s*$/.exec(target); if(m) target=m[1]; }
{ const p=target.indexOf("|"); if(p>=0) target=target.slice(0,p);
  const h=target.indexOf("#"); if(h>=0) target=target.slice(0,h);
  const c=target.indexOf("^"); if(c>=0) target=target.slice(0,c); target=target.trim();
}
const exportFile = app.metadataCache.getFirstLinkpathDest(target, tp.file.path);
if (!exportFile || exportFile.extension !== "md") { new Notice("⚠️ 'export' not resolvable to .md"); return; }
let text = await app.vault.read(exportFile);

// ATX H5 (exactly five #'s)
const rxAtx = /^(?: {0,3})#####[ \t]+.*\n?/gm;   // note the space/tab after hashes
const atxCount = (text.match(rxAtx) || []).length;
text = text.replace(rxAtx, "");

// tidy & write
text = text.replace(/\n{3,}/g, "\n\n");
await app.vault.modify(exportFile, text);
new Notice(`🗑 Removed ${atxCount} H5 header${atxCount===1?"":"s"} in '${exportFile.basename}'.`);
%>
