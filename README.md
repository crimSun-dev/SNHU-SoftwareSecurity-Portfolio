# CS 305 Software Security Portfolio

**Student:** Draven Chen  
**Course:** CS-305 Software Security  
**Scenario:** Artemis Financial

## Portfolio Artifact

This repository contains the completed **Artemis Financial Practices for Secure Software Report** from Project Two. The report documents the checksum design, certificate and TLS configuration, dependency remediation, and the tests used to confirm the refactored application stayed functional and secure.

- [CS 305 Practices for Secure Software Report.docx](./CS%20305%20Practices%20for%20Secure%20Software%20Report.docx)

## Journal Reflection

### Client and software requirements

Artemis Financial is a financial consulting firm that builds individualized savings, retirement, investment, and insurance plans. Global Rain asked me to secure the firm’s Spring Boot REST API before it exposed customer records on a public network. Those records include legal names, dates of birth, Social Security numbers, account and routing numbers, balances, and insurance information. The firm needed encrypted communications and a way to verify that financial documents were not altered in transit.

### Finding vulnerabilities, and why secure code matters

The manual review was strongest where each finding named a file, a line, a CWE, a severity, and the effect on Artemis Financial, rather than stopping at a category name. The dependency scan was useful for the same reason: it reported 270 findings, and most of them collapsed into one action, upgrading the unsupported Spring Boot 2.2.4 parent. Secure code matters here because the product is the customer’s trust. A changed account number or a leaked plan is a direct loss, and a firm that holds nonpublic personal information is expected to encrypt data in transit and manage known vulnerabilities. Software security protects the firm’s money, its regulatory standing, and the reputation it sells.

### What was challenging or helpful

The difficult part was making the static test honest. The supplied dependency-check 5.3.0 plugin could not reach NIST’s retired NVD 1.1 feeds and failed with HTTP 403, so a failed or stale scan would have been meaningless. Moving to dependency-check 12.1.0, then separating real risk from duplicated CPE matches, was the useful discipline. The vulnerability assessment process flow helped by forcing each security area to be justified against the actual architecture, including treating Client/Server as limited for a single-process service so transport findings were not counted twice.

### Layers of security, and how I would assess risk next time

I added protection from the data outward. SHA-256 checksums with constant-time comparison protect integrity. TLS 1.3 and 1.2 with AES-GCM and ECDHE, and no plaintext listener, protect the channel. Length limits, HTML encoding, a fail-closed Spring Security policy, and browser headers (HSTS, Content-Security-Policy, nosniff, and X-Frame-Options) cover what TLS does not. The dependency upgrade removed the known CVEs. Next time I would keep a current dependency-check release in the build, fail the build above an agreed CVSS threshold, generate a software bill of materials, and choose mitigations by exploitability and business impact. SpotBugs with FindSecBugs would catch defects a dependency scan cannot see.

### Proving the application was functional and secure

A successful start was not treated as proof. After each change I ran the full Maven test suite: eleven tests and zero failures, including NIST SHA-256 vectors, a tamper check, an XSS regression, and header checks. I confirmed the live digest with `sha256sum`, confirmed TLS 1.3 and a refused TLS 1.0 handshake with OpenSSL and curl, and confirmed a script payload came back escaped. To see whether refactoring introduced new vulnerabilities, I re-ran dependency-check after every upgrade. Spring Boot 3.5.6 still had 143 findings, 4.1.1 had 11 in Tomcat, and pinning Tomcat 11.0.25 brought the scan to zero vulnerabilities across 33 artifacts. The tests passed after each of those passes.

### Resources, tools, and practices worth reusing

I relied on OWASP Dependency-Check, the OWASP Top 10 and ASVS, CWE, and the National Vulnerability Database, along with NIST SP 800-52, NIST SP 800-131A, and FIPS 180-4. In code I stayed with platform components: Java `MessageDigest`, Spring `HtmlUtils`, Spring Security’s header writers, and `keytool` for the certificate. The practice I would repeat is to keep cryptography in one place, encode output, fail closed, and put a regression test on every security fix.

### What I would show a future employer

I would show this report. It records a vulnerability assessment of a financial API, a prioritized remediation plan, and a refactor that added checksum verification and TLS, then drove known dependency vulnerabilities from 143 to zero without breaking the test suite. That is evidence I can find issues, rank them, fix them, and prove the fix.
