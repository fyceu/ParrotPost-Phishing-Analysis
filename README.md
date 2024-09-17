## ParrotPost
This is a writeup for ParrotPost: Phishing Analysis room on TryHackMe. You can read more [here](https://tryhackme.com/r/room/parrotpost)
## Scenario
While working as a SOC Analyst for _Flying-Sec_, you receive an incoming report from senior executive Paul Feathers. Paul recently received an email from _ParrotPost_, a legitimate company email tool, asking him to log into his account to resolve an issue with his account information.

Your task is to investigate the email and determine whether it is a legitimate request or a phishing attempt. 
## Executive Summary
### Analysis Steps

1. Header Analysis: Extracted email header artifacts
2. Attachment Analysis: Utilized [CyberChef](https://gchq.github.io/CyberChef/) to decode the HTML attachment
3. Code De-Obfuscation: Simplified the obfuscated code for analysis
4. Network Analysis: Monitored network for successful HTTP GET requests to malicious server
### Findings

- EMAIL CONFIRMED TO BE MALICIOUS
- ATTACHMENT CONTAINED ENCODED HTML, JAVASCRIPT, AND CSS
- NETWORK CAPTURED PROVED CREDENTAILS BEING STOLEN AND REDIRECTED TO EVIL SERVER

An in-depth analysis and determination of my findings can be found here
## Conclusion
### Recommendations


