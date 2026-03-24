# X) Read/watch/listen/summarize

## X.1 




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


I didn't install anything, I simply used `busybox` to setup a simple web server and started the SSH server

```bash
$ busybox httpd -p 80
$ sudo systemctl start ssh
```

The scan was taking too long, so I `ctrl+c`:d the operation and changed the timing template to `T5`:

<img width="1582" height="832" alt="2026-03-24-19:59:05" src="https://github.com/user-attachments/assets/6a2c1438-a459-44ab-83b1-43eab0295148" />

<img width="1579" height="651" alt="2026-03-24-19:59:48" src="https://github.com/user-attachments/assets/14f7584f-d7b0-423d-9c29-bfb8ea7ecdf4" />


This time `nmap` found 2 open ports, 80 serving http requests and 22 for SSH, it even identified the version on the latter. This time it also managed to give us some hints on the OS.

The scan took over one minute tho, I suspect it's because it was trying to find nonexisting HTML pages 🤷‍♂️.
