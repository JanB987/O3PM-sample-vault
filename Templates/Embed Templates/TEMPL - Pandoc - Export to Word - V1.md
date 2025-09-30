<%*
/**
 * Exports the export note (frontmatter `export: [[...]]`) to DOCX via Pandoc Plugin.
 * Accepts BOTH:
 *   Frontmatter on the calling note:
 *     pandoc_export: "My Output Name"        # without .docx
 *     pandoc_reference: "[[path/to/ref.docx]]"  # wikilink, vault-relative, or absolute
 *   OR template variables (tp.variables):
 *     docx_name, reference_doc
 *
 * It temporarily injects YAML with reference-doc paths and (optionally) renames
 * the export note so the output .docx gets your desired name, then restores.
 */

tR = "";

// ---------- helpers ----------
const sleep = (ms) => new Promise(r => setTimeout(r, ms));
const toInnerWikilink = (s) => {
  if (!s) return null;
  const m = /^\s*\[\[([\s\S]*?)\]\]\s*$/.exec(String(s));
  return m ? m[1] : String(s);
};

// Make a filesystem path for a value that may be:
//  - wikilink to a vault file
//  - vault-relative string
//  - absolute OS path
const toFsPath = (maybePath) => {
  if (!maybePath) return null;
  let p = toInnerWikilink(maybePath).trim();

  // Resolve wikilink into a vault file
  const tf = app.metadataCache.getFirstLinkpathDest(p.replace(/[#|^].*$/,""), tp.file.path);
  if (tf) {
    const base = app.vault.adapter.getBasePath?.();
    if (base) {
      const abs = (base.endsWith("/") || base.endsWith("\\")) ? (base + tf.path) : (base + "/" + tf.path);
      return abs;
    }
  }

  // If already absolute, return as-is
  const isAbsWin = /^[a-zA-Z]:[\\/]/.test(p);
  const isAbsUnix = p.startsWith("/");
  if (isAbsWin || isAbsUnix) return p;

  // Treat as vault-relative
  const base = app.vault.adapter.getBasePath?.();
  if (!base) return null;
  const abs = (base.endsWith("/") || base.endsWith("\\")) ? (base + p) : (base + "/" + p);
  return abs;
};

const toFileUri = (absPath) => absPath ? ("file:///" + absPath.replace(/\\/g, "/")) : null;

// Find a Pandoc Plugin DOCX command
const findPandocDocxCommand = () => {
  const cmds = app.commands.listCommands();
  return cmds.find(c => /pandoc/i.test(c.id) && /docx|word/i.test(c.name))
      || cmds.find(c => /pandoc/i.test(c.name) && /docx|word/i.test(c.name));
};

// ---------- resolve export note ----------
let exp = tp.frontmatter?.export; if (Array.isArray(exp)) exp = exp[0];
if (!exp) { new Notice("⚠️ No 'export' frontmatter set"); return; }

let target = String(exp);
{ const m=/^\s*\[\[([\s\S]*?)\]\]\s*$/.exec(target); if (m) target = m[1]; }
{ const i=target.indexOf("|"); if(i>=0) target=target.slice(0,i);
  const j=target.indexOf("#"); if(j>=0) target=target.slice(0,j);
  const k=target.indexOf("^"); if(k>=0) target=target.slice(0,k);
  target = target.trim();
}
const exportFile = app.metadataCache.getFirstLinkpathDest(target, tp.file.path);
if (!exportFile || exportFile.extension !== "md") { new Notice("⚠️ Export note not found"); return; }

// ---------- read desired output name & reference from variables OR frontmatter ----------
const desiredName =
  (tp.variables?.docx_name && String(tp.variables.docx_name)) ||
  (tp.frontmatter?.pandoc_export && String(tp.frontmatter.pandoc_export)) ||
  "";

const refInput =
  (tp.variables?.reference_doc && String(tp.variables.reference_doc)) ||
  (tp.frontmatter?.pandoc_reference && String(tp.frontmatter.pandoc_reference)) ||
  null;

const refFsPath = toFsPath(refInput);
const refUri = toFileUri(refFsPath);

// ---------- save originals ----------
const originalText = await app.vault.read(exportFile);
const originalPath = exportFile.path;
const originalBasename = exportFile.basename;

// ---------- optional rename to control output .docx name ----------
let renamed = false;
if (desiredName && desiredName !== originalBasename) {
  const newPath = originalPath.replace(/[^/\\]+\.md$/i, desiredName + ".md");
  try {
    await app.fileManager.renameFile(exportFile, newPath);
    renamed = true;
  } catch (e) {
    new Notice("⚠️ Could not rename export note; proceeding with original name.");
  }
}

// ---------- inject temporary YAML with reference-doc (plain path + URI, both keys) ----------
let injected = false;
if (refFsPath) {
  const yamlLines = [
    "---",
    `reference-doc: "${refFsPath.replace(/\\/g,"/")}"`,
    `reference-docx: "${refFsPath.replace(/\\/g,"/")}"`,
    ...(refUri ? [`reference-doc-uri: "${refUri}"`] : []), // diagnostic/alt
    "---",
    ""
  ];
  const injectedYaml = yamlLines.join("\n");
  try {
    await app.vault.modify(exportFile, injectedYaml + originalText);
    injected = true;
  } catch (e) {
    new Notice("⚠️ Could not inject reference-doc YAML; exporting without it.");
  }
} else if (refInput) {
  new Notice("⚠️ Reference DOCX path could not be resolved. Check 'pandoc_reference'.");
}

// ---------- open the export note & run Pandoc DOCX ----------
const leaf = app.workspace.getLeaf(false);
await leaf.openFile(exportFile);

// Ensure the configured output folder exists (Pandoc plugin setting).
// If it doesn't, Pandoc can throw: withBinaryFile: does not exist.
try {
  const settings = app.plugins?.plugins?.["obsidian-pandoc"]?.settings;
  const outDir = settings?.outputFolder || settings?.docx_outputFolder || null;
  if (outDir) {
    const base = app.vault.adapter.getBasePath?.();
    const absOut = (base && !/^[a-zA-Z]:[\\/]|^\//.test(outDir))
      ? ((base.endsWith("/")||base.endsWith("\\")) ? base + outDir : base + "/" + outDir)
      : outDir;
    // Try to create folder if missing (best effort)
    try { await app.vault.adapter.mkdir(absOut); } catch (_) { /* ignore if exists */ }
  }
} catch (_) { /* non-fatal */ }

const cmd = findPandocDocxCommand();
if (!cmd) {
  new Notice("⚠️ Pandoc DOCX command not found. Check the Pandoc Plugin is installed and commands available.");
} else {
  app.commands.executeCommandById(cmd.id);
  new Notice(`📦 Pandoc DOCX export triggered${desiredName ? " → " + desiredName + ".docx" : ""}.`);
}

// Give the plugin a moment to read the file before restore
await sleep(300);

// ---------- restore original content & name ----------
if (injected) {
  try { await app.vault.modify(exportFile, originalText); } catch (e) { /* ignore */ }
}
if (renamed) {
  try { await app.fileManager.renameFile(exportFile, originalPath); } catch (e) { /* ignore */ }
}
%>
