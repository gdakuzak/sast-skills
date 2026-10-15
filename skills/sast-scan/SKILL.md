---
name: sast-scan
description: >-
  Run a full SAST security assessment on the project in the current working
  directory: codebase analysis, all vulnerability checks in parallel subagents,
  and a consolidated report in sast/final-report.md. Resumes from existing
  sast/ files. Use when asked to "run vulnerability scan", "find vulnerabilities",
  "security assessment" or "/sast:sast-scan".
---

# SAST Security Assessment

Your goal is to identify security vulnerabilities in the codebase located in the current working directory (the project the user is in). All output goes to `sast/` at that project root.

Skills from this plugin are namespaced: invoke them as `sast:<skill>` (e.g. `sast:sast-sqli`). Tell each subagent the full namespaced name.

---

## Step 1: Codebase Analysis & Threat Modeling

Before running, check if `sast/architecture.md` already exists. If it does, skip this step.

Run the `sast:sast-analysis` skill directly (this one stays in-session since later steps depend on reading its output).

**Wait for this step to finish before proceeding.**

---

## Step 2: Vulnerability Detection (Parallel)

Run all checks at the same time. Skip any task where the output file already exists.

- Skip IDOR if `sast/idor-results.md` already exists.
- Skip SQLi if `sast/sqli-results.md` already exists.
- Skip SSRF if `sast/ssrf-results.md` already exists.
- Skip XSS if `sast/xss-results.md` already exists.
- Skip RCE if `sast/rce-results.md` already exists.
- Skip XXE if `sast/xxe-results.md` already exists.
- Skip File Upload if `sast/fileupload-results.md` already exists.
- Skip Path Traversal if `sast/pathtraversal-results.md` already exists.
- Skip SSTI if `sast/ssti-results.md` already exists.
- Skip JWT if `sast/jwt-results.md` already exists.
- Skip Missing Auth if `sast/missingauth-results.md` already exists.
- Skip Business Logic if `sast/businesslogic-results.md` already exists.
- Skip GraphQL injection if `sast/graphql-results.md` already exists.
- Skip Hardcoded Secrets if `sast/hardcodedsecrets-results.md` already exists.

Start **one subagent per check**, all **in parallel**, each with a dedicated task. Give each subagent the same instruction pattern, using the skill name and paths from the table:

> Read `sast/architecture.md` for context, then run the named SAST skill. Write all findings to that skill's results file. Clean up any intermediate recon or threat files for that skill when done.

| Skill | Results file | Typical intermediate files to clean |
|-------|----------------|--------------------------------------|
| sast-idor | `sast/idor-results.md` | `sast/idor-recon.md` |
| sast-sqli | `sast/sqli-results.md` | `sast/sqli-recon.md`, `sast/sqli-batch-*.md` |
| sast-ssrf | `sast/ssrf-results.md` | `sast/ssrf-recon.md` |
| sast-xss | `sast/xss-results.md` | `sast/xss-recon.md` |
| sast-rce | `sast/rce-results.md` | `sast/rce-recon.md`, `sast/rce-batch-*.md` |
| sast-xxe | `sast/xxe-results.md` | `sast/xxe-recon.md` |
| sast-fileupload | `sast/fileupload-results.md` | `sast/fileupload-recon.md`, `sast/fileupload-batch-*.md` |
| sast-pathtraversal | `sast/pathtraversal-results.md` | `sast/pathtraversal-recon.md`, `sast/pathtraversal-batch-*.md` |
| sast-ssti | `sast/ssti-results.md` | `sast/ssti-recon.md` |
| sast-jwt | `sast/jwt-results.md` | `sast/jwt-recon.md` |
| sast-missingauth | `sast/missingauth-results.md` | `sast/missingauth-recon.md`, `sast/missingauth-batch-*.md` |
| sast-businesslogic | `sast/businesslogic-results.md` | `sast/businesslogic-threats.md`, `sast/businesslogic-batch-*.md` |
| sast-graphql | `sast/graphql-results.md` | `sast/graphql-recon.md` |
| sast-hardcodedsecrets | `sast/hardcodedsecrets-results.md` | `sast/hardcodedsecrets-recon.md`, `sast/hardcodedsecrets-batch-*.md` |

Wait for all subagents to finish before proceeding.

---

## Step 3: Report Generation

After all subagents from Step 2 finish, generate the final consolidated report.

Skip this step if `sast/final-report.md` already exists.

Launch a single subagent:

> Read all available `sast/*-results.md` files and `sast/architecture.md` for context, then run the sast-report skill to generate `sast/final-report.md` with all findings ranked by severity and confidentiality impact.

---

## Step 4: Wrap up

Tell the user where the report is (`sast/final-report.md`) with a short severity count, and suggest adding `sast/` to `.gitignore` if it isn't there.
