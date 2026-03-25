# X) Read/watch/listen/summarize

## X.1 Darknet Diaries - [EP 106: Greg](<https://darknetdiaries.com/episode/160/>)
This ep tells a story about how Greg Linares (AKA Laughing Mantis) became the youngest hacker to arrested in Arizona.
- Found exploits and wrote malware in MIDDLESCHOOL
- Actually used the malware he wrote on criminals
- In high school: Why bother attending the boring classes when you can just hack the system and give yourself perfect attendance and passing grades
  - Well, it works until it doesn't, and he ended up getting arrested and thrown into juvie
- Greg was blessed, as the police fumbled the evidence and the case got dropped
- This didn't stop him from being kicked out of his home tho, so he ended up living in a halfway home until 18
- He did some music and toured the world for a bit
- After his music career, he got a job coding backend at this massage parlor, while creating exploits and publishing them online on the side
  - That's how he got scouted by a cyber security company
  - At the company he worked with a team finding zero days
  - The work entailed a lot of reverse engineering and modifying/cracking binaries
- He found a remote code execution vulnerability from the latest microsoft word version
  - Make the program crash --> take control of a pointer --> inject shell code into memory during the crash ==> Arbitrary remote code execution
  - Only problem was that it could only be done with a debugger attached to the program
  - Because he didn't verify it all the way, and the press release was already out, his career was now on the line
  - He got one chance of redeeming himself, and after a 3 day bender found a vulnerability in the microsoft visio program
  - Face of the company saved and greg got to keep his job
- Greg went on to have a pretty succesful red-teaming career
- Big note from Greg: Never overlook layer 2 (data-link) attacks in penetration testing / red-teaming. That's that bread and butter!


## X.2 [Intelligence-Driven Computer Network Defence Informed by Analysis of Adversary Campaigns and Intrusion Kill Chains](<https://lockheedmartin.com/content/dam/lockheed-martin/rms/documents/cyber/LM-White-Paper-Intel-Driven-Defense.pdf>)

**Abstract**
- APT / Advanced Persistent Threat = A class of threats representing well-resourced and trained adversaries conducting multi-year intrusion campaigns
- Network defence techniques leveraging knowledge about these adversaries can create an intelligence feedback loop, coupled with a kill chain model to describe phases of intrusion, among other things, form the basis of intelligence-driven CND (Computer Network Defense)

**3.2 Intrusion Kill Chain**
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


I didn't install anything, I simply used `busybox` to setup a simple web server and started the `SSH server`

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

Even with the T level "insane" it took way over 1 minute to complete the scan... I suspect it's because it was probing for nonexisting HTML pages 🤷‍♂️.



------




# E) HTB
**Objective**
- Solve a box of your choosing from HackTheBox


