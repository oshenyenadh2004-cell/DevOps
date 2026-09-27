# Threat Model — NodeGoat DevSecOps Project

## 1. System Overview
[Reference Akash's architecture diagram: browser, Express app, MongoDB, trust boundaries between them]

NodeGoat runs as a fairly standard three-tier setup — browser on one end, an Express/Node.js server handling requests in the middle, MongoDB storing everything at the back. The main trust boundary that matters here is where user input first hits the server, since nothing coming from the browser can really be trusted. There's a second boundary between the app and the database, which matters because a lot of NodeGoat's known issues come from queries being built directly from user input without checking it first.

## 2. STRIDE Analysis

| Threat ID | Description | STRIDE Category | Component/Data Flow |
|-----------|--------------|------------------|----------------------|
| T1 | Login and search fields pass raw user input into MongoDB queries, so an attacker can manipulate the query itself instead of just the data (classic NoSQL injection in this app) | Tampering | Browser → Express → MongoDB |
| T2 | The Memos feature lets users store free text that gets shown to others later without being cleaned up first — a script planted there runs in whoever views it | Tampering / Info Disclosure | Browser → Express → MongoDB → Browser |
| T3 | Because state-changing requests don't check where they came from, a victim could be tricked into submitting a request just by visiting a malicious page while logged in | Spoofing | Browser → Express (profile/account endpoints) |
| T4 | Some config values and keys are sitting directly in the source rather than being pulled from environment variables, so anyone who gets repo access gets the credentials too | Info Disclosure | Source code / config files |

## 3. Risk Assessment

| Threat ID | Likelihood | Impact | Score | Justification |
|-----------|------------|--------|-------|----------------|
| T1 | 3 | 3 | 9 | This is one of the most commonly demonstrated NodeGoat flaws, so it's easy to trigger, and it can lead straight to bypassing login |
| T2 | 2 | 2 | 4 | Needs a bit of setup (attacker posts the payload, someone else has to open it), so it's a step slower to pull off, but it can still leak session data |
| T3 | 2 | 2 | 4 | Depends on getting a logged-in user to click something external, so it's not trivial, but the app currently gives no protection against it either |
| T4 | 2 | 3 | 6 | Not something an outside attacker can trigger directly, but if the repo or a screenshot leaks, the damage is immediate and hard to undo |

## 4. Threat-to-Control Mapping

| Threat ID | Mitigating Control | Where It Lives |
|-----------|----------------------|-------------------|
| T1 | Semgrep rules catching unsanitized query construction, plus basic input validation in the route handlers | sast-semgrep gate |
| T2 | Output should be escaped before being rendered back to users; semgrep can also catch obvious unescaped-output patterns | sast-semgrep gate |
| T3 | Adding CSRF middleware and checking for it as part of dependency review | dependency-audit gate |
| T4 | Gitleaks blocks any commit that contains something that looks like a key or credential | secrets-scan gate, config in .gitleaks.toml |