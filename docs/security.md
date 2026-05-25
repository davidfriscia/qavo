---
layout: default
title: Security
nav_order: 4
---

# Security Policy

## Supported versions

Security fixes are applied to the latest released MINOR version of the current MAJOR line. When a new MAJOR is released, the previous MAJOR receives security fixes for a defined transition period (stated in the release notes).

| Version | Supported |
|---|---|
| Latest MINOR of current MAJOR | ✅ Yes |
| Previous MAJOR (transition period) | ✅ Yes (see release notes) |
| Older versions | ❌ No |

---

## Reporting a vulnerability

**Do not open a public GitHub issue for security vulnerabilities.**

To report a vulnerability, please use one of the following channels:

- **GitHub private vulnerability reporting:** use the "Report a vulnerability" button in the Security tab of the relevant repository (`qavo-be` or `qavo-fe`). This is the preferred channel.
- **Email:** send a description to the maintainer at the email address in the repository's `pom.xml` / `package.json`. Encrypt your message with the maintainer's public GPG key if the content is sensitive.

Please include:

- A description of the vulnerability and its potential impact.
- Steps to reproduce or a proof-of-concept (if possible).
- The affected version(s) and module(s).
- Any suggested mitigation or fix, if you have one.

---

## Response process

1. **Acknowledgement** — you will receive acknowledgement within 72 hours.
2. **Assessment** — the maintainer will assess severity and scope, and keep you informed of progress.
3. **Fix and release** — a fix will be developed, tested, and released as a PATCH. The release notes will credit the reporter (unless you prefer to remain anonymous).
4. **Disclosure** — after the fix is released, the vulnerability may be publicly disclosed with appropriate detail. The timeline is agreed with the reporter.

---

## Scope

This policy covers the Qavo platform modules (`qavo-be`, `qavo-fe`) and the documentation in this repository. It does not cover applications built on top of Qavo — those are the responsibility of their respective maintainers.

---

## Note on AI-assisted development

Qavo is built with AI assistance. The maintainers are aware that AI-generated code can contain subtle security issues and apply extra diligence in review, especially in security-sensitive areas (authentication, authorization, cryptography, input validation). If you find a vulnerability that appears to stem from AI-generated code, please report it through this process like any other — the origin of the code does not affect the seriousness with which it is treated.
