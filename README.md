TryHackMe: Carnage - DFIR & Network Packet Analysis

## Overview
This repository contains my investigation notes and findings for the Carnage lab on TryHackMe. The objective of this analysis is to inspect a network packet capture (PCAP) following a security alert, reconstruct the attack lifecycle, and extract key Indicators of Compromise (IoCs).

Using Wireshark for deep packet inspection alongside VirusTotal for threat intelligence correlation, I mapped out the complete intrusion chain—from initial malspam delivery to secondary C2 beaconing and target web stack identification.

---

## Technical Artifacts & Indicators of Compromise (IoCs)

### Initial Access & Malspam Infrastructure
* Malspam Sender: farshin@mailfa.com
* SMTP Server Domain: mail.mailfa.com
* Malicious Staging Payload: chart-1530076591.xls
* Payload Hosting Infrastructure: maldivehost.net

### Reconnaissance & Discovery
* External IP Lookup Service: api.ipify.org
* Reconnaissance Timestamp: 2021-09-24 17:00:04 UTC

### Command & Control (C2) & Evasion
* Primary C2 Endpoint: 185.106.96.158 (survmeter.live)
* Secondary C2 Endpoint: 185.125.204.174 (securitybusinpuff.com)
* Host Header Spoofing (Malleable C2): Spoofed ocsp.verisign.com to bypass perimeter filters
* Initial POST Beacon Payload Size: 281 bytes

### Target Endpoint & Vulnerable Stack
* Server Banner: Apache/2.4.49 (cPanel) OpenSSL/1.1.1l mod_bwlimited/1.4
* Associated Vulnerability: Apache 2.4.49 Path Traversal & RCE (CVE-2021-41773 / CVE-2021-42013)

---

## Investigation Walkthrough

### 1. Phishing & Initial Delivery Vector
* Filtered packet traffic using smtp to isolate inbound email sessions.
* Reconstructed the TCP stream to analyze raw SMTP interaction (AUTH LOGIN), identifying the sender address (farshin@mailfa.com) and sending mail server (mail.mailfa.com).
* Identified secondary HTTP traffic retrieving a malicious Excel spreadsheet (chart-1530076591.xls) hosted on maldivehost.net.

### 2. Post-Infection Discovery & Recon
* Performed a string search across packet details for api to locate outbound reconnaissance calls.
* Uncovered an outbound DNS query for api.ipify.org recorded at 2021-09-24 17:00:04 UTC, establishing the exact timeframe of post-compromise host discovery.

### 3. Command & Control (C2) Analysis
* Applied the display filter http.request.method == POST to isolate outbound C2 callbacks.
* Examined the initial POST request body length (281 bytes) transmitted to the remote server.
* Discovered Cobalt Strike C2 traffic directed at 185.106.96.158 (survmeter.live). The operator configured a Malleable C2 profile spoofing legitimate Host headers (ocsp.verisign.com) to obscure callback activity.
* Correlated IP indicators with VirusTotal community sandbox data to uncover an additional secondary C2 server at 185.125.204.174 (securitybusinpuff.com).

### 4. Server Banner & Stack Fingerprinting
* Carved HTTP response headers across TCP streams to evaluate server responses.
* Extracted the full server response banner: Apache/2.4.49 (cPanel) OpenSSL/1.1.1l mod_bwlimited/1.4.
* Confirmed the target environment was running Apache 2.4.49, a version widely targeted for path traversal and arbitrary code execution flaws.

---

## Key Takeaways & Applied Skills
* Deep Packet Inspection: Utilized custom display filters, protocol hierarchy statistics, and TCP stream reconstruction in Wireshark.
* Evasion Recognition: Identified domain fronting and Host header spoofing techniques used to hide malicious outbound traffic.
* Threat Intelligence Correlation: Cross-referenced network packet indicators with OSINT and VirusTotal sandbox telemetry to expose hidden C2 infrastructure.
