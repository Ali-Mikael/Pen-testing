# X) Read/Watch/Listen & Summarize
**Instructions**
- A few bullet points per section is enough for summarization

## X.1) Jaswal 2020: Mastering Metasploit
[Chapter 1: Approaching A Penetration Test Using Metasploit](<https://learning.oreilly.com/library/view/mastering-metasploit/9781838980078/B15076_01_Final_ASB_ePub.xhtml#_idParaDest-31>)
> Starting from section "Conducting a penetration test with Metasploit" until the end of the chapter


**Metasploit Terminology**
- Exploit - Block of code that exploits a vulnerability in a target
- Payload - Code that runs on the target after successful exploitation
- Auxiliary - Modules to provide additional functionalities (scanning, fuzzing etc..)
- Encoders - Obfuscating modules to avoid detection
- Meterpreter - A payload that uses in-memory DLL injection stagers. Provides various functions to be performed on target


**Benefits of Pen Testing With Metasploit**
- It's open source
- Ease of use, especially when conducting large scale operations
- Smart payload generation and easy to switch between payloads mid test
- Cleaner exits: Ability to leave the system without crashing it and maintaining persistent access


**Using Databases in Metasploit**
- Stores the results of gathered intelligence automatically
- Helps us build a knowledge base of hosts, services and vulnerabilities
- Some DB commands:
  - `analyze` - Analyzes db info about a target IP or range
  - `db_connect` - Interact with databases (other than the default one)
  - `db_export` / `db_import`- Does what the name suggests
  - `db_save` / `db_remove` - Same here
- It's good to keep separate tests separate!
  - You achieve this by making use of the `workspace` feature


The author does an excellent job in explaining the steps from reconnaissaince to exploiting a vulnerability all the way to moving laterally and gaining access to other systems/machines. Quick recap of what took place -->

**The Systematic Approach**
1. New test, new workspace: `workspace -a`
2. `No-ping` Nmap scan on the target --> Find open ports
3. `Port:445` => Possibly running an _SMB service on Windows7-10_
4. New scan but only for the port `445` using script `smb-os-discovery`
5. Results suggest: Windows 7 SPI Ultimate
6. Windows 7 is highly vulnerable against the `EternalBlue` exploit
7. Confirm vulnerability using the `smb-vuln-ms17-010` script
8. Double-confirm using Metasploit module: `auxiliary/scanner/smb/smb_ms17_010`
9. Use EternalBlue eploit module against target and gain system shell using `reverse TCP payload`
10. Upgrade shell to meterpeter: `sessions -u`
11. Avoid detection my migrating process
12. Enumerate domain & Domain Controller details using the `enum_domain` module
13. DC is on a separate network
14. Find out DC is accessible through the compromised host using the `arp` command
15. Add route to target network using the `autoroute` module
16. Find processes running _domain admin privileges_ using the `ps` command
17. Load `incognito` plugin on Meterpreter shell and list available tokens
18. Admin token was used and impersonated
19. Background sessions and load `current_user_psexec` module
20. Run module with initially compromised host session and use DC as RHOST
21. Bind TCP payload
22. Exploiting DC with SYSTEM-level privileges and gain Meterpreter access
23. `smart_hashdump` module to dump all hashes
24. Load mimikatz & kiwi plugins on Meterpreter and run `kerberos` & `creds_all` to find clear-text creds for user Apex on the DC



## X.2) No Port Scan
**Task**
- Explain what `nmap -sn` does
- Don't guess, back it up with sources

### Ping Scan
> `-sn` formerly known as `-sP` a.k.a "ping scan"

Nmap sends something called _host discovery probes_ to hosts on the network and only prints out available hosts that respond to the probes, while skipping the port scan after discovery.


**Q:** What is a "host discovery probe" ?

**A:** The default host discovery done with `-sn` consists of the following: 
- An **ICMP echo request**
- A **TCP SYN to port 443**
- A **TCP ACK to port 80**
- An **ICMP timestamp request**


**Good To Know:**
- When executed as an unprivileged user: only SYN packets are sent to P:80 & P:443
- Executed as a privileged user: ARP requests are sent (unless `--send-ip` is specified)


**Credit**
- Gordon "Fyodor" Lyon, author of Nmap

**Source**
- Nmap Reference Guide (manual pages)
- In order to view it on Linux/MacOS: Download Nmap and type `man nmap` in the terminal emulator








--------








# B) Metasploit DB
**Objective**
- Save the results from port scanning to Metasploit databases
- Scan it so that Metasploitable is included
- You should include at least version scanning (-sV)


## db_nmap





