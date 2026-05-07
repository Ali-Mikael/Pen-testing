# X) Read/Watch/Listen & Summarize
**Objective**
- A few bullet points per section is enough to summarize
- Add your own observation, question or idea

## X.1) [Cracking Passwords with Hashcat](<https://terokarvinen.com/2022/cracking-passwords-with-hashcat/>)
- Systems store hashes, not original passwords
- Hashing is a one way function, you can't turn it back to a password
- BUT, you can make a computer try every word in the dictionary and compare the hashes
- Get going:
  - Download `hashcat`
  - Aquire wordlist of passwords to try
  - Identify hash type by using `hashid` for example
  - Then give hashcat the type, the hash and the output file and fire away
 
## X.2) [Crack File Password With John](<https://terokarvinen.com/2023/crack-file-password-with-john/>)
- File formats might support password protected encryption, John the Ripper on the other hand might be able to crack these passwords with a dictionary attack
- Get the latest version of John source code without the fluff by using the `--depth=1` flag in your git `git clone` command
- Compile the project
- Why from source?
  - It's the jumbo version, a.k.a more features!
- Cracking the ZIP password happens in 2 stages:
  - Extracting the hash
  - Setting John up for a dictionary attack against the hash




------------------------



# Hashcat
**Objective**
- Install Hashcat
- Test it by breaking an example password

## Password1234
Hashcat is part of Kali's default arsenal. If for some reason you're using Kali and don't have it:
```bash
$ sudo apt update && sudo apt install hashcat -y
```

Let's create a hash we can try to crack like so:
```bash
┌──(㉿)
└─$ echo -n password | sha1sum | tee secret.txt
5baa61e4c9b93f3f0682250b6cf8331b7ee68fd8  -
```
`-n` flag omits trailing newline, command in the middle of the pipeline computes the hash using the `SHA1` algorithm. Last part of the pipeline prints the result to `stdout` and saves it to a file called `secret.txt`.

> [!NOTE]
> It's good to note that the hyphen (-) at the end of the output is going to cause problems when trying to crack the hash.
>
> We can either manually remove it or:
> ```bash
> $ echo -n password | sha1sum | awk '{print $1}' | tee secret.txt
> ```

We recall the options for Hashcat by skimming its manual pages and end up with the following command:
```console
┌──(㉿)
└─$ hashcat -m 100 secret.txt /usr/share/wordlists/rockyou.txt -o cracked -O --quiet
```
- `-m`: Hash type (100 = SHA1)
- `secret.txt`: Input file
- `-o cracked`: Output file
- `-O`: Enable optimized kernel (don't know what it actually does under the hood but hashcat suggested it so why not 😂)

The last flag `--quiet` suppresses the verbose output, we're only here for the password:
```console
┌──(㉿)
└─$ cat cracked 
5baa61e4c9b93f3f0682250b6cf8331b7ee68fd8:password
```




-----------





# B) John
**Objective**
- Intall John the Ripper
- Crack the password for an example file

## Hidden content
John the Ripper is part of Kali's default arsenal as well:

<img width="1782" height="274" alt="2026-05-07-15:26:48" src="https://github.com/user-attachments/assets/5f1300c9-9b54-42ab-b1e3-0166fc0c2a7e" />

It does say it's the `jumbo` version so we'll go ahead with this one and see how it works out! We can always compile the latest version from source later on!

Create the file to crack and compress it using encryption, when you pass the `-e` flag you'll be prompted for the password:
```bash
┌──(㉿)
└─$ zip -e confidential.zip confidential.txt 
Enter password: 
Verify password: 
  adding: confidential.txt (deflated 15%)
```


Now we have a file to play with. We extract the hash using `zip2john` and direct the output to a new file:
```bash
┌──(㉿)
└─zip2john confidential.zip > confidential.hash    
ver 2.0 efh 5455 efh 7875 confidential.zip/confidential.txt PKZIP Encr: TS_chk, cmplen=104, decmplen=108, crc=5ECEF8EE ts=7C6F cs=7c6f type=8
```

Using the hash file, we set John the Ripper out on a mission to crack it:
```bash
┌──(㉿)
└─$ john confidential.hash
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Proceeding with wordlist:/usr/share/john/password.lst
1234             (confidential.zip/confidential.txt)     # <--
Session completed. 
```
The command is very simple, `john` followed by the hash-file to crack. It then proceeds to extract the password = `1234`, which we use to `unzip` the file:

<img width="1412" height="373" alt="2026-05-07-15:57:56" src="https://github.com/user-attachments/assets/2281627f-919b-46c7-a4ff-f66b622824c3" />



# C) File
**Objective**
- Create an encrypted file or search for one online
- Break the encryption
- (some other format than the one you already tried)

## File1



