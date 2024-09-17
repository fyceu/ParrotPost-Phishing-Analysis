## Scenario
While working as a SOC Analyst for _Flying-Sec_, you receive an incoming report from senior executive Paul Feathers. Paul recently received an email from _ParrotPost_, a legitimate company email tool, asking him to log into his account to resolve an issue with his account information.

Your task is to investigate the email and determine whether it is a legitimate request or a phishing attempt. 

***Answers to the Room's questions can be found at the very end of the writeup.***
## Analysis

### Header Analysis
   Our first step is to conduct header analysis on the provided EML file: ```URGENTParrotPostAccountUpdateRequired.eml```

We can run the following command to read through said file  and obtain important artifacts:
```bash
subl /root/Rooms/ParrotPost/URGENTParrotPostAccountUpdateRequired.eml
```

![Screenshot_2](https://github.com/user-attachments/assets/ed4fc5ba-f3d4-480b-b4cf-a8dd4db3cc3b)

Below are some of the common artifacts extracted during header analysis:
- Date: ```Sun, 30 Apr 2023 20:50:15 -0000```
- Subject: ```URGENT: ParrotPost Account Update Required```
- From: ```Parrot Post Webmail <no-reply@postparrot.thm>```
- To: ```Paul Feathers <pfeathers@flying-sec.thm>```
- IP Address: ```109.250.120.0```

Using the captured IP, we can conduct Reverse IP Lookup to find more information about the sender.
![Screenshot_3](https://github.com/user-attachments/assets/4537a44c-2a76-4baa-8b34-8021af747e8d)
### Email Attachment Analysis

Doing some more analysis within the EML file, we can see that there was an attachment associated with this email, ```ParrotPostACTIONREQUIRED.htm```, which was encoded in base64.
![Screenshot_4](https://github.com/user-attachments/assets/99ba5f30-1959-425a-a8cc-433a424fc904)

Since the attachment is in HTML, we can take a peek into this attachment within Sublime
```bash
subl /root/Rooms/ParrotPost/ParrotPostACTIONREQUIRED.htm
```

This document declares a variable ```b64``` and sets it to a long string of encoded data.
![Screenshot_5](https://github.com/user-attachments/assets/39bf1d5d-876a-4dc2-9a50-00f3ae76f333)

This variable is followed by some Javascript functions.
![Screenshot_6](https://github.com/user-attachments/assets/28186a8e-7001-424c-b2df-97684a1a3607)

Let's dissect the following code
```javascript
document.write(unescape(atob(b64)));
```
- ```atob()```: function decodes string based on parameter, in this case base64
- ```unescape()```: function converts and escaped characters in the decoded string into their original form
- ```document.write()```: displays decoded and unescaped string onto webpage

We can decode the ```b64``` variable by copying and pasting its contents into [CyberChef](https://gchq.github.io/CyberChef/)
![Screenshot_7](https://github.com/user-attachments/assets/815f56ee-2d15-413a-91f6-b2b088ac48ca)

The format of the decoded HTML follows this format:
1. decoded HTML
2. encoded header (indicated by < h1> tag)
3. decoded comments (indicated by <!)


Perusing through the decoded output, we can see that this HTM file generates a fake login portal of ParrotPost. 
- If a user submits their credentials, they get captured, encoded, and sent to their evil server. 
- The page will then toss an error and redirect them to the legitimate ParrotPost login page. 

Let's save this output to a new file: ```decoded_webpage.htm```

There also includes a base64 string within the decoded comments that didn't decode completely. Let's apply Form Base64 to it one more time to see its output.
![Screenshot_8](https://github.com/user-attachments/assets/a594d503-0ba4-4e7c-9f21-9efd3aa150c4)

### Obfuscation

Open up ```decoded_webpage.htm```

Within the header, the adversary utilizes HTML Entity Encoding to obfuscate their code, making it difficult for humans to decipher the code's logic. We can utilize CyberChef once again to decode the header. 

![Screenshot_9](https://github.com/user-attachments/assets/c2148f3c-4a40-4795-8d8d-b3d4e9c27f6a)

The adversary also utilizes minimization to remove any spaces and whitespace between lines. This makes the code more difficult to dissect. Let's utilize CSS Beautify and JavaScript Beautify to make the HTML code more readable for our eyes. Replace the output to our ```decoded_webpage.htm``` file.

![Screenshot_10](https://github.com/user-attachments/assets/8a3d8e31-8bbb-4c29-97af-61db743c4891)

Now that the code is more digestible, we are able to see the logic behind their code. They also left comments that helps us understand what each statement does.
![Screenshot_11](https://github.com/user-attachments/assets/a7d48978-17bb-4d8b-b28f-a528659389f0)

## Network Monitor
**Typically, we would never want to open any malicious attachments received via email.**
**We would want to use analysis and sandboxing tools (URL2PNG, HybridAnalysis, URLScan, VMs, etc.) for safe investigations.** 

However, within this sandbox environment, we can go a step further by monitoring our network when submitting fake credentials to the malicious login page. 

Upon opening the malicious webpage file, we can verify that it is a fake login page for ParrotPost. It autofilled the email field with the victim's email in hopes they input their legitimate credentials. 

![Screenshot_12](https://github.com/user-attachments/assets/c06c7ce5-6e15-4c5b-a3fc-da1893688c4e)

Let's open up the Network Tab within Firefox's developer tools. Then let's submit fake credentials instead and monitor for any HTTP Requests. 
- email: test@test.com 
- password: test

Once clicking the login button, we are presented with an HTTP GET request. 
- ```Code 200```: successfully connected to the endpoint ```evilparrot.thm```
- Our email and password were captured and sent as query parameters in the GET request
![Screenshot_13](https://github.com/user-attachments/assets/83ac50fc-ea8c-4acc-bd11-f2f75772b2cc)


We can view the response tab to view any response content. 
![Screenshot_14](https://github.com/user-attachments/assets/a3f6e25a-5918-4a2e-92a5-8ae07c4ba882)

Heading to the page ```http://evilparrot.thm:8080/creds.txt``` reveals a dump of all captured credentials.
![Screenshot_15](https://github.com/user-attachments/assets/662875d9-49f6-4724-98ee-b6697109b1af)
## Questions
1. According to the IP Address, what country is the sending email server associated with? ```Latvia```
2. If Paul replies to this email, which email address will his reply be sent to? ```no-reply@postparr0t.thm```
3. What is the value of the custom header in the email? ```THM{y0u_f0und_7h3_hj34d3er}```
4. What encoding scheme is used to obfuscate the web page contents? ```base64```
5. What is the built-in JavaScript function used to decode the web page before writing it to the page? ```atob()```
6. After the initial base64 decoding, what is the value of the leftover base64 encoded comment? ```THM{d0ubl3_3nc0d3d}```
7. After decoding the HTML Entity characters, what is the text inside of the h1 tag?```ParrotPost Secure Webmail Login```
8. What is the reverse of CSS Minify? ```CSS Beautify```
9. What is the URL that receives the login request when the login form is submitted? ```http://evilparrot.thm:8080/cred-capture.php```
10. What is the JavaScript property that can redirect the browser to a new URL? ```window.location.href```
11. What is the flag you receive after sending fake credentials to the/cred-capture.php endpoint? ```THM{c4p7ur3d_y0ur_cr3d5}```
12. What is the path on the web server hosting hte log ofd captured credentials ```/creds.txt```
13. Based on the log, what is Chris Smith's password? ```FlyL1ke!A~Bird```