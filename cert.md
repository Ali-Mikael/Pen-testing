# X) Read/Watch/Listen & Summarize

**Objective**
- A few bullet points per section is enough to summarize
- Add your own observation, question or idea


## X.1) [OWASP A01:2021](<https://owasp.org/Top10/2021/A01_2021-Broken_Access_Control/index.html>) - Broken Access Control
**Common AC vulnerabilities:**
- Principle of least privilege violation
- Bypassing AC checks with URL tampering
- Insecure direct object references
- Abusing JWT
- CORS misconfiguration

## X.2) PortSwigger Academy
[IDOR](<https://portswigger.net/web-security/access-control/idor>)
- a.k.a Insecure Direct Object References
- A vulnerability that arises from accessing objects directly with user-supplied input
- Most commonly associated with horizontal privilege escalation
- For example:
  - A website retrieves information from the database using the following URL
    - `https://bank/user123?statement=123`
  - One could simply change the customer string and ID for bank statement to retreive arbitrary statements
 


[Path Traversal](<https://portswigger.net/web-security/file-path-traversal>)
- This type of vulnerability arises when a web server retrieves _files_ based on a _filename_ **appended** to a **base file path**
- Say a web server is serving content from `/home/webUser/content/*`, this is the base file path!
- If no controls are in place, one could read files from the home directory by tampering with the URL and replacing allowed file `file1` with `../secretFile.txt`
- Simply put: `/home/webUser/content/file1` now becomes `/home/webUser/secretFile`












