---

## Incident Summary

At 2:14 AM on September 27, 2026, an unauthorized foreign actor executed a distributed credential stuffing attack against the Maplewood Health System billing portal, authenticating as `billing_admin_03` using a compromised password (`Maplewood2024!`). During the 47-minute active session, the attacker accessed sensitive insurance records for 3,247 unique patients—including names, dates of birth, policy numbers, and claim histories—before the SOC on-call analyst terminated the session.

---

## TLS Assessment

**TLS Version:** TLS 1.3  
**Status:** Compliant  
**What TLS Protected:** Transport security, channel confidentiality, and data integrity between the client browser and the server, protecting the session payload against packet sniffing, tampering, and man-in-the-middle interception.  
**What TLS Did Not Protect:** Application-layer authentication and identity verification. TLS verified the server's identity to the client, but it could not determine whether the human user presenting valid credentials was an authorized administrator or an imposter.

---

## Authentication Controls Gap Analysis

| Control | Required | Status | Finding |
|---------|----------|--------|---------|
| MFA | Maplewood sensitive-account standard | Not Implemented | A password alone was sufficient to authenticate to a privileged administrative account accessing sensitive patient data. Implementing MFA would have blocked access even with a known password. |
| Failed Attempt Protection | Account-based throttling and alerting | Not Implemented | 847 failed attempts over 72 hours across 12 IP addresses triggered no account-level throttling or alert. NIST SP 800-63B-4 sets an upper bound of 100 consecutive failures for rate limiting. |
| Password Policy | NIST SP 800-63B-4 aligned | Needs Improvement | Required 12 characters and composition rules but lacked blocklist screening against leaked or commonly used values. `Maplewood2024!` satisfied local rules despite being compromised. |
| Compromised Credential Response | Detect and invalidate confirmed compromised authenticators | Not Implemented | The password appeared in a breach dataset six months prior, but Maplewood lacked an automated or operational process to screen for leaked authenticators and force a reset. |
| Automated Attack Controls | Throttling, bot detection, or adaptive controls as appropriate | Not Implemented | No bot detection, adaptive risk-based challenges (CAPTCHA), or behavioral checks were present to slow or detect the multi-IP distributed authentication script. |

---

## OpenSSL Commands Practiced

| Command | Purpose |
|---------|---------|
| `openssl genrsa -out private_key.pem 2048` | Generates a 2048-bit RSA private key pair containing the prime factors ($p, q$), private exponent ($d$), and modulus ($n$) used for cryptographic identity and key exchange. |
| `openssl rsa -in private_key.pem -pubout` | Mathematically derives and outputs the public key ($e, n$) from the private key structure, which can be freely shared or placed in TLS X.509 certificates. |
| `openssl rsa -in private_key.pem -text -noout` | Decodes and displays the internal ASN.1 structure of the private key, exposing the raw prime factors, exponents, CRT coefficients, and modulus without printing the encoded file. |
| `cat public_key.pem` | Prints the PEM-formatted ASCII file bounded by `BEGIN PUBLIC KEY` tags, containing the Base64-encoded DER representation of the public key structure. |

---

## Escalation Summary

Confirmed findings establish that account `billing_admin_03` was compromised via credential stuffing after 847 failed login attempts, exposing 3,247 patient insurance records over a 47-minute window. While transport encryption (TLS 1.3) functioned as intended, critical application-layer control failures occurred due to the absence of MFA, lack of NIST SP 800-63B-compliant compromised password screening, and missing account-level rate limiting. 

Decisions requiring authorized leadership review include determining whether session tokens were exfiltrated for persistent access, reviewing system-wide credential reset requirements, formally approving mandatory MFA enforcement across all billing endpoints, and initiating required HIPAA breach notification procedures.

---
*CPSC 4584 | Governors State University | Fall 2026*
