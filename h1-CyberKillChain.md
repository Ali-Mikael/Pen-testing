# X) Read/watch/listen/summarize

## X.1 




------

# A) Install Kali
Kali is already installed:

<img width="1196" height="470" alt="2026-03-24-18:20:07" src="https://github.com/user-attachments/assets/193092ff-99bb-4f0e-a026-ee9b488d0f58" />


I installed the netinstaller ISO image from their website and then set it up in a VM. 
Using **KVM/QEMU** for virtualization/emulation and then manage the whole shabang using **libvirt**.

# B) Kali goes dark
**Objective**
- Disconnect kali-vm from the network
- Prove with tests that it cannot reach the internet

Issue the command
```
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
```
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

Source: nmap man pages


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


I didn't install anything, I simply used busybox to setup a simple web server and started the SSH server

```
$ busybox httpd -p 80
$ sudo systemctl start ssh
```

The scan was taking too long, so I changed the timing template to `T5` 



