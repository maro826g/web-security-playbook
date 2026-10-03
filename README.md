# Web Security Playbook

My notes and lab walkthroughs from the [PortSwigger Web Security Academy](https://portswigger.net/web-security), written in my own words. Each note explains how a vulnerability class works, how I exploited it in the labs (requests, payloads, dead ends), and what I would try beyond the lab on a real target.

> Educational use only. Everything here was done on PortSwigger's intentionally vulnerable lab environments. This repo is not affiliated with PortSwigger.

## Contents

| Topic | Note |
|---|---|
| Cross-Site Scripting & CSP (including CSP bypass and dangling markup) | [XSS](vulnerabilities%20/XSS.md) |
| Cross-Site Request Forgery | [CSRF](vulnerabilities%20/CSRF.md) |
| CORS misconfigurations | [CORS](vulnerabilities%20/CORS.md) |
| Server-Side Request Forgery | [SSRF](vulnerabilities%20/SSRF.md) |
| Server-Side Template Injection | [SSTI](vulnerabilities%20/SSTI.md) |
| OS Command Injection | [Command Injection](vulnerabilities%20/Command%20Injection.md) |
| Access Control | [Access Control Vulnerabilities](vulnerabilities%20/Access%20Control%20Vulnerabilities.md) |
| Business Logic | [Business logic vulnerabilities](vulnerabilities%20/Business%20logic%20vulnerabilities.md) |
| Race Conditions (including Turbo Intruder scripting) | [Race Condition](vulnerabilities%20/Race%20Condition.md) |
| JWT attacks | [JWT's Vulnerabilities](vulnerabilities%20/JWT%27s%20Vulnerabilities.md) |
| File Upload | [File Upload Vulnerabilities](vulnerabilities%20/File%20Upload%20Vulnerabilities.md) |
| Information Disclosure | [Information Disclosure](vulnerabilities%20/Information%20Disclosure.md) |
| API Testing | [API testing](vulnerabilities%20/API%20testing.md) |
| WebSockets | [Websocket vulnerabilities](vulnerabilities%20/Websocket%20vulnerabilities%20.md) |
| Web LLM attacks | [Web LLM attacks](vulnerabilities%20/Web%20LLM%E2%80%99s%20attacks.md) |

## Roadmap

Not written up yet: SQL injection, authentication, path traversal, XXE, insecure deserialization, OAuth, HTTP request smuggling, host header attacks, web cache poisoning, clickjacking, NoSQL injection, prototype pollution.

## Related work

- [DVHRA](https://github.com/maro826g/damn-vuln-app): a deliberately vulnerable Spring Boot HR portal I built (SQLi, reflected XSS, SSRF, IDOR, broken access control), with a patched `damn-secured-app` version.
- [DVHRA write-up on Medium](https://medium.com/@amro.khaledo826/under-the-hood-of-dvhra-root-cause-analysis-and-mitigations-in-modern-web-apps-9787a081a61e): root causes and mitigations for each flaw.

## Author

Amr Khaled, application security and CTF (team G4mra).

[LinkedIn](https://www.linkedin.com/in/amrk1/)
