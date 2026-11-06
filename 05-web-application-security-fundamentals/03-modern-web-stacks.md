# Modern Web Stacks

**Section:** 05 — Web Application Security Fundamentals  
**Room:** 03  
**Documentation:** In progress; Task 1 introduction and Tasks 2–3 practical notes with selected screenshots recorded.

## Task 1 — Introduction

Web stack fingerprinting uses response headers, cookie names, error messages, URL patterns, and HTML source to identify the technologies behind a web application. These clues help focus version checks and vulnerability research.

### Workflow

1. **Fingerprint:** inspect ordinary HTTP responses before sending exploit payloads.
2. **Verify:** corroborate the technology and version, then check relevant CVE advisories and their applicability.
3. **Validate in the lab:** test the applicable vulnerability, understand its root cause, and record the observed impact.

### Learning Objectives

| Technology | CVE | Focus |
| --- | --- | --- |
| Next.js | [CVE-2025-29927](https://github.com/vercel/next.js/security/advisories/GHSA-f82v-jwr5-mffw) | Bypassing authorization checks implemented in middleware. |
| Django | [CVE-2021-35042](https://www.djangoproject.com/weblog/2021/jul/01/security-releases/) | SQL injection through unsanitized input to `QuerySet.order_by()`, with database extraction as the lab objective. |
| Apache HTTP Server 2.4.49 | [CVE-2021-41773](https://httpd.apache.org/security/vulnerabilities_24.html#CVE-2021-41773) | Path traversal and file disclosure; remote code execution (RCE) depends on access controls and CGI configuration. |

### Lab Environment

The room describes an Ubuntu 22.04 virtual machine with four services:

| Port | Service described in the room |
| --- | --- |
| `3000` | MERN / Express |
| `3001` | Next.js / React Server Components (RSC) |
| `8000` | Django |
| `8080` | Apache HTTP Server 2.4.49 in a Docker container |

Access is through the TryHackMe AttackBox or a machine connected to the lab through the TryHackMe VPN.

The table records the room-provided setup. Task 2 documents requests to port `3000`, and Task 3 records Next.js-related responses on port `3001`. The remaining services and exact software versions have not been independently verified.

### Prerequisites

- [HTTP in Detail](https://tryhackme.com/room/httpindetail)
- [Linux Shells](https://tryhackme.com/room/linuxshells)
- [Networking Essentials](https://tryhackme.com/room/networkingessentials)

### Important Note

A version banner such as `Server: Apache/2.4.49` is a useful clue, but does not by itself prove exploitability. Headers can be modified or hidden; corroborate the fingerprint and verify the vulnerability's required conditions. See [OWASP's web server fingerprinting guidance](https://wstg.owasp.org/v4.2/4-Web_Application_Security_Testing/01-Information_Gathering/02-Fingerprint_Web_Server/).

## Task 2 — MERN Stack

These notes summarize my own Kali terminal session against the authorized lab service on port `3000`. Room answers, session values, and protected response values are excluded.

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
| Lab vulnerability test | The update was acknowledged, and a subsequent request returned previously protected content. The protected response value is excluded. |

Only the initial HEAD request displayed an HTTP status line. The remaining observations describe response bodies; their HTTP status codes were not captured in this transcript.

### Screenshots

Selected evidence from the same terminal session.

**Route handling and the initial access check**

![Unmatched GET requests and denied access before the lab test](./assets/modern-web-stacks/mern-route-and-access-checks.png)

**Homepage request using the cookie-jar option**

![Request using curl's cookie-jar option and the returned homepage](./assets/modern-web-stacks/mern-session-cookie-jar.png)

**Ordinary profile update and acknowledgement**

![JSON profile-update request and the server's acknowledgement](./assets/modern-web-stacks/mern-normal-profile-update.png)

**Prototype pollution test and acknowledgement**

![Prototype pollution request from my lab session and the server's update acknowledgement](./assets/modern-web-stacks/mern-prototype-pollution-request.png)

**Protected response with the answer redacted**

![Protected response with the answer obscured in the supplied screenshot](./assets/modern-web-stacks/mern-protected-response-redacted.png)

### Interpretation and Limitations

- **Method matters:** a route is matched using both its path and HTTP method. The unsuccessful GET did not establish that the update endpoint was absent. See [Express routing](https://expressjs.com/en/guide/routing/).
- **Error format matters:** the unhandled-route message appeared within HTML, rather than as a standalone plain-text response. Its wording is a fingerprinting clue, not unique proof of a framework.
- **An acknowledgement has limits:** an update response does not independently prove which fields were stored or that every submitted key is accepted.
- **Observed impact and root cause are different evidence:** the transcript shows a change from denied access to protected content. The room's code example attributes this to unsafe object merging and inherited-property lookup; these requests did not inspect the running JavaScript process directly.

### Prototype Pollution and Key Takeaways

Prototype pollution occurs when untrusted data changes properties on a JavaScript prototype. Code that reads a missing own property may then obtain an inherited value, which can affect an authorization decision. An own property takes precedence over an inherited one, and objects with a null prototype do not inherit from `Object.prototype`.

The main defensive lessons are to validate permitted input fields, use safe merging, and keep authorization state under trusted server-side control. Own-property checks can prevent inherited values being mistaken for explicit state, but do not make user-controlled privilege fields safe. See [MDN's prototype pollution guidance](https://developer.mozilla.org/en-US/docs/Web/Security/Attacks/Prototype_pollution).

## Task 3 — React / Next.js

React provides the UI components, while Next.js adds routing and server-side rendering. This task focuses on identifying a Next.js App Router application and understanding how middleware authorization can fail.

### Fingerprinting and React Server Components

| Clue in my responses | Interpretation |
| --- | --- |
| `X-Powered-By: Next.js` | The server identifies itself as Next.js. |
| `Vary` includes `RSC` and `Next-Router-*` entries | Consistent with App Router requests and different response representations. |
| `x-nextjs-cache: HIT` and prerender headers | Next.js cache and prerender metadata for the requested page. |
| `/_next/static/` asset paths | Framework-generated JavaScript assets in the returned HTML. |
| `self.__next_f.push(...)` scripts | Inline data associated with Next.js streaming the React Server Component payload. |

Together, these clues support the framework identification; they do not establish an exact version.

React Server Components render on the server. Their rendered tree, references to Client Components, and passed props travel in the RSC payload using React's Flight format. On the first load, Next.js also returns HTML; the browser uses the payload to reconcile the component tree and JavaScript to hydrate interactive Client Components. Data sent to the browser remains visible to the client. See the [Next.js component documentation](https://nextjs.org/docs/app/getting-started/server-and-client-components).

### Commands and Options

These commands are from my terminal session, with the lab address replaced by `<LAB_IP>`.

```bash
curl -I "http://<LAB_IP>:3001/"
curl -v "http://<LAB_IP>:3001/dashboard"
```

| Option | Purpose |
| --- | --- |
| `-I` | Sends HEAD and displays response headers without the body. |
| `-v` | Displays connection details and request/response headers alongside the response body. |
| `-H` | Supplies a request header; used in the later lab comparison. |

The dashboard command does not use `-L`, so curl displays the initial redirect instead of following it.

### Practical Observations

| Check | Observed result |
| --- | --- |
| Root HEAD request | `HTTP/1.1 200 OK`, an HTML content type, and the Next.js-related headers above. |
| Ordinary dashboard request | `HTTP/1.1 307 Temporary Redirect` with `Location: /login`. |
| Modified request in the lab | Dashboard HTML was returned, including Next.js assets and inline RSC data. |

The modified request captured the response body without a status line. The observed access change is consistent with the middleware-bypass scenario; no remote code execution was tested.

### Screenshots

**Next.js response headers**

![Root HEAD response showing Next.js, RSC, router, and cache headers](./assets/modern-web-stacks/nextjs-response-headers.png)

**Dashboard access before the lab comparison**

![Ordinary dashboard request returning a 307 redirect to the login page](./assets/modern-web-stacks/nextjs-dashboard-redirect.png)

**Dashboard response after the lab comparison**

![Dashboard HTML and inline RSC data after the lab test, with both copies of the flag redacted in the supplied screenshot](./assets/modern-web-stacks/nextjs-dashboard-response-redacted.png)

### Middleware Authorization — CVE-2025-29927

Middleware can check an incoming request before a protected page is reached. Affected Next.js versions trusted an internal `x-middleware-subrequest` header in a way that allowed middleware execution to be skipped. Applications relying entirely on those middleware checks could expose protected routes.

The room also distinguishes middleware placed at the project root from middleware under `src`; project layout affects its internal identification. The important lesson is the trust boundary: client-supplied metadata must not decide whether an authorization check runs.

For local reproduction, the room specifies a production build: `npm run build` followed by `npm start`, with scripts mapped to `next build` and `next start`. Development mode (`next dev`) is a different execution environment. These are setup notes, not commands shown in my terminal evidence. See the [Next.js CLI reference](https://nextjs.org/docs/app/api-reference/cli/next).

### Related RSC Vulnerability — CVE-2025-55182

This is a separate issue involving unsafe decoding of requests in affected React Server Components packages, potentially allowing unauthenticated remote code execution. Finding RSC markers alone does not demonstrate this vulnerability. It is included as context from the task; the recorded lab exercise concerned middleware authorization. See the [React security advisory](https://react.dev/blog/2025/12/03/critical-security-vulnerability-in-react-server-components).

### Key Takeaways

- Combine header and HTML clues before identifying a stack; check versions and deployment conditions separately.
- Enforce authorization where protected data is accessed, as well as any early middleware checks.
- Apply the relevant framework security updates. For CVE-2025-29927, filtering externally supplied `x-middleware-subrequest` headers is the documented interim workaround.

References: [Next.js middleware advisory](https://github.com/vercel/next.js/security/advisories/GHSA-f82v-jwr5-mffw) · [Vercel's technical postmortem](https://vercel.com/blog/postmortem-on-next-js-middleware-bypass).

[Back to the section overview](./README.md)
