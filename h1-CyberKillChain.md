# X) Read/watch/listen/summarize

## X.1 Darknet Diaries - [EP 106: Greg](<https://darknetdiaries.com/episode/160/>)
Greg Linares A.K.A Laughing Mantis
- Found exploits and wrote malware in MIDDLESCHOOL
- Actually used the malware he wrote on criminals
- In high school: Why bother attending the boring classes when you can just hack the system and give yourself perfect attendance and passing grades
  - Well, it works until it doesn't, and he ended up getting arrested and thrown into juvie
  - This made him the youngest hacker to be arrested in Arizona at the time
- Greg was blessed, as the police fumbled the evidence and the case got dropped
- This didn't stop him from being kicked out of his home tho, so he ended up living in a halfway home until 18
- He did some music and toured the world for a bit
- After his music career, he got a job coding backend at this massage parlor, while creating exploits and publishing them online on the side
  - That's how he got scouted by a cyber security company
  - At the company he worked with a team finding zero days
  - The work entailed a lot of reverse engineering and modifying/cracking binaries
- He found a remote code execution vulnerability from the latest Microsoft word version
  - Make the program crash --> take control of a pointer --> inject shell code into memory during the crash ==> Arbitrary remote code execution
  - Only problem was that it could only be done with a debugger attached to the program
  - Because he didn't verify it all the way, and the press release was already out, his career was now on the line
  - He got one chance to redeem himself, and after a 3 day bender found a vulnerability in the microsoft visio program
  - Face of the company saved and greg got to keep his job
- Greg went on to have a pretty succesful red-teaming career
- Big note from Greg: Never overlook layer 2 (data-link) attacks in penetration testing / red-teaming. That's that bread and butter!



------



## X.2 [Intelligence-Driven Computer Network Defence Informed by Analysis of Adversary Campaigns and Intrusion Kill Chains](<https://lockheedmartin.com/content/dam/lockheed-martin/rms/documents/cyber/LM-White-Paper-Intel-Driven-Defense.pdf>)

### Abstract
- APT / Advanced Persistent Threat = A class of threats representing well-resourced and trained adversaries conducting multi-year intrusion campaigns
- Network defence techniques leveraging knowledge about these adversaries can create an intelligence feedback loop, coupled with a kill chain model to describe phases of intrusion, among other things, form the basis of intelligence-driven CND (Computer Network Defense)

### Chapter 3.2 - Intrusion Kill Chain
- Kill chain => a systematic process to **target** and **engage** an adversary to _create desired effects_
- U.S military doctrine defines:
  - **Find** (targets)
  - **Fix** (their location)
  - **Track** (and observe)
  - **Target** (with suitable weapon/asset)
  - **Engage** (adversary)
  - **Asses** (effects)
- The paper expands on this concept by building a new kill chain model, one specifically for intrusions
- Defined as:
  - **Reconnaissance**
  - **Weaponization** (weaponizing a deliverable such as documents and PDFs)
  - **Delivery**
  - **Exploitation**
  - **Installation** (remote access trojan or backdoor --> creating a persistent foothold)
  - **Command & Control**
  - **Actions on Objectives** (only after all aforementioned steps are completed can the intruder take actions to achieve their original objectives)



-------




# X.3 - [The Art of Hacking](<https://learning.oreilly.com/videos/the-art-of/9780135767849/9780135767849-SPTT_04_00>)

### Chapter 4.1 - Understanding Active Reconnaissance
- Active recon entails sending traffic into the target network. Port scans, vulnerability scans etc
  - For contrast, with _passive reconnaissance_ **no traffic** is directly sent to the target environment
- While doing the recon, the traffic being sent progressively gets louder, this happens for 2 reasons: The scans get more aggressive, and we want to know if somebody is actually looking/monitoring.


### Chapter 4.2 - Exploring Active Reconnaissance Methodologies from an Ethical Hacker Perspective
- Due to time constraints, having a solid methology is a must!
- Reconnaissance is key in sifting through large amounts of data in order to pinpoint worthy targets to attack
- Example:
    1. Port Scanning
       - Confirm passive recon intel is correct, plus find additional open ports
    3. Web Service Review
       - What services/applications are in play?
       - Which ones do we want to focus on?
    5. Vulnerability Scanning
       - Done last, as this one creates a lot of noice and most likely to set off alarms


### Chapter 4.3 - Surveying Essential Tools for Active Reconnaissance
**Port Scanning**
- **Nmap**
  - The most versatile and stable
  - Using the `-Pn` flag is helpful in bypassing firewalls when scanning from outside a network
  - You might want to scan ports randomly to avoid detection
- **Masscan**
  - The fastest port scanner
- **Udpprotoscanner**
  - Fast UDP port scanner

**Web Service Review**
- **EyeWitness**
  - When provided with a target list, EyeWitness will go through it, mapping out high value targets
  - It will write you a report consisting of details about each website along with screenshots


### Chapter 4.4 - Surveying Essential Tools fo Active Reconnaissance: Network and Web Vulnerability Scanners
**Network Vulnerability Scanners**
- OpenVAS (free & open source)
- Nessus, Nexpose, Qualys
- Nmap (limited vuln scanning)
  - Uses the NSE (Nmap Scripting Engine)

**Web Vulnerability Scanners**
- Nikto
- WPScan (specific to WordPress websites)
- SQLMap (SQLi etc..)
- Burp Suite 
- Zed Attack Proxy (similar to burp but free)




# X.4 - [KK:2003:36](<https://www.finlex.fi/fi/oikeuskaytanto/korkein-oikeus/ennakkopaatokset/2003/36>)
A teenager had port scanned servers belonging to a bank and ended up having to pay a pretty hefty fine for it.

What can we learn from this?
- Always be careful even when doing ethical hacking / penetration testing so that you don't touch stuff you don't have permission to, even by accident!

------




# A) Install Kali
Kali Linux is already present:

<img width="1489" height="475" alt="2026-03-24-22:25:36" src="https://github.com/user-attachments/assets/78653900-1836-4413-b556-d607e199f85b" />


It's pretty easy to setup, here's how I did it:
- Download the `kali linux netinstaller ISO image` 
- Create a new VM from the ISO
- Follow the steps in the installer and configure the new system
- Once everything is done, remove the ISO and reboot the machine


I'm using **KVM/QEMU** for virtualization/emulation and then manage the whole shabang using **libvirt**.




------




# B) Kali goes dark
**Objective**
- Disconnect kali-vm from the network
- Prove with tests that it cannot reach the internet

Issue the command
```bash
$ nmcli networking off
```

We confirm that it works by sending a few ICMP echo requests to Cloudflare's DNS resolver before and after:
- <img width="860" height="460" alt="2026-03-24-18:23:30" src="https://github.com/user-attachments/assets/ae6a51ee-ad19-42f8-9207-f25627c9807e" />



------



# C) Scan
**Objective**
- Scan the 1000 most common ports on your machine
- Explain cmd params
- Analyze and explain the results

Command used:
```bash
$ sudo nmap -T4 -A 127.0.0.1
```
- `-T4` - The T option refers to "Timing template", while the number corresponds to the level. The lower the level, the longer it takes to scan
  - Levels are:
  - paranoid (0)
  - sneaky (1)
  - polite (2)
  - normal (3)
  - aggressive (4)
  - insane (5)
- `A` - Enable OS detection, version detection, script scanning, and traceroute

_Source: nmap man pages_

<img width="1577" height="387" alt="2026-03-24-19:39:29" src="https://github.com/user-attachments/assets/77501207-1258-4ca0-83c4-986a59877157" />


**Results**
- Host is up
- All 1000 common ports are closed
- The OS detection did not work
- Traceroute tells us the target is "0 hop away", signifying that we scanned localhost




-----



# D) The Port Got Action
**Objective**
- Install two arbitrary daemons and scan again


Everything you need is already installed, so I simply spin up a web server using `busybox` and the `SSH server` using `systemctl`

```bash
$ busybox httpd -p 80
$ sudo systemctl start ssh
```

The scan was taking too long, so I `ctrl+c`:d the operation and changed the timing template to `T5`:

<img width="1582" height="832" alt="2026-03-24-19:59:05" src="https://github.com/user-attachments/assets/6a2c1438-a459-44ab-83b1-43eab0295148" />

--some output redacted--

<img width="1579" height="651" alt="2026-03-24-19:59:48" src="https://github.com/user-attachments/assets/14f7584f-d7b0-423d-9c29-bfb8ea7ecdf4" />


This time `nmap` found 2 open ports:
- Port `80` listening for http requests
- Port `22` waiting for SSH connections. It also identified the SSH version
- It also managed to give us some hints on the OS!

Even with T level "insane" it took way over 1 minute to complete the scan... I suspect it's because it was probing for nonexisting HTML pages 🤷‍♂️



------




# E) HTB
**Objective**
- Solve a box of your choosing from HackTheBox

## Setting up
I already have an account set up and a few labs under my belt, so I navigated to --> `HTB Labs` --> `Starting point` --> `Tier 1 - Fundamental Exploitation` and chose the box called `Sequel` 

Connect to the HTB network using OpenVPN:
```bash
$ sudo openvpn ./starting_points_eu-starting-point-1-dhcp.ovpn
```

<img width="1415" height="422" alt="2026-03-26-16:53:33" src="https://github.com/user-attachments/assets/1355710b-a692-42a0-8d71-b231db41e419" />


## Workflow
I started by doing a `default script scan` on the target

```bash
$ nmap -sC -T4 10.129.42.29
```
<img width="1684" height="570" alt="2026-03-26-17:01:45" src="https://github.com/user-attachments/assets/bab75602-a92a-439b-ab35-d39791ba9854" />


From the scan we get to know that we're working with a db-server running MySQL (as well as it's version).

Next we try connecting to the server using the SQL CLI client:

<img width="1174" height="116" alt="2026-03-26-17:13:14" src="https://github.com/user-attachments/assets/722aaf3d-a108-411b-a9a5-87916c6e7cab" />

There was an error with TLS (which is used by default), so we search the man pages for a solution.
```Terminal
$ man mysql
```
1. We find the correct flag to use (--skip-tls) and try again.
2. This time we get `access denied`, so I remove the `-p` flag (password) and try again.
3. It doesn't work either, so we change user to  `root`:

<img width="1324" height="536" alt="2026-03-26-17:17:08" src="https://github.com/user-attachments/assets/91cea834-4d7c-4c05-9f84-9465f9ef5b56" />


The root account wasn't secured properly, so we're able to access the database!

> [!TIP]
> Search for "SQL cheat sheet" online and place it so that you can quickly recall the syntax!

Once inside, we want to know what databases are present:
```sql
MariaDB > SHOW databases;
```

<img width="304" height="227" alt="2026-03-26-17:23:31" src="https://github.com/user-attachments/assets/fb7915e5-3c30-4265-8ab7-0f25bfba59c5" />


The other DBs are standard, so we take `htb` under inspection
```sql
MariaDB > USE htb;
MariaDB [htb]> SHOW tables;
MariaDB [htb]> SELECT * FROM config;
```
<img width="1052" height="545" alt="2026-03-26-17:29:33" src="https://github.com/user-attachments/assets/bafcb9ee-fe4a-488b-9459-b42060879c8b" />


And there you have it!

<img width="451" height="63" alt="2026-03-26-17:31:27" src="https://github.com/user-attachments/assets/3557ce44-941d-410d-8066-ea52f9a569bb" />

