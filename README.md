# Claude Code Source Exposure via Source Map (.map) – Security Report

## Summary

On **March 31, 2026**, a security issue was identified in the npm package:

```
@anthropic-ai/claude-code@2.1.88
```

The package unintentionally included a **source map file (`cli.js.map`)** containing the `sourcesContent` field, which allowed full reconstruction of the original TypeScript source code.

This represents a **software supply chain exposure** caused by improper handling of build artifacts during the release process.

---

## Technical Details

### Affected Package Contents

```
@anthropic-ai/claude-code@2.1.88

cli.js        (13 MB)
cli.js.map    (59.8 MB)  <-- vulnerable artifact
```

The `.map` file contained:

* `sources` → original file paths
* `sourcesContent` → full source code (plaintext)

---

## Reconstruction Process

The exposed source was reconstructed locally using the following script:

```js
import fs from "fs";
import path from "path";

const map = JSON.parse(fs.readFileSync("cli.js.map", "utf-8"));

if (!map.sources || !map.sourcesContent) {
  console.error("No se puede reconstruir");
  process.exit(1);
}

map.sources.forEach((source, i) => {
  let filePath = source.replace(/^(\.\.\/)+/, "");

  const content = map.sourcesContent[i];
  if (!content) return;

  const fullPath = path.join("reconstructed", filePath);

  fs.mkdirSync(path.dirname(fullPath), { recursive: true });
  fs.writeFileSync(fullPath, content);
});

console.log("reconstruido");
```

### Result

* Reconstruction was successful
* Full project structure restored
* Thousands of source files recovered

---

## Reconstructed Architecture Overview

The recovered project includes:

* Command system (`commands/`) → CLI operations (~80+ commands)
* Tool system (`tools/`) → execution layer (Bash, File, Web, MCP, etc.)
* Services layer (`services/`) → API, OAuth, plugins, telemetry
* Agent orchestration (`coordinator/`, `tasks/`)
* Permission system (`components/permissions/`, `utils/permissions/`)
* Bridge system (`bridge/`) → IDE communication
* Plugin architecture (`plugins/`)
* Remote execution capabilities (`remote/`, `RemoteTriggerTool`)

---

## Security Impact

### 1. Intellectual Property Exposure

* Full internal architecture disclosed
* Design patterns, workflows, and internal logic exposed

---

### 2. Attack Surface Expansion

The leak enables attackers to:

* Analyze tool execution flows
* Identify permission enforcement weaknesses
* Study agent coordination logic
* Reverse engineer trust boundaries

---

### 3. High-Risk Components Identified

Potentially sensitive modules include:

* `BashTool` → shell command execution
* `FileWriteTool` / `FileEditTool` → filesystem manipulation
* `RemoteTriggerTool` → remote execution triggers
* `MCPTool` → external service interaction
* `Bridge` → IDE communication layer (possible auth surface)

---

### 4. Permission System Exposure

The internal permission model is fully visible:

* Approval flows
* Bypass modes
* Enforcement logic

This significantly reduces the effort required to:

* Craft prompt injection attacks
* Abuse tool execution pathways

---

## Root Cause

The issue originated from:

* Inclusion of `.map` file in production npm package
* Presence of `sourcesContent` inside the source map
* Public accessibility of original source paths

### Classification

* CWE-530: Exposure of Sensitive Information via Source Code
* CWE-215: Information Exposure Through Debug Information
* Supply Chain Misconfiguration

---

## Recommended Mitigations

### Immediate

* Remove `.map` files from npm distribution
* Revoke public access to any linked storage (R2/S3)

---

### Build Pipeline Fixes

* Disable source map generation in production:

  ```
  sourceMap: false
  ```

* Or use:

  ```
  hidden-source-map
  ```

* Strip `sourcesContent` before publishing

---

### Secure Release Practices

* Audit build artifacts before publishing
* Implement CI/CD validation checks
* Use private storage for debug artifacts
* Apply least-privilege access to storage buckets

---

### Defense in Depth

* Harden permission system (server-side enforcement)
* Reduce trust in client-side logic
* Monitor abnormal tool usage patterns

---

## Responsible Disclosure & Ethics

This report is published strictly for:

* Educational purposes
* Defensive security research
* Software supply chain awareness

No modifications were made to the original software beyond reconstruction from publicly available artifacts.

---

## Disclaimer

This repository does not claim ownership of any original source code.

All rights belong to their respective owners.

This work:

* Does not aim to harm the vendor
* Does not encourage misuse
* Is intended to highlight a security misconfiguration

The author assumes no responsibility for:

* Misuse of the information provided
* Any actions taken by third parties based on this research

---

## Timeline

* 2026-03-31 → Exposure identified via npm package
* 2026-03-31 → Source reconstruction completed

---

## Final Notes

This incident highlights a critical but common failure in modern development:

Build artifacts can be as sensitive as source code itself.

Proper handling of `.map` files is essential, especially in distributed environments like npm.

---

## Author

Security Researcher / Community Contributor

---

## Contribution

If this report is useful, contributions and improvements are welcome.
