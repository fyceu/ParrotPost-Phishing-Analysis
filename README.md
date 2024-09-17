## ParrotPost
This is a writeup for ParrotPost: Phishing Analysis room on TryHackMe. You can read more [here](https://tryhackme.com/r/room/parrotpost)
## Scenario
While working as a SOC Analyst for _Flying-Sec_, you receive an incoming report from senior executive Paul Feathers. Paul recently received an email from _ParrotPost_, a legitimate company email tool, asking him to log into his account to resolve an issue with his account information.

Your task is to investigate the email and determine whether it is a legitimate request or a phishing attempt. 
## Executive Summary
### Analysis Steps

1. **Header Analysis**: Extracted email header artifacts
2. **Attachment Analysis**: Utilized [CyberChef](https://gchq.github.io/CyberChef/) to decode the HTML attachment
3. **Code De-Obfuscation**: Simplified the obfuscated code for analysis
4. **Network Analysis**: Monitored network for successful HTTP GET requests to malicious server
### Findings

- Email was sent from a spoofed email address with an IP address originating from Latvia
- Email attachment contained HTML, encoded in Base64, that creates a fake ParrotPost Login Page
- Upon de-obfuscation of the HTML, it was discovered that the phishing page captures submitted credentials
- Submitting fake login credentials to through the phishing login page, we observed the credentials being sent as query parameters in an HTTP GET request to the malicious server
- Investigating HTTP Response led us to a credential dump of the attacker

**An in-depth analysis and determination of my findings can be found [here](https://github.com/fyceu/ParrotPost-Phishing-Analysis/blob/main/Analysis.md)**

**Answers to the questions in this room can be found [here](https://github.com/fyceu/ParrotPost-Phishing-Analysis/blob/main/Analysis.md#questions)**
## Recommendations
- **Password reset**:
	- Reset the password of Paul Feather's account to ensure prevent unauthorized access
- **Monitor for unusual activity**:
	- Monitor past activity on Paul's account to determine if unauthorized access occurred
- **Block malicious IP and Domain**:
	- Block ```109.250.120.0```
		- IP address utilized specifically for malicious purposes.
		- No business is conducted with this IP address, therefore it will cause no disruption to business operations
	- Block ```evilparrot.thm```
		- Entire Domain is utilized specifically for malicious purposes. 
		- No Business is conducted with this domain, therefore blocking it will cause no disruption business operations
- **Security awareness training**:
	- Notify employees of a successful phishing attempt occurring within the company. Remind employees to not click links in emails, open unsolicited attachments, and the potential risks that can occur of a successful phishing attack.
	- Have Paul complete Security Awareness Training about Phishing
- **Email Filtering**:
	- Tune email security filters to flag emails received from external sources
	- Flag emails that contain suspicious attachments (HTML)
- **Report findings**: 
	- Report the phishing attempt to the legitimate email provider ParrotPost
	- Report to any security authorities, providing details of findings such as malicious IP addresses, domains, hashes of any attachments, etc. 

