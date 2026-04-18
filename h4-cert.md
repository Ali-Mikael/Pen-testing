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
- Free, open-source penetration testing tool
- Specifically for testing web applications
- [Source](<https://www.zaproxy.org/getting-started/>)

Install and start it on Kali:
```bash
$ sudo apt install zaproxy
$ zaproxy
```
From this point on, there's two ways we can go about this. The first option i'm going to show now, and it's perferct if we just quickly want to test a website. 

ZAP actually [generates](<https://www.zaproxy.org/docs/desktop/addons/network/options/servercertificates/#generate>) a Root CA certificate for us. We automatically use it by launching the browser through ZAP like so:

<img width="1635" height="1086" alt="2026-04-16-23:30:43" src="https://github.com/user-attachments/assets/13f686e1-11c5-4191-9d6d-67da36ef3af8" />


In order to capture pictures, one must: --> `Ctrl+Alt+O` --> type "display" in the search bar --> select the option --> check the selection on "Process images in HTTP requests/responses".


Back on the ZAP interface we can see all the requests and replies:

<img width="1911" height="1129" alt="2026-04-17-17:19:36" src="https://github.com/user-attachments/assets/2b1233fe-4107-49f5-a330-79ed6626d978" />


The second method you're going to see next!




-------






# B) FoxyProxy
**Objective**
- Install FoxyProxy standard
- Add ZAP as a proxy to it

## Setting up
Open up extension/add-ons manager, search for "foxyproxy standard" and lastly `Add to Firefox`.

What is FoxyProxy?
- From my understanding it's a proxy manager, especially useful when you have multiple proxies


Because we want foxyproxy to manage the connection, we're going to do the second version of setting up ZAP. 

We add the CA certificate to our browser like so:
- In ZAP: `Ctrl+Alt+O` --> `Network` --> `Server Certificates` --> then click save and specify where to save it
- In Firefox: Open settings --> `Privacy & Security` --> `View Certificates` --> `Authorities` --> `Import` --> "Trust this CA to identify websites"
  - <img width="1213" height="702" alt="2026-04-17-18:42:11" src="https://github.com/user-attachments/assets/bc1a2e20-f990-40c7-b6f5-6875738f3b2e" />


Once we have that sorted, we'll add the ZAP config to foxyproxy
<img width="1753" height="612" alt="2026-04-17-18:47:46" src="https://github.com/user-attachments/assets/c806a8ba-fa2a-463b-adad-30644d276991" />




Now we can pin the extension to our tab bar and change settings on the go

<img width="481" height="592" alt="2026-04-17-19:09:33" src="https://github.com/user-attachments/assets/6b420978-3820-4800-b6c7-3d30acbd0af3" />
- If we enable ZAP, everything flows through it
- If we user "Proxy by Patterns", only portswigger goes through for now

Let's showcase quickly that it works.


### Exhibit A - ZAP is on
I browsed to `terokarvinen.com` and `archive.org` and we can see it in the output (not launching any spiders or attacks, just normal browsing)

<img width="1621" height="297" alt="2026-04-18-13:42:54" src="https://github.com/user-attachments/assets/72a5012a-3a8f-4f94-8c05-3e127e77dfcf" />


### Exhibit B - Proxy by pattern is on
First we browse to a random site and then the target that matches the pattern
Let the timestamps server as adequate proof that it works!

<img width="591" height="370" alt="2026-04-18-13:48:16" src="https://github.com/user-attachments/assets/055b1749-741d-44ef-8490-6d0846287481" />

<img width="568" height="379" alt="2026-04-18-13:48:37" src="https://github.com/user-attachments/assets/41ae50dc-fc69-4957-8573-772f3cea7b8f" />

<img width="799" height="172" alt="2026-04-18-13:49:26" src="https://github.com/user-attachments/assets/5a58f78f-fd84-4f80-99ab-e35873036619" />









--------








# C) [Reflected XSS](<https://portswigger.net/web-security/cross-site-scripting/reflected/lab-html-context-nothing-encoded>)
**Description**
- Lab contains a simple reflected cross-site scripting vulnerability in the search functionality

**Objective**
- Perform a cross-site scripting attack that calls the `alert` function


This one was pretty simple, we can exploit the vulnerability without even opening the developer tools.

We type into the input field
```html
<script>alert('gotcha')</script>
```
Press enter and we get the alert

<img width="769" height="279" alt="2026-04-18-14:49:00" src="https://github.com/user-attachments/assets/22695c9f-d16c-402a-9bbf-593b86f191d2" />








--------






# D) [Stored XSS](<https://portswigger.net/web-security/cross-site-scripting/stored/lab-html-context-nothing-encoded>)



