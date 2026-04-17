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


In order to capture pictures, one must: --> `Ctrl+Alt+O` --> type "display" in the search bar --> select the option --> check the selection on "Process images in HTTP requests/responses".


Back on the ZAP interface we can see all the requests and replies:

<img width="1911" height="1129" alt="2026-04-17-17:19:36" src="https://github.com/user-attachments/assets/2b1233fe-4107-49f5-a330-79ed6626d978" />





-------






# B) FoxyProxy
**Objective**
- Install FoxyProxy standard
- Add ZAP as a proxy to it

## Proxy
Open up extension/add-ons manager, search for "foxyproxy standard" and lastly `Add to Firefox`.

What is FoxyProxy?

Their words:

<img width="876" height="726" alt="2026-04-17-17:27:31" src="https://github.com/user-attachments/assets/bf2de97e-9412-4bde-bb9a-acd182c4273c" />


Because we want foxyproxy to manage the connections it complicates things a bit, so we're going to have to add the ZAP certificate to our browser
- `Ctrl+Alt+O` --> `Network` --> `Server Certificates` --> then click save and specify location
- Open up firefox settings --> Privacy & Security --> View Certificates --> Authorities --> Import --> "Trust this CA to identify websites"
  - <img width="1213" height="702" alt="2026-04-17-18:42:11" src="https://github.com/user-attachments/assets/bc1a2e20-f990-40c7-b6f5-6875738f3b2e" />


Once we have that sorted, we'll go to foxyproxy settings and add ZAP
- Title: ZAP
- Hostname: 127.0.0.1 (localhost)
- Port: 8080
- Pattern:
  - Include: All: Wildcard: `*portswigger.net*`



Now when we pin the extension to our tab bar, we can quickly change between settings
- If we enable ZAP, everything flows through it
- If we user "Proxy by Patterns", only portswigger goes through for now
- <img width="481" height="592" alt="2026-04-17-19:09:33" src="https://github.com/user-attachments/assets/6b420978-3820-4800-b6c7-3d30acbd0af3" />



--------





# C) [Reflected XSS](<https://portswigger.net/web-security/cross-site-scripting/reflected/lab-html-context-nothing-encoded>)
Objective
- Do nothing and wait


