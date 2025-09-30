<%*
tR = "";
// resolve export note
let exp = tp.frontmatter?.export; if (Array.isArray(exp)) exp = exp[0];
if (exp == null) { new Notice("⚠️ No 'export' frontmatter set"); return; }
let target = String(exp);
const m = /^\s*\[\[([\s\S]*?)\]\]\s*$/.exec(target); if (m) target = m[1];
for (const [re,cut] of [[/\|/,'|'], [/#/,'#'], [/ \^/,'^']]) { const i = target.indexOf(cut); if (i>=0) target = target.slice(0,i); }
target = target.trim();
const exportFile = app.metadataCache.getFirstLinkpathDest(target, tp.file.path);
if (!exportFile || exportFile.extension !== "md") { new Notice("⚠️ 'export' not resolvable to .md"); return; }

let text = await app.vault.read(exportFile);

// strip YAML-like blocks: --- … --- containing at least one "key:"
const lines = text.split(/\r?\n/);
let out = [], removed = 0;
for (let i=0;i<lines.length;i++){
  if (/^\s*---\s*$/.test(lines[i])) {
    let j=i+1, hasKey=false;
    while (j<lines.length && !/^\s*---\s*$/.test(lines[j])) {
      if (/^[A-Za-z0-9_.-]+\s*:\s*/.test(lines[j])) hasKey=true;
      j++;
    }
    if (j<lines.length && /^\s*---\s*$/.test(lines[j]) && hasKey) {
      removed++; i=j; if (i+1<lines.length && /^\s*$/.test(lines[i+1])) i++; continue;
    }
  }
  out.push(lines[i]);
}
const cleaned = out.join("\n");
if (cleaned !== text) await app.vault.modify(exportFile, cleaned);
new Notice(`🧹 Removed ${removed} YAML block${removed===1?"":"s"} from '${exportFile.basename}'.`);
%>