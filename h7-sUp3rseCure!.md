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
Hashcat is part of Kali's default arsenal. If for some reason you're using Kali and don't have it, do:
```bash
$ sudo apt update && sudo apt install hashcat -y
```

Let's create a hash we can try to crack like so:
```bash
┌──(㉿)
└─$ echo -n password | sha1sum | tee secret.txt
5baa61e4c9b93f3f0682250b6cf8331b7ee68fd8  -
```
The command in the middle of the pipeline creates a hash of the word we echoed using the `SHA1` algorithm, the last part of the command prints it to `stdout` and saves it to a file called `secret.txt`

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
- `-O`: Enable optimized kernel (don't know what it actually does under the hood but hashcat suggested it so why not)

The last flag `--quiet` suppresses the verbose output, we're only here for the password:
```console
┌──(㉿)
└─$ cat cracked 
5baa61e4c9b93f3f0682250b6cf8331b7ee68fd8:password
```

