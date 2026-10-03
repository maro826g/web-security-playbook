# Web Security Playbook

My notes and lab walkthroughs from the [PortSwigger Web Security Academy](https://portswigger.net/web-security), written in my own words. Each note explains how a vulnerability class works, how I exploited it in the labs (requests, payloads, dead ends), and what I would try beyond the lab on a real target.

> Educational use only. Everything here was done on PortSwigger's intentionally vulnerable lab environments. This repo is not affiliated with PortSwigger.

## Contents

| Topic | Note |
|---|---|
| Cross-Site Scripting & CSP (including CSP bypass and dangling markup) | [xss.md](vulnerabilities/xss.md) |
| Cross-Site Request Forgery | [csrf.md](vulnerabilities/csrf.md) |
| CORS misconfigurations | [cors.md](vulnerabilities/cors.md) |
| Server-Side Request Forgery | [ssrf.md](vulnerabilities/ssrf.md) |
| Server-Side Template Injection | [ssti.md](vulnerabilities/ssti.md) |
| OS Command Injection | [command-injection.md](vulnerabilities/command-injection.md) |
| Access Control | [access-control.md](vulnerabilities/access-control.md) |
| Business Logic | [business-logic.md](vulnerabilities/business-logic.md) |
| Race Conditions (including Turbo Intruder scripting) | [race-conditions.md](vulnerabilities/race-conditions.md) |
| JWT attacks | [jwt.md](vulnerabilities/jwt.md) |
| File Upload | [file-upload.md](vulnerabilities/file-upload.md) |
| Information Disclosure | [information-disclosure.md](vulnerabilities/information-disclosure.md) |
| API Testing | [api-testing.md](vulnerabilities/api-testing.md) |
| WebSockets | [websockets.md](vulnerabilities/websockets.md) |
| Web LLM attacks | [web-llm-attacks.md](vulnerabilities/web-llm-attacks.md) |


## Related work

- [DVHRA](https://github.com/maro826g/damn-vuln-app): a deliberately vulnerable Spring Boot HR portal I built (SQLi, reflected XSS, SSRF, IDOR, broken access control), with a patched `damn-secured-app` version.
- [DVHRA write-up on Medium](https://medium.com/@amro.khaledo826/under-the-hood-of-dvhra-root-cause-analysis-and-mitigations-in-modern-web-apps-9787a081a61e): root causes and mitigations for each flaw.

## Author

Amr Khaled, application security and CTF (team G4mra).
