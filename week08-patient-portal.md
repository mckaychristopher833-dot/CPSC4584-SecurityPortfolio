# Week 8 Lab: Patient Portal Web Vulnerability & HTTP Analysis

**Organization:** Maplewood Health System  
**System Evaluated:** Patient Portal (`portal.maplewoodhealth.org`)  
**Incident Reference:** WEB-2026-1019-001  
**Date:** October 19, 2026  
**Analyst:** Christopher McKay

---

## 1. Incident Brief & Summary

### Executive Summary
On October 17, 2026, the Maplewood IT helpdesk received three separate patient reports describing unexpected behavior on the patient portal (`portal.maplewoodhealth.org`):
1. A patient reported being redirected to an unfamiliar external site after submitting credentials.
2. A second patient noticed their username and password visible in plain text within the browser address bar.
3. A third patient noted that the portal was accessible over unencrypted HTTP without a browser security warning.

Following helpdesk escalation, the Security Operations Center (SOC) reviewed web application server logs and HTTP traffic. The investigation confirmed four structural vulnerabilities within the login flow deployment. No active exploitation or malicious breach has been confirmed. The observed vulnerabilities present significant exposure risks for patient credentials and Protected Health Information (PHI).

### Observed Vulnerability Findings

| Finding | Location | Observed Value | Vulnerability Classification | Severity | Severity Justification |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1** | Login request method | `GET /login?username=jsmith&password=Summer2026!` | Credential Exposure in URL | High | Directly exposes plaintext credentials across browser histories, server logs, and proxy records. |
| **2** | Login response `Set-Cookie` | `Set-Cookie: sessionid=a3f8c2d1; Path=/` | Missing Cookie Flags (`HttpOnly`, `Secure`, `SameSite`) | High | Leaves active session tokens vulnerable to XSS script theft, network interception, and CSRF attacks. |
| **3** | Login response headers | Missing `Strict-Transport-Security`, `Content-Security-Policy`, `X-Frame-Options` | Missing Security Response Headers | Medium | Allows protocol downgrade to HTTP, frame embedding (clickjacking), and inline script execution. |
| **4** | Post-login redirect | `next=https://external-site.com/collect` accepted and executed | Unvalidated Redirect (Open Redirect) | High | Enables attackers to craft legitimate portal links that redirect users to external phishing destinations. |

---

## 2. Technical Written Analysis

### Prompt 1: GET vs. POST Credential Exposure Analysis
The Maplewood patient portal submits authentication credentials using an HTTP `GET` request (`GET /login?username=jsmith&password=Summer2026!`), placing sensitive credentials directly into URL query parameters. Data embedded in GET query strings is regularly recorded across browser histories, client-side web logs, web server access logs, intermediate proxy/firewall logs, and analytics tools, and can be transmitted to external sites via the `Referer` header.

To mitigate this risk, the login form must be updated to use the HTTP `POST` method over TLS (`HTTPS`), passing credentials within the request body rather than the URL. While `POST` does not encrypt data on its own, it prevents sensitive credential strings from being permanently logged in URLs, browser address bars, and web infrastructure logs.

### Prompt 2: Session Cookie Security Assessment
The application sets session identifiers using `Set-Cookie: sessionid=a3f8c2d1; Path=/` without essential security flags:
- **Missing `HttpOnly`:** Permits client-side JavaScript to read the session cookie, leaving the account vulnerable to session hijacking if a Cross-Site Scripting (XSS) vulnerability is exploited.
- **Missing `Secure`:** Allows the cookie to be transmitted over unencrypted HTTP connections, exposing the session token to network interception and Man-in-the-Middle (MitM) attacks.
- **Missing `SameSite`:** Fails to restrict cross-site cookie transmission, increasing exposure to Cross-Site Request Forgery (CSRF) attacks.

The application server must issue session cookies with all security attributes enabled: `Set-Cookie: sessionid=...; Path=/; HttpOnly; Secure; SameSite=Strict`.

### Prompt 3: Developer Escalation & Action Plan

#### Incident Details
- **Incident ID:** WEB-2026-1019-001
- **Target System:** Patient Portal (`portal.maplewoodhealth.org`)
- **Escalation Recipients:** Web Development Team, Application Security Engineering, and SOC Leadership

#### Confirmed Findings & Technical Impact
1. **Credential Exposure (GET Method):** Plaintext credentials are stored in server access logs and browser histories.
2. **Insecure Session Tokens:** Lack of `HttpOnly` and `Secure` attributes exposes active session tokens to hijack risks via XSS or network sniffing.
3. **Missing Browser Protections:** Lack of HSTS, CSP, and X-Frame-Options allows protocol downgrade, framing, and uncontrolled script execution.
4. **Open Redirection:** Parameter `next=` redirects users to untrusted external URLs (`https://external-site.com/collect`), enabling highly credible phishing lures.

#### Proposed Action Items for Remediation
1. **HTTP Method Update:** Convert the authentication form to issue HTTP `POST` requests over HTTPS with credentials in the request body.
2. **Cookie Hardening:** Configure the web server and application framework to attach `HttpOnly`, `Secure`, and `SameSite=Strict` attributes to all session cookies.
3. **Security Response Headers:** Implement `Strict-Transport-Security` (HSTS with `max-age=31536000`), `Content-Security-Policy` (CSP), and `X-Frame-Options: DENY` (or `frame-ancestors 'none'`) across all portal HTTP responses.
4. **Redirect Validation:** Enforce strict server-side URL validation or fixed relative routing for post-login redirects to prevent unvalidated external redirects.

#### Responsibility Boundaries
- **SOC Analyst Role:** Inspect and document raw HTTP evidence, assess risk exposure, maintain audit logs, and escalate findings through appropriate communication channels. SOC analysts do **not** modify portal source code, reconfigure production web servers, contact patients directly, or execute penetration tests on live infrastructure.
- **Development & Security Engineering Role:** Author code fixes, update web server configurations, perform validation testing in non-production environments, and deploy patches to production.

---

## 3. Terminal HTTP Header Inspection Log (`example.com`)

> **Note:** The command outputs below represent benign practice observations executed against `example.com` in the CyLab Security Academy webshell. They serve to demonstrate HTTP inspection methods and do **not** represent live Maplewood Health System incident evidence.

### Entry 1: Retrieve HTTP Response Headers
- **Command:**
  ```bash
  curl -I [http://example.com](http://example.com)
