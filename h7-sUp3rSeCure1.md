# X) Read/Watch/Listen & Summarize
**Objective**
- A few bullet points per section is enough to summarize
- Add your own observation, question or idea

## X.1) [Cracking Passwords with Hashcat](<https://terokarvinen.com/2022/cracking-passwords-with-hashcat/>)
- Systems store hashes, not original passwords (or at least they should)
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



# A) Hashcat
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
- Install John the Ripper
- Crack the password for an example file

## Hidden content
John the Ripper is part of Kali's default arsenal as well:

<img width="1782" height="274" alt="2026-05-07-15:26:48" src="https://github.com/user-attachments/assets/5f1300c9-9b54-42ab-b1e3-0166fc0c2a7e" />

It does say it's the `jumbo` version so we'll go ahead with this one and see how it works out! We can always compile the latest version from source later on if need be!

Create the file to crack, compress it, and encrypt it. When you pass the `-e` flag you'll be prompted for the password:
```console
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




-----------



# C) File
**Objective**
- Create an encrypted file or search for one online
- Break the encryption
- (some other format than the one you already tried)

## PDF
I thought it'd be fun to crack a PDF, so I searched for ways to password protect PDFs and found a [blogpost](<https://www.baeldung.com/linux/file-pdf-set-password>) suggesting `pdftk`.

The tool can be installed like so:
```bash
$ sudo apt update && sudo apt install pdftk
```

And the PDF can be encrypted like so:
```bash
$ pdftk bash-cheatsheet.pdf output confidential.pdf user_pw PROMPT
```
<img width="1657" height="760" alt="2026-05-07-17:30:48" src="https://github.com/user-attachments/assets/b26c5acc-b0fb-4fdb-b208-a970960eeacb" />


Let's extract the hash from the file using `pdf2john` and save the output:
```console
┌──(㉿)
└─$ pdf2john confidential.pdf > pdf.hash
```

Then put Ol John to work:
```console
┌──(㉿)
└─$ john pdf.hash 
```
Well, it didn't actually work for some reason, it was huffing and puffing for about 10 minutes. I aborted the mission and changed the wordlist, when we went again it took less than a second!
```console
┌──(steve㉿flyingcarpet)-[~]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt pdf.hash 
Using default input encoding: UTF-8
Loaded 1 password hash (PDF [MD5 SHA2 RC4/AES 32/64])

password1234     (confidential.pdf)     
```

## Mission accomplished
Now we can open the file again:
```bash
$ xdg-open confidential.pdf
```
<img width="1186" height="364" alt="2026-05-07-18:39:33" src="https://github.com/user-attachments/assets/bf9a4493-7be4-405d-83ee-f3e519590ac6" />

<img width="913" height="761" alt="2026-05-07-18:40:53" src="https://github.com/user-attachments/assets/0e7ad1c5-95b9-4429-abf2-85816b353e9d" />



----------------




# D) Hash
**Objective**
- Create a password hash and break it
- You can search online on crate your own
- Use a different format than one you already tried


## S3cret1234!
So we did `SHA1` earlier, which is not really considered that secure anymore, so let's up the ante and use `SHA512` now!
```console
┌──(㉿)
└─$ echo -n "S3cret1234\!" | sha512sum | awk '{print $1}' | tee secret.txt
fd6372b5770b5b4c497ac62746dfb807fc4bf1d4ecc01c1787b0a8d6424af1d40c67db7f9e1f46a656ad759678817f2616eed20f2f509a5e68a6afda70769131
```
Yeeeeeeeeeahh that's a long hash right there, let's get cracking.

See if `hashid` recognizes it:
```console
┌──(㉿)
└─$ hashid secret.txt                              
--File 'secret.txt'--
Analyzing 'fd6372b5770b5b4c497ac62746dfb807fc4bf1d4ecc01c1787b0a8d6424af1d40c67db7f9e1f46a656ad759678817f2616eed20f2f509a5e68a6afda70769131'
[+] SHA-512 
[+] Whirlpool 
[+] Salsa10 
[+] Salsa20 
[+] SHA3-512 
[+] Skein-512 
[+] Skein-1024(512) 
--End of file 'secret.txt'--
```
There it is, first result!

We'll try with `-m 1700` first I guess:

<img width="1849" height="769" alt="2026-05-07-18:58:37" src="https://github.com/user-attachments/assets/379acc02-9921-4a18-8391-7fc794432c33" />

```console
┌──(㉿)
└─$ hashcat -m 1700 secret.txt /usr/share/wordlists/rockyou.txt -o cracked -O
hashcat (v7.1.2) starting
```
But it returned empty handed... Let's fix that by adding a new entry to rockyou:
```console
┌──(steve㉿flyingcarpet)-[~/box/secret]
└─$ echo -n "S3cret1234\!" | sudo tee /usr/share/wordlists/rockyou.txt
[sudo] password for steve: 
S3cret1234!
```
And then run hashcat again:
```console
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 1700 (SHA2-512)
Hash.Target......: fd6372b5770b5b4c497ac62746dfb807fc4bf1d4ecc01c1787b...769131
Kernel.Feature...: Optimized Kernel (password length 0-31 bytes)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Candidates.#01...: S3cret1234! -> S3cret1234!


┌──(㉿)
└─$ cat cracked
fd6372b5770b5b4c497ac62746dfb807fc4bf1d4ecc01c1787b0a8d6424af1d40c67db7f9e1f46a656ad759678817f2616eed20f2f509a5e68a6afda70769131:S3cret1234!
```
And there you go, password recovered.


I saw `RSA private key` when going through all the available formats for hashcat. I wanted to try it, so I created a new key:
```console
┌──(㉿)
└─$ ssh-keygen -t rsa -N 'Sup3rsecret1234!'
Generating public/private rsa key pair.
Enter file in which to save the key (/home/steve/.ssh/id_rsa): ./my_key              
Your identification has been saved in ./my_key
Your public key has been saved in ./my_key.pub
```
We can verify it's password protected:

<img width="1234" height="162" alt="2026-05-07-19:31:48" src="https://github.com/user-attachments/assets/d8ac603a-af4e-4397-8ae1-7099c11d5f5f" />

The `-y` flag is used to read a private key and print the public key to `stdout` (source `ssh-keygen` man pages).

Let's extract the password hash:
```console
┌──(㉿)
└─$ ssh2john my_key > my_key.hash
```
And put `Hashcat` to work:
```console
┌──(㉿)
└─$ hashcat -m 22911 my_key.hash /usr/share/wordlists/rockyou.txt -o keySecret   
hashcat (v7.1.2) starting

Hashfile 'my_key.hash' on line 1 (my_key...f0095940aad45d030aa605e0a$24$486): Token length exception

* Token length exception: 1/1 hashes
  This error happens if the wrong hash type is specified, if the hashes are
  malformed, or if input is otherwise not as expected (for example, if the
  --username or --dynamic-x option is used but no username or dynamic-tag is present)
```
But we get an error..

I removed `my_key:` from the beginning of the file, but that didn't help. I generated the hash again and used the `--username` flag with hashcat but that didn't help either. I didn't know how to proceed, but then I thought: why make things complicated when they don't need to be. If John can do it let him finish the job!!
```console
┌──(㉿)
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt my_key.hash 

Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 2 for all loaded hashes
Cost 2 (iteration count) is 24 for all loaded hashes

0g 0:00:00:00 DONE S3cret1234!  # <-- Here we go
Session completed.
```
John's a good guy, he gets it! Let's take the win and move on. 🙏




---------------


# E) Dictionary
**Objective**
- Demonstrate how you make a dictionary for `john` or `hashcat`


--------------


# F) Hash Rules
**Objective**
- Showcase Hashcat rules




-------------




# G) Flag Preparation
**Objective**
- Prepare your machine for the grand finalè A.K.A Capture The Flag
- If you already have a working Kali VM on a normal (amd64) PC, you don't really have to do anything in this section


## Kali
<img width="1826" height="490" alt="2026-05-07-20:17:00" src="https://github.com/user-attachments/assets/ef6aceec-f331-4b3d-b831-3b42cea9874a" />


I do have to give my attacker some more storage!

## Resize Workflow
1. Shut down the VM.
2. Add some juice to the disk image:
```bash
[ ~ ] ❯❯ sudo qemu-img resize /var/lib/libvirt/images/kali.qcow2 +20G
Image resized.
```
3. Power the VM back on
4. `lsblk` to confirm that the disk grew:
```bash
┌──(㉿)
└─$ lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sr0     11:0    1 1024M  0 rom  
vda    254:0    0   60G  0 disk 
├─vda1 254:1    0 37.9G  0 part /
├─vda2 254:2    0    1K  0 part 
└─vda5 254:5    0  2.1G  0 part [SWAP]
```
It did! Now we can make it available to the system.

5. Fire up the partition manipulation tool
```bash
$ sudo parted
```
7. Type `print` to confirm layout
```bash
Number  Start   End     Size    Type      File system     Flags
 1      1049kB  40.7GB  40.7GB  primary   ext4            boot
 2      40.7GB  42.9GB  2239MB  extended                  lba
 5      40.7GB  42.9GB  2239MB  logical   linux-swap(v1)  swap
```
> [!NOTE]
> The order is problematic, as the swap partition comes after the root, solution:
> ```bash
> $ sudo swapoff -a
> # Then delete the swap partition (we'll just create a swapfile later)
> ```

9. `resizepart 1` --> `100%` (Use 100% of available space)
```console
(parted) resizepart 1                                                     
Warning: Partition /dev/vda1 is being used. Are you sure you want to continue?
Yes/No? yes                                                               
End?  [40.7GB]? 100%
```
10. Resize the filesystem
```console
┌──(㉿)
└─$ sudo resize2fs /dev/vda1                          
resize2fs 1.47.4 (6-Mar-2025)
Filesystem at /dev/vda1 is mounted on /; on-line resizing required
old_desc_blocks = 5, new_desc_blocks = 8
The filesystem on /dev/vda1 is now 15728384 (4k) blocks long.
```
11. Done!
```console
vda    254:0    0   60G  0 disk 
└─vda1 254:1    0   60G  0 part /
```



