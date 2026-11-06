# Modern Web Stacks

**Section:** 05 — Web Application Security Fundamentals  
**Room:** 03  
**Documentation:** In progress; introduction and Task 2 practical notes recorded.

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

The table records the room-provided setup. The Task 2 transcript below documents requests to port `3000`; it does not independently verify the other services or software versions.

## Prerequisites

- [HTTP in Detail](https://tryhackme.com/room/httpindetail)
- [Linux Shells](https://tryhackme.com/room/linuxshells)
- [Networking Essentials](https://tryhackme.com/room/networkingessentials)

## Important Note

A version banner such as `Server: Apache/2.4.49` is a useful clue, but does not by itself prove exploitability. Headers can be modified or hidden; corroborate the fingerprint and verify the vulnerability's required conditions. See [OWASP's web server fingerprinting guidance](https://wstg.owasp.org/v4.2/4-Web_Application_Security_Testing/01-Information_Gathering/02-Fingerprint_Web_Server/).

## Task 2 — MERN Stack

These notes summarize my own Kali terminal session against the authorized lab service on port `3000`. Quiz-answer identifiers, session values, protected response content, and the exploit request are excluded.

### Stack Components and Fingerprinting

MERN combines MongoDB for data storage, Express.js for the web backend, React for the frontend, and Node.js as the JavaScript runtime. This is the stack identified by the room; my HTTP requests do not independently verify every component or its version.

I inspected response metadata, an unknown route, and the homepage. These provide clues about the exposed application, while the presence of a frontend root element alone does not establish which frontend framework is running.

### Commands and Options

The following are selected commands I ran. The lab IP is replaced with `<LAB_IP>`, and the HTTP scheme is written explicitly.

```bash
curl -I "http://<LAB_IP>:3000/"
curl "http://<LAB_IP>:3000/nonexistent"
curl -c cookies.txt "http://<LAB_IP>:3000/"
```

The transcript also includes cookie replay and JSON POST requests. The relevant options are:

| Option or argument | Purpose |
| --- | --- |
| `-I` | Sends an HTTP HEAD request to retrieve headers without a response body. |
| `-c cookies.txt` | Saves received cookies to a cookie-jar file at the end of the operation. |
| `-b cookies.txt` | Reads that file and sends cookies matching the requested URL. |
| `-X POST` | Explicitly selects POST. It is redundant in these requests because `-d` already selects POST. |
| `-H "Content-Type: application/json"` | Declares that the request body contains JSON. |
| `-d` | Supplies the request body; here it was used to send JSON data. |

Cookie persistence lets separate requests reuse session state. Possessing a session cookie does not itself grant privileged access. See the [curl manual](https://curl.se/docs/manpage.html) for option behavior.

### Practical Observations

| Check | Observed result |
| --- | --- |
| Header inspection | The root HEAD request returned `HTTP/1.1 200 OK` and an HTML content type. Answer-bearing metadata is omitted. |
| Unknown route | The response was an HTML document containing `Cannot GET /nonexistent` inside a `<pre>` element. |
| Update route using GET | The response reported that the GET request was not handled. A later POST to the same path was acknowledged. |
| Session capture | The root request with `-c cookies.txt` returned the application's HTML page; later requests used `-b cookies.txt`. |
| Protected resource before testing | Access was denied both without cookie replay and when replaying the saved session. |
| Ordinary JSON update | The server returned `{"status":"updated"}`. I did not separately retrieve the saved profile fields. |
| Lab vulnerability test | The update was acknowledged, and a subsequent request returned previously protected content. The payload and protected response are excluded. |

Only the initial HEAD request displayed an HTTP status line. The remaining observations describe response bodies; their HTTP status codes were not captured in this transcript.

### Interpretation and Limitations

- **Method matters:** a route is matched using both its path and HTTP method. The unsuccessful GET did not establish that the update endpoint was absent. See [Express routing](https://expressjs.com/en/guide/routing/).
- **Error format matters:** the unhandled-route message appeared within HTML, rather than as a standalone plain-text response. Its wording is a fingerprinting clue, not unique proof of a framework.
- **An acknowledgement has limits:** an update response does not independently prove which fields were stored or that every submitted key is accepted.
- **Observed impact and root cause are different evidence:** the transcript shows a change from denied access to protected content. The room's code example attributes this to unsafe object merging and inherited-property lookup; these requests did not inspect the running JavaScript process directly.

### Prototype Pollution and Key Takeaways

Prototype pollution occurs when untrusted data changes properties on a JavaScript prototype. Code that reads a missing own property may then obtain an inherited value, which can affect an authorization decision. An own property takes precedence over an inherited one, and objects with a null prototype do not inherit from `Object.prototype`.

The main defensive lessons are to validate permitted input fields, use safe merging, and keep authorization state under trusted server-side control. Own-property checks can prevent inherited values being mistaken for explicit state, but do not make user-controlled privilege fields safe. See [MDN's prototype pollution guidance](https://developer.mozilla.org/en-US/docs/Web/Security/Attacks/Prototype_pollution).

[Back to the section overview](./README.md)
