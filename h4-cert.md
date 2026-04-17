 # X) Read/Watch/Listen & Summarize

**Objective**
- A few bullet points per section is enough to summarize
- Add your own observation, question or idea


## X.1) [OWASP A01:2021](<https://owasp.org/Top10/2021/A01_2021-Broken_Access_Control/index.html>) - Broken Access Control
**Common access control vulnerabilities:**
- Violation of the principle of least privilege
- Bypassing access control checks with URL tampering
- Insecure direct object references
- Abusing JWT
- CORS misconfiguration


## X.2) PortSwigger Academy
[IDOR](<https://portswigger.net/web-security/access-control/idor>)
- a.k.a Insecure Direct Object References
- A vulnerability that arises from accessing objects directly with user-supplied input
- Most commonly associated with horizontal privilege escalation
 

[Path Traversal](<https://portswigger.net/web-security/file-path-traversal>)
- This type of vulnerability arises when a web server retrieves _files_ based on a _filename_ **appended** to a **base file path**
- Say a web server is serving content from `/home/webUser/content/*`, this is the base file path!
- If no controls are in place, one could read files from the home directory by tampering with the URL and replacing allowed file, say `file1`, with `../secretFile.txt`
  - Simply put: `/home/webUser/content/file1` now becomes `/home/webUser/secretFile`


[Cross-site scripting (XSS)](<https://portswigger.net/web-security/cross-site-scripting>)
- This vulnerability allows an attacker to inject malicious scripts into a trusted site, causing the victim’s browser to execute them as if they were legit. Leading to the attacker gaining the same access rights as the victim user.
3 main types:
1. Reflected XSS
   - The malicious script comes from the current HTTP request
3. Stored XSS
   - The malicious script comes from a website's database
   - The data is then included in later HTTP responses
5. DOM-based XSS
   - The vulnerability exists in client-side code, rather than server-side






------






# A) Totally Legit Certificate
**Objective**
- Install OWASP ZAP
- Generate a CA-certificate and install it on your browser
- Set ZAP as a proxy in your browser
- Set ZAP to capture also pictures
- Show that the requests appear in the ZAP interface


## Zed Attack Proxy
What is ZAP?
- Free, open-source penetration testing tool. Specifically for testing web applications
- [Source](<https://www.zaproxy.org/getting-started/>)

Install and start it on Kali:
```bash
$ sudo apt install zaproxy
$ zaproxy
```

ZAP [generates](<https://www.zaproxy.org/docs/desktop/addons/network/options/servercertificates/#generate>) a Root CA certificate for us, and we use it automatically by launching the browser through ZAP like so (2 birds 1 stone):

<img width="1635" height="1086" alt="2026-04-16-23:30:43" src="https://github.com/user-attachments/assets/13f686e1-11c5-4191-9d6d-67da36ef3af8" />

<img width="1909" height="493" alt="2026-04-16-23:31:24" src="https://github.com/user-attachments/assets/1de06f54-5c75-4952-bcac-65c9a44c3a96" />

Back on the ZAP interface we can see all the requests:

<img width="1623" height="1087" alt="2026-04-16-23:33:42" src="https://github.com/user-attachments/assets/ffb63332-bf7d-4359-af4a-e8a266832b88" />


In order to capture pictures, one must --> `Ctrl+Alt+O` --> type "display" in the search bar and select the option --> check the selection on "Process images in HTTP requests/responses".








