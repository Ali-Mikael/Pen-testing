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
Hashcat is part of Kali's default arsenal. If you're using Kali and for some reason don't have it:
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
```bash
┌──(㉿)
└─$ hashcat -m 100 secret.txt /usr/share/wordlists/rockyou.txt -o cracked -O --quiet
```
- `-m`: Hash type (100 = SHA1)
- `secret.txt`: Input file
- `-o cracked`: Output file
- `-O`: Enable optimized kernel (don't know what it actually does under the hood but hashcat suggested it so why not 😂)

The last flag `--quiet` suppresses the verbose output, we're only here for the password!
```bash
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
```bash
┌──(㉿)
└─$ zip -e confidential.zip confidential.txt 
Enter password: 
Verify password: 
  adding: confidential.txt (deflated 15%)
```


Now we have a file to play with. We extract the hash using `zip2john` and store it in a new file:
```bash
┌──(㉿)
└─zip2john confidential.zip > confidential.hash    
```

Using the hash file, we set John the Ripper out on a mission to crack it:
```bash
┌──(㉿)
└─$ john confidential.hash

# Some output redacted for clarity #
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
I thought it'd be fun to crack a PDF, so I searched for ways to encrypt PDFs and found a [blogpost](<https://www.baeldung.com/linux/file-pdf-set-password>) suggesting `pdftk`.

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
```bash
┌──(㉿)
└─$ pdf2john confidential.pdf > pdf.hash
```

Then put Ol John to work:
```bash
┌──(㉿)
└─$ john pdf.hash 
```
Well, it didn't actually work for some reason, it was huffing and puffing for about 10 minutes. I aborted the mission and changed the wordlist. When we went again it took less than a second!
```bash
┌──(㉿)
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt pdf.hash

# Some output redacted for clarity #
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
- You can search online or create your own
- Use a different format than one you already tried


## S3cret1234!
We did `SHA1` earlier, which is not really considered that secure anymore, so let's up the ante and try `SHA512` now!
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
It does, first result!

We'll try with `-m 1700` first I guess:

<img width="1849" height="769" alt="2026-05-07-18:58:37" src="https://github.com/user-attachments/assets/379acc02-9921-4a18-8391-7fc794432c33" />

```bash
┌──(㉿)
└─$ hashcat -m 1700 secret.txt /usr/share/wordlists/rockyou.txt -o cracked -O
hashcat (v7.1.2) starting
```
But it returns empty handed... Let's fix that by adding a new entry to rockyou:
```bash
┌──(steve㉿flyingcarpet)-[~/box/secret]
└─$ echo -n "S3cret1234\!" | sudo tee /usr/share/wordlists/rockyou.txt 
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


## SSH private keys?
I saw `RSA private key` when going through all the available formats for hashcat. I wanted to try it, so I created a new password protected key:
```bash
┌──(㉿)
└─$ ssh-keygen -t rsa -N 'Sup3rsecret1234!'

# Some output redacted #
Generating public/private rsa key pair.
Enter file in which to save the key: ./my_key
```
Verify it works as intended by providing the wrong password:

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

I removed `my_key:` from the beginning of the file ...but that didn't help. I then generated the hash again and used the `--username` flag with `hashcat`, ...but that didn't help either. I didn't know how to proceed, but then I thought: why make things complicated when they don't need to be. If John can do it let him finish the job!!
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
- Demonstrate how to make a dictionary for `john` or `hashcat`

## BYOD (Bring Your Own Dictionary)
I searched for ways to create your own wordlists and stumbled upon the tool `cupp`, it can be installed on Kali:
```
$ sudo apt update && sudo apt install cupp -y
```
The description from man pages: _"Generate dictionaries for attacks from personal data"_. This intrigued me, as you can give it personal information on someone, and it will then go ahead and create you a word list based on the info you provided.

You can provide all the information interactively:
```
┌──(㉿)
└─$ cupp -i
```
<img width="1800" height="800" alt="2026-05-10-13:47:39" src="https://github.com/user-attachments/assets/00880b78-f35d-47b4-98b9-f99f814baee0" />

```console
   cupp.py!                 # Common
      \                     # User
       \   ,__,             # Passwords
        \  (oo)____         # Profiler
           (__)    )\   
              ||--|| *      [ Muris Kurgas | j0rgan@remote-exploit.org ]
                            [ Mebus | https://github.com/Mebus/]


[+] Insert the information about the victim to make a dictionary
[+] If you don't know all the info, just hit enter when asked! ;)

> First Name: Marcus
> Surname: Aurelius
> Nickname: Marky
> Birthdate (DDMMYYYY): 26040121


> Partners) name: Annia Galeria Faustina
> Partners) nickname: Fauzia
> Partners) birthdate (DDMMYYYY): 01010130


> Child's name: Lucilla
> Child's nickname: Lucy
> Child's birthdate (DDMMYYYY): 01010130


> Pet's name: Doggy
> Company name: Roman empire


> Do you want to add some key words about the victim? Y/[N]: y
> Please enter the words, separated by comma. [i.e. hacker,juice,black], spaces will be removed: meditations,thinker,philosophy,empire,rome,romanEmpire,stoicism,logic,reason,physics,god,Domitia,Calvilla 
> Do you want to add special chars at the end of words? Y/[N]: y
> Do you want to add some random numbers at the end of words? Y/[N]:Y
> Leet mode? (i.e. leet = 1337) Y/[N]: Y

[+] Now making a dictionary...
[+] Sorting list and removing duplicates...
[+] Saving dictionary to marcus.txt, counting 45074 words.
[+] Now load your pistolero with marcus.txt and shoot! Good luck!
```
We now have a dictionary of `45073` words, here's a snippet of that file (words chosen arbitrarily):
```console
rome_61214
rome_621
stoicism
stoicism!
stoicism!!$
stoicism!!*
stoicism!!@
3mp1r3_2009
3mp1r3_2010
3mp1r3_2011
3mp1r3_21
3mp1r3_2104
```
Marcus not really being a computer wizard, I think we would crack his password relatively quickly with this list!

## Further modding
In the configuration file `/etc/cupp.cfg` you can control how the words are generated, for example, changing `leet` mode:
```
[leet]
a=4
i=1
e=3
t=7
o=0
s=5
g=9
z=2
```
This is an actual entry in the file, if you've seen the victim substituting `L` as `/` for some reason, you can go ahead and add a new entry for it!

Here's 2 other examples for specifying special characters and random numbers -->
```console
[specialchars]
chars=!,@,'#',$,%%,&,*

[nums]
from=0
to=100
```


## Automation
I wanted to build on this list by using `cewl` ([Custom Word List generator](<https://github.com/digininja/CeWL>)). It intrigued me for it's ability to crawl URLs you give it, at a specified depth, and return with a list of words. Let's try it out by giving it the wikipedia page of [Marcus Aurelius](<https://en.wikipedia.org/wiki/Marcus_Aurelius>).

If you give the `-h` flag to `cewl`, it will give you a short but comprehensive list on all the options (funny enough it was more comprehensive than the man pages this time).

I used the long form of the flags so it would be easy to understand the options used, here's the command:
```bash
┌──(㉿)
└─$ cewl --min_word_length 6 -a --meta_file marcusMeta.txt --max_word_length 16 --write marcusAutomatic.txt https://en.wikipedia.org/wiki/Marcus_Aurelius
```
The default depth the spider will crawl is 2 links from the main site.

It was going on for about 11 minutes, I thought something was wrong so I `ctrl+c`'d the operation
```console
^CHold on, stopping here ...
```
Apparently it was still busy working, but luckily it did save all the work. The `meta` file which was supposed to store all meta information was empty, but the wordlist was populated:
```bash
┌──(㉿)
└─$ wc -l marcusAutomatic.txt 
34122 marcusAutomatic.txt
```
All of `34122` words! Here's a snippet of arbitrarily chosen words from the list:
```console
Marcus
Wikipedia
Aurelius
Empire
Special
Template
wikipedia
identifier
Category
Birley
Hadrian
Lucius
btitle
Abookrft
aulast
aufirst
University
Antoninus
philosophy
BookSources
emperor
History
bookrft
Faustina
dynasty
Ancient
template
Trajan
International
Carrera
Romanism
philosophizing
constituents
hollow
lookout
columnae
expressive
clumsier
worsen
expecting
Pitholaus
hemmed
Onesicrates
plight
fatigue
scorched
interposition
enchantments
performances
upwards
```

## Joining forces
I started thinking, could we combine the `cupp` permutation functionality with the vanilla wordlist generated by `cewl`? Turns out we can! 

Let's spice things up a bit and pass the wordlist to `cupp` like so:
```
┌──(㉿)
└─$ cupp -w marcusAutomatic.txt 

   cupp.py!            
      \                
       \   ,__,       
        \  (oo)____    
           (__)    )\   
              ||--|| *      


> Do you want to concatenate all words from wordlist? Y/[N]: n
> Do you want to add special chars at the end of words? Y/[N]: Y
> Do you want to add some random numbers at the end of words? Y/[N]:Y
> Leet mode? (i.e. leet = 1337) Y/[N]: Y

[+] Now making a dictionary...
[+] Sorting list and removing duplicates...
[+] Saving dictionary to marcusAutomatic.txt.cupp.txt, counting 17170464 words.
[+] Now load your pistolero with marcusAutomatic.txt.cupp.txt and shoot! Good luck!
```
Our improved dictionary:
```
┌──(㉿)
└─$ mv marcusAutomatic.txt.cupp.txt marcusEnhanced2p0.txt
                                                                                                                              
┌──(㉿)
└─$ wc -l marcusEnhanced2p0.txt                                       
17170463 marcusEnhanced2p0.txt
```
That's a pretty hefty dictionary right there.. 😆 If we're not cracking the password with this one, we were never gonna get it to begin with.

Here's a quick snippet:
```
Cl4r3nc3
Cl4r3nc3!
Cl4r3nc3!!
Cl4r3nc3!!!
Cl4r3nc3!$!
Cl4r3nc3!*
Cl4r3nc3!@%
Cl4r3nc3$$*
Cl4r3nc3$$@
Cl4r3nc3%*$
Cl4r3nc3%@
Cl4r3nc3%@!
Cl4r3nc3&!!
Cl4r3nc3&!$
Cl4r3nc3&&%
```







--------------








# F) Hash Rules
**Objective**
- Showcase Hashcat rules

## Rule based attack
The man pages made me none the wiser on the use of rules in hashcat, so I navigated to the hashcat [wiki](<https://hashcat.net/wiki/doku.php?id=rule_based_attack>). 
I read through the wiki and got some idea on how to create rules. The wiki guided me further to check out the `rules/` folder, so I did
```bash
$ locate hashcat | grep -i rule
```
And found my way to `/usr/share/hashcat/rules`:
```bash
┌──(㉿)-[/usr/share/hashcat/rules]
└─$ ls
best66.rule                  leetspeak.rule                                  T0XlC.rule
combinator.rule              oscommerce.rule                                 T0XlCv2.rule
d3ad0ne.rule                 rockyou-30000.rule                              toggles1.rule
dive.rule                    specific.rule                                   toggles2.rule
generated2.rule              stacking58.rule                                 toggles3.rule
generated.rule               T0XlC_3_rule.rule                               toggles4.rule
hybrid                       T0XlC-insert_00-99_1950-2050_toprules_0_F.rule  toggles5.rule
Incisive-leetspeak.rule      T0XlC_insert_HTML_entities_0_Z.rule             top10_2025.rule
InsidePro-HashManager.rule   T0XlC-insert_space_and_special_0_F.rule         unix-ninja-leetspeak.rule
InsidePro-PasswordsPro.rule  T0XlC-insert_top_100_passwords_1_G.rule
```
There were a lot of very illustrative examples I took inspiration from when creating my own ruleset.


Only by trying stuff out do we actually learn, so let's start by creating the basis for this attack:
```bash
# Create the password hash
┌──(㉿)
└─$ echo -n "SuperS3cretPassword123" | sha256sum | awk '{print $1}' | tee secret.txt
ca0a0a9af0ba1c6b6ec5c3adb855016c3f8450ef91bcca6135bb50c5f76dbf1f

# Create the wordlist
┌──(㉿)
└─$ echo "password\nsecret\nsecretpassword\nsupersecret\nsupersecretpassword" > wlist.txt
                                                                                                                              
┌──(㉿)
└─$ cat wlist.txt 
password
secret
secretpassword
supersecret
supersecretpassword                                                    
```

Now for the rule creation, I created a `rules` file and populated it with the following:
```console
:
u
c
t
T0

$0
$1
$2
$3
$4
$5
$6
$7
$8
$9
$1$2
$1$2$3
$1$2$3$4

so0
si1
se3
sa@
so0 si1 se3 sa@
so0 si1 se3 $1$2$3
so0 si1 se3 $1$2$3$4
so0 si1 se3 $1$2$3$4$!

## The winner ->
T0 T5 TB o63 $1$2$3
```
We're including some basics like `case toggle`, `number appending` and `leet`. Check out the rule section in the [wiki](<https://hashcat.net/wiki/doku.php?id=rule_based_attack>) if you want to understand this better!

## Executing the attack
Identify the hash
```bash
┌──(㉿)
└─$ hashid secret.txt 
--File 'secret.txt'--
Analyzing 'ca0a0a9af0ba1c6b6ec5c3adb855016c3f8450ef91bcca6135bb50c5f76dbf1f'
[+] Snefru-256 
[+] SHA-256 
[+] RIPEMD-256 
[+] Haval-256 
[+] GOST R 34.11-94 
[+] GOST CryptoPro S-Box 
[+] SHA3-256 
[+] Skein-256 
[+] Skein-512(256) 
--End of file 'secret.txt'--     
```
`SHA256` is the most likely candidate.

Use the rules to crack the password:
```bash
┌──(㉿)
└─$ hashcat -a 0 -m 1400 -r rules secret.txt wlist.txt -o cracked -O

#--Some output redacted!--#

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 27
                                             
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 1400 (SHA2-256)
Hash.Target......: ca0a0a9af0ba1c6b6ec5c3adb855016c3f8450ef91bcca6135b...6dbf1f

Kernel.Feature...: Optimized Kernel (password length 0-31 bytes)
Guess.Base.......: File (wlist.txt)
Guess.Mod........: Rules (rules)

Candidate.Engine.: Device Generator
Candidates.#01...: password123 -> SuperS3cretPassword123


┌──(㉿)
└─$ cat cracked          
ca0a0a9af0ba1c6b6ec5c3adb855016c3f8450ef91bcca6135bb50c5f76dbf1f:SuperS3cretPassword123
```
**Explained**
- Our rule file had a bunch of rules, but the one that cracked the password was the last line
```
T0 T5 TB o63 $1$2$3
```
- `TX`: Toggle case at position X
  - Example: `T0 T5` transforms `waddup` to `WadduP`
- `oXY`: Overwrite char at position X with Y
  - Example: `o1X` transforms `waddup` to `wXddup`
- `$X`: Append X
  - Example: `$!` transforms `waddup` to `waddup!`


-------------




# G) Flag Preparation
**Objective**
- Prepare your machine for the grand finalè A.K.A Capture The Flag
- If you already have a working Kali VM on a normal (amd64) PC, you don't really have to do anything in this section


## Kali
<img width="1826" height="490" alt="2026-05-07-20:17:00" src="https://github.com/user-attachments/assets/ef6aceec-f331-4b3d-b831-3b42cea9874a" />


I do have to give my attacker some room to grow!

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

5. Fire up the `partition manipulation tool`
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
> The order is problematic, as the swap partition comes **after** the root. We can't really use the `100%` paramater to make the partition use all the newly made available space :/
>
> My solution:
> ```bash
> $ sudo swapoff -a
> # Then delete the swap partition (we'll just create a swapfile later)
> (parted) rm 2
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



