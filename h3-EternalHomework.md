# X) Read/Watch/Listen & Summarize
**Instructions**
- A few bullet points per section is enough for summarization

## X.1) Jaswal 2020: Mastering Metasploit
[Chapter 1: Approaching A Penetration Test Using Metasploit](<https://learning.oreilly.com/library/view/mastering-metasploit/9781838980078/B15076_01_Final_ASB_ePub.xhtml#_idParaDest-31>)
> Starting from section "Conducting a penetration test with Metasploit" until the end of the chapter


**Metasploit Terminology**
- **Exploit** - Block of code that exploits a vulnerability in a target
- **Payload** - Code that runs on the target after successful exploitation
- **Auxiliary** - Modules to provide additional functionalities (scanning, fuzzing etc..)
- **Encoders** - Obfuscating modules to avoid detection
- **Meterpreter** - A payload that uses in-memory DLL injection stagers. Provides various functions to be performed on target


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

Nmap sends something called _host discovery probes_ to the network and prints out available hosts that respond to the probes while skipping the post discovery port scan.


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
- Save the port scanning results to a Metasploit database
- You should include at least version scanning (-sV)


## db_nmap
To start scanning and saving into the db, one must first start the database:
```bash
$ sudo msfdb init
```

Now let's access the Metasploit Framework console
```bash
$ sudo msfconsole
```

We then use the aggressive scan (`-A`) option, enumerating open ports, service versions and more:
```
msf > db_nmap -A -T4 192.168.120.215
```
By replacing the normal `nmap` command with `db_nmap` the results are automatically saved to the database, in the next task we'll look through it!





--------





# C) Saved Results
**Objective**
- Examine information saved to the Metasploit database
- Use commands `hosts` & `services`
- Try to filter these lists and search from them


## Recall What We Scanned
By issuing the `hosts` command we can see which hosts are live on the network. This because we used `db_nmap` while conducting a `ping scan` earlier.

<img width="1508" height="292" alt="2026-04-08-15:18:46" src="https://github.com/user-attachments/assets/218eecdf-0ab0-4b20-8509-0420ce431328" />


The `services` command shows all services that were identified during the host scan, if we further specify the `-u` flag it shows only services that are up:

<img width="1513" height="783" alt="2026-04-08-15:25:12" src="https://github.com/user-attachments/assets/e82c6e4f-db71-44cf-bdab-605de87a3ccc" />


By using the `-S` flag we can search for specific information, and with the `-p` flag we filter by port numbers:

<img width="1397" height="442" alt="2026-04-08-15:29:56" src="https://github.com/user-attachments/assets/60dae7de-c577-4f27-a6cb-94e7f553c5af" />





---------





# D) Internet Famous
**Objective**
- Find an exploit that comes with Metasploitable that has been public/famous

-------

We can search for `exploit modules` in Metasploitable by using the following command:
```bash
msf > search type:exploit
```

Here we can use the `-S` flag again to filter for results. S

ay we want all exploits relating to `ftp` for `Linux`
```bash
msf > search type:exploit platform:linux -S ftp
```

<img width="1590" height="492" alt="2026-04-08-16:22:42" src="https://github.com/user-attachments/assets/18b7391c-eea3-4e7d-83ff-358fa8529d26" />


## World famous
In order to adhere to objective, here's a world famous exploit for you:

<img width="1583" height="373" alt="2026-04-08-16:32:36" src="https://github.com/user-attachments/assets/f8821353-75b5-4735-a3da-56d3ddd95a6f" />





-------





# E) Compare
**Objective**
- Compare the saving of results via `nmap -oA results.txt` vs `db_nmap`
- Outline the positives in each

-------


By saving the results through nmap's builting -oA function
```
sudo nmap -sV 192.168.120.215 -oA target
```
We get 3 files: 
- The normal output you'd get in the terminal
- A more concise version (regarding to format)
- An xml version

<img width="1030" height="159" alt="2026-04-08-16:41:03" src="https://github.com/user-attachments/assets/823dedc5-d46a-4ee2-86f8-66d86ddbd6e4" />

This is easier to parse programatically and create more comples queries, especially for large scale operations.
Also it's convenient to feed the parsed results to say another program etc.


Then again `db_nmap` integrates so much better with the Metasploit framework and we're able to have a more clean, uninterrupted workflow. For example, we can set the `RHOSTS` variable through `services`.


<img width="1576" height="571" alt="2026-04-08-16:47:20" src="https://github.com/user-attachments/assets/cb082cfd-917c-437b-90b1-4501cfb91856" />


Now say we want to test a vulnerability for all hosts running a certain service for example, it's easy! We filter all hosts relating to our query and set the RHOSTS variable accordingly -->

<img width="1570" height="290" alt="2026-04-08-16:48:21" src="https://github.com/user-attachments/assets/c719dc2f-bbf3-4b72-b25e-7d29edf3d69c" />







--------





# F) Break In
**Objective**
- Break into Metasploitable's `vsftpd` service


## Workflow
As you might've noticed from the last screen-capture, we know the version, se searching for an exploit becomes easy:
```bash
msf > search type:exploit vsftpd 2.3.4
```
Side note: I noticed that we can skip the -S flag and just append the version and the filtering works all the same.

<img width="1565" height="305" alt="2026-04-08-16:56:22" src="https://github.com/user-attachments/assets/551cfa94-1ada-41e3-9956-d00849cc0138" />


We got the ID of the exploit so we use it with the command
```bash
msf > use 0
```

Check what mandatory options need to be set by using the `options` cmd:

<img width="1586" height="280" alt="2026-04-08-17:01:10" src="https://github.com/user-attachments/assets/a7315f86-f42d-4898-87aa-8ebd724d4d74" />


We already set `RHOSTS` in the previous task so we're good to go!

Let's `check` if the target is vulnerable to this exploit:

<img width="1058" height="82" alt="2026-04-08-17:03:37" src="https://github.com/user-attachments/assets/b7f20688-a5e5-4d1b-ba73-90d974cdc4c0" />


Now we can `run` the module.
```
msf exploit(unix/ftp/vsftpd_234_backdoor) > run
```

I forgot to set the `LHOST` option, so we'll set it and run again:

<img width="1522" height="271" alt="2026-04-08-17:09:55" src="https://github.com/user-attachments/assets/1b57dde0-b1c8-4f06-834c-05ebbac4bb87" />



And just like that we have access the server:

<img width="1581" height="804" alt="2026-04-08-17:10:27" src="https://github.com/user-attachments/assets/eecd6baa-dde0-44cc-88cd-9d58af0dc1c0" />

<img width="1134" height="275" alt="2026-04-08-17:17:02" src="https://github.com/user-attachments/assets/d6272453-4348-45f5-8f07-b0c92c5ea6c8" />


There's a lot of functionality in the meterpeter shell, but we can also drop into a system shell and exit it anytime we want:

<img width="748" height="187" alt="2026-04-08-17:21:38" src="https://github.com/user-attachments/assets/e7012f83-84b6-4920-b965-61b395482870" />





--------





# G) Lateral Movement
**Objective**
- Gather information on Metasploitable necessary for lateral movement
- Analyze and explain how you would utilize the results


## Gathering Intel
Simplest way imo to gather all the users on the system is to drop to a system shell and give the following command:
```bash
cat /etc/passwd | cut -d ":" -f1 > users.txt
```
What the command does
- Prints out the `passwd` file
- Uses `cut` with the options: delimiter = ":", field_to_print = 1
  - So we only get the first field, which is the user
- Then saves the output into a file called `users.txt`


Snippet of the file:
- <img width="878" height="535" alt="2026-04-08-23:06:25" src="https://github.com/user-attachments/assets/764584f8-21c9-4176-944d-6aa212694df0" />



Let's also save the hashes along with respective users:
```bash
cat /etc/shadow | cut -d ":" -f1-2 | grep -vE '\*$' > hashes.txt
```
Explained:
- Same as before, but this time we want the first and second field >> `-f1-2`
- We invert the grep, meaning we exclude all lines ending with an asterisk >> `-vE '\*$'`
  - Why?
    - Because there's a lot of users (usually daemons) that don't have any password access, noted with the `*` in the password field
    - <img width="638" height="311" alt="2026-04-08-23:16:10" src="https://github.com/user-attachments/assets/cca05535-88f6-4911-9987-cd7236df8984" />
- Lastly we save the output >> `hashes.txt`
- I wanted to keep the `!` and `!!` here becuase they will tell if a user account is accessible without a password or whether the password is locked


<img width="1132" height="433" alt="2026-04-08-23:21:48" src="https://github.com/user-attachments/assets/78d6fa67-718e-4ba5-8ec0-7277651642ce" />



Lastly we exfiltrate the findings and remove the files:

<img width="1590" height="203" alt="2026-04-08-23:30:48" src="https://github.com/user-attachments/assets/cd21d5db-2fb0-4a39-aa42-b2c713794bd5" />


I noticed the user `user` had some SSH keys we could steal:

<img width="1262" height="143" alt="2026-04-08-23:29:57" src="https://github.com/user-attachments/assets/41265b35-e97e-40d2-833c-730d70a36496" />

Also did the same for `msfadmin`!


While going through my findings I noticed that the `download` command messed up the `hashes.txt` file. It ended up creating a directory of it and storing the users file inside it, so I had to go back and redo the same steps.


Our findings so far:

<img width="1199" height="396" alt="2026-04-08-23:42:13" src="https://github.com/user-attachments/assets/0b184500-c7fc-42d8-9c05-50a519c16504" />




