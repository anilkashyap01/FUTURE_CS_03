# 🛡️ API Security Risk Analysis Report

**Target API:** `https://reqres.in` (ReqRes API Testing Platform)
**Assessment Type:** Read-Only API Security Risk Analysis  
**Tools Utilized:** Postman (HTTP Client), Browser DevTools  
**Date:** August 30, 2026


## 📋 1. Executive Summary
A security assessment was conducted against the ReqRes demo API to identify architectural security flaws aligning with the OWASP API Security Top 10. The analysis focused on endpoint exposure, data formatting, and rate-limiting controls. The assessment revealed vulnerabilities related to unauthenticated data access and excessive data exposure (exposing PII), though the platform demonstrated robust rate-limiting controls.

---

## 🔍 2. Identified API Security Risks

### Risk 1: Open / Unauthenticated Endpoint (Broken Authentication)
*   **Severity:** 🔴 **HIGH**
*   **Endpoint Tested:** `GET https://reqres.in/api/users/2`
*   **Observation:** The API endpoint was successfully queried without providing any authentication headers (e.g., Bearer Token, API Key, or Session Cookie). The server responded with an HTTP `200 OK` status and returned the requested user record. *(See evidence in `Screenshot From 2026-08-30 16-45-52.png`)*
*   **Business Impact:** In a production SaaS environment, leaving user data endpoints unauthenticated allows malicious actors to access, scrape, and exfiltrate the entire user database anonymously, leading to severe privacy breaches and regulatory fines.
*   **Remediation Suggestion:** 
    *   Implement strict Authentication controls (e.g., OAuth 2.0 or JWT).
    *   Ensure the API gateway drops requests lacking a valid `Authorization` header, returning an HTTP `401 Unauthorized` status.

### Risk 2: Excessive Data Exposure (PII Leakage)
*   **Severity:** 🟡 **MEDIUM / HIGH**
*   **Endpoint Tested:** `GET https://reqres.in/api/users/2`
*   **Observation:** The JSON response payload returned complete user objects, including `id`, `email`, `first_name`, `last_name`, and `avatar` URL links. 
*   **Business Impact:** APIs often return all available database fields for a user, assuming the front-end application will filter what the user actually sees. Attackers intercepting the raw API traffic can view sensitive fields (like emails or internal IDs) that were not intended for public display.
*   **Remediation Suggestion:** 
    *   Implement strict data filtering at the API layer (not the client level).
    *   Use GraphQL or specific REST query parameters (e.g., `?fields=first_name,last_name`) to ensure the API only returns the exact data necessary for the UI component being rendered.

### Risk 3: Rate Limiting & Resource Exhaustion Controls
*   **Severity:** 🟢 **LOW (Mitigated / Controlled)**
*   **Endpoint Tested:** `GET https://reqres.in/api/users` (Rapid succession testing)
*   **Observation:** The testing successfully triggered the server's rate-limiting defenses. The server responded with an HTTP `429 Too Many Requests` status code. 
*   **Evidence from Headers:** 
    *   `X-Ratelimit-Limit: 40`
    *   `X-Ratelimit-Remaining: 0`
    *   *JSON Body:* `"error": "rate_limit_exceeded"`
*   **Business Impact:** Because the API correctly limits unauthenticated requests, it is highly resilient against automated scraping, brute-force attacks, and Denial of Service (DoS) attempts.
*   **Remediation Suggestion:** The current implementation is secure. To further enhance visibility, ensure that `429` errors are actively logged and monitored by the SOC team to identify aggressive IP addresses.

---

## 🛠️ 3. Security Headers Analysis

Passive inspection of the HTTP Response Headers revealed several positive security configurations:
*   `X-Content-Type-Options: nosniff`: Prevents MIME-sniffing attacks.
*   `X-Frame-Options: DENY`: Protects against Clickjacking attacks.
*   `Strict-Transport-Security: max-age=31536000; includeSubDomains`: Enforces secure HTTPS connections.
*   `X-Xss-Protection: 0`: While seemingly counter-intuitive, setting this to `0` is modern best practice when paired with a strong Content Security Policy (CSP).


---


## 📁 4. Repository Structure
- `/report/` - Contains the final PDF API Security Risk Analysis Report.
- `/screenshots/` - Contains proof-of-concept visual evidence from Postman:
  - `Screenshot From 2026-08-30 16-32-26.png` (429 Rate Limit Verification)
  - `Screenshot From 2026-08-30 16-45-52.png` (200 OK Unauthenticated Data Exposure)
