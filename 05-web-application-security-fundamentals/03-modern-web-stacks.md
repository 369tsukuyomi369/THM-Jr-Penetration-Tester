# Modern Web Stacks

**Section:** 05 — Web Application Security Fundamentals  
**Room:** 03  
**Documentation:** In progress; introduction recorded and Task 2 outlined.

## Introduction

Web stack fingerprinting uses response headers, cookie names, error messages, URL patterns, and HTML source to identify the technologies behind a web application. These clues help focus version checks and vulnerability research.

## Workflow

1. **Fingerprint:** inspect ordinary HTTP responses before sending exploit payloads.
2. **Verify:** corroborate the technology and version, then check relevant CVE advisories and their applicability.
3. **Validate in the lab:** test the applicable vulnerability, understand its root cause, and record the observed impact.

## Learning Objectives

| Technology | CVE | Focus |
| --- | --- | --- |
| Next.js | [CVE-2025-29927](https://github.com/vercel/next.js/security/advisories/GHSA-f82v-jwr5-mffw) | Bypassing authorization checks implemented in middleware. |
| Django | [CVE-2021-35042](https://www.djangoproject.com/weblog/2021/jul/01/security-releases/) | SQL injection through unsanitized input to `QuerySet.order_by()`, with database extraction as the lab objective. |
| Apache HTTP Server 2.4.49 | [CVE-2021-41773](https://httpd.apache.org/security/vulnerabilities_24.html#CVE-2021-41773) | Path traversal and file disclosure; remote code execution (RCE) depends on access controls and CGI configuration. |

## Lab Environment

The room describes an Ubuntu 22.04 virtual machine with four services:

| Port | Service described in the room |
| --- | --- |
| `3000` | MERN / Express |
| `3001` | Next.js / React Server Components (RSC) |
| `8000` | Django |
| `8080` | Apache HTTP Server 2.4.49 in a Docker container |

Access is through the TryHackMe AttackBox or a machine connected to the lab through the TryHackMe VPN.

These are room-provided setup details. Service availability, versions, and exploitation results have not yet been documented from practical testing.

## Prerequisites

- [HTTP in Detail](https://tryhackme.com/room/httpindetail)
- [Linux Shells](https://tryhackme.com/room/linuxshells)
- [Networking Essentials](https://tryhackme.com/room/networkingessentials)

## Important Note

A version banner such as `Server: Apache/2.4.49` is a useful clue, but does not by itself prove exploitability. Headers can be modified or hidden; corroborate the fingerprint and verify the vulnerability's required conditions. See [OWASP's web server fingerprinting guidance](https://wstg.owasp.org/v4.2/4-Web_Application_Security_Testing/01-Information_Gathering/02-Fingerprint_Web_Server/).

## Task 2 — MERN Stack

*Outline to expand with my own explanations and lab evidence.*

### Stack Components

- Roles of MongoDB, Express.js, React, and Node.js.

### Fingerprinting

- HTTP headers, session cookies, error responses, and frontend clues.
- Corroborating observations and recognizing the limits of individual signals.

### Requests and Sessions

- HTTP methods, JSON request bodies, and cookie persistence with `curl`.
- Comparing ordinary application behavior with access-controlled behavior.

### Prototype Pollution

- JavaScript prototype inheritance and unsafe recursive object merging.
- How inherited properties can affect authorization checks.

### Practical Observations

- Add my own commands, observations, and redacted evidence.
- Distinguish expected behavior from results actually observed.

### Key Takeaways

- Summarize the root cause, limitations, and security lessons.

[Back to the section overview](./README.md)
