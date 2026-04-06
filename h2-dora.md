# X) Read/Watch/Listen & Summarize
**Objective**
- A few bullet points per section is enough to summarize
- Add your own observation, question or idea

## X.1) [Dora and TLPT testing](<https://terokarvinen.com/buuri-2026-dora-and-threat-lead-penetration-testing/buuri-2026-dora-and-threat-lead-penetration-testing--teros-pentest-course.pdf>)
**DORA**
- Digital Operations Resilience Act
- EU-wide regulation for financial sector resilience
- Requirements for two types of testing:
  1. Basic digital operational resilience testing: vulnerability assesments, source code reviews etc..
  2. Threat-led penetration testing (TLPT)


**TIBER**
- Threat Intelligence-Based Ethical Red Teaming
- TIBER-EU > TIBER-FI
- Following TIBER, a financial entity ensures their project is well-controlled, produces good results and fullfils requirements
- TIBER projects
  - Key roles:
    - Control Team (CT), Control Team Lead (CTL)
    - Threat Intelligence Provider (TIP)
    - Red Team Testers (RTT)
    - Blue Team (BT)


**Common Red Team Blunders**
- Treating the project as test of skill and not moving forward soon enough
- Overconfidence and not tailoring tools enough
- Underestimating the maturity of control mechanisms
- Lack of clear communicationg with the control team

 
**Insight**
- Marko made a good point about being an expert in IT technology before being an expert in testing
- Without having a foundational knowledge of how the systems work, how does one expect to break them?



## X.2 ) [DORA regulation](<https://eur-lex.europa.eu/eli/reg/2022/2554/oj/eng>)
**Article 26: Advanced testing of ICT tools, systems and processes based on TLPT**
- Financial entities have to carry out advanced testing by means of TLTP at least every 3 years (can be increased or reduced)
- Testing have to cover several or all critical functions of a financial entity, and testing shall be performed on live production systems


**Article 27: Requirements for testers for the carrying out of TLPT**
- Testers shall:
  - Be of the highest suitability and reputability
  - Possess expertise in threat intelligence, penetration testing and red team testing
  - Be certified by an accreditation body or adhere to formal codes of conduct or ethical testing
- When using internal testers:
  - Use has been approved by the relevant competent authority or by the single public authority designated in accordance with Article 29(9) and (10)
  - Verification that the financial entity has sufficient dedicated resources and ensured that conflicts of interest are avoided throughout the design and execution phases of the test
  - The threat intelligence provider is external to the financial entity

**Thoughts**
- Don't know how far the **mandated** pen testing has evolved, but curious to see when it's going to land at all companies handling any sensitive customer data

## X.3) [TIBER-FI Procedures and Guidelines](<https://www.suomenpankki.fi/globalassets/bof/en/money-and-payments/the-bank-of-finland-as-catalyst-payments-council/tiber-fi/tiber-fi-2.0-procedures-and-guidelines.pdf>)

**5.4: Testing Phase**
> Note: Excluding all 5.4.x sections as per instructions
- The red-teaming phase consists of two separate process steps, namely: the Red Team Test Plan (RTTP) creation and active testing
- When the RTTP reaches its final stage, stakeholders hold a RTTP meating, where the plan is approved
- Different methologies might then be used to conduct the testing, the document here presents the intrustion kill-chain covered in the previous assigment
- To facilitate more effective testing, and keeping in mind the advantage actual threat actors have as well as time/resource constraints, the entity being tested may deliver additional information to testers to speed up the process etc..
- Should a "leg-up" be provided, it has to be duly logged!


**Thoughts**
- The document illuminates the contrast between ethical hackers and real world threat actors, and how it might effect on test results
- Expanding on this topic, what then becomes the line an ethical hacker should never cross in terms of tools/methologies used?





------




# A) Metasploitable
**Objective**
- Install Metasploitable 2 on a VM

## Walkthrough

Download the `metasploitable` zip file from [here](<https://www.rapid7.com/products/metasploit/metasploitable/>).

Unzipping the file leaves us with the `Metasploitable2-linux` directory.

By issuing the `file` command we get additional information on the content:

<img width="1702" height="259" alt="2026-04-02-22:49:46" src="https://github.com/user-attachments/assets/3c82e6d5-ef31-4b64-a4f0-1eb770208983" />


We're mainly concerned with the `Metasploitable.vmdk` file which is the `disk image` for the VM. 
As i'm using `KVM/QEMU` for virtualization we'll start off by converting the `vmdk` image to `qcow`.

**Q:** What's `qcow`? 

**A:** It's a _virtual hard disk_ format used by QEMU!


We perform the conversion like so:
```bash
$ qemu-img convert -p -f vmdk -O qcow2 Metasploitable.vmdk metasploitable2.qcow2
```

I noticed the disk was on the smaller side, so I doubled the size before starting up the VM 


> [!NOTE]
> We still have to make the new space available to the system once inside

<img width="1381" height="567" alt="2026-04-03-00:08:03" src="https://github.com/user-attachments/assets/48d36854-97b2-4f8e-8e24-aab6fb5781d9" />

<img width="1423" height="252" alt="2026-04-03-00:08:42" src="https://github.com/user-attachments/assets/0c5f65a9-07ff-457e-9c92-26f703661af8" />




We then move the disk to the `libvirt` image pool:
```bash
$ sudo mv metasploitable.qcow2 /var/lib/libvirt/images
```


Inside `virt-manager`:
- We create a new virtual machine and use option `Import existing disk image`
  - <img width="663" height="549" alt="2026-04-02-23:20:49" src="https://github.com/user-attachments/assets/6693fa72-3ea8-4dc3-b889-807d1b36b5d4" />
- Choose the `metasploitable.qcow2` image as the volume
- Assign **4MB of RAM** along with **2 vCPU's**
- Use virtual NAT networking

When the configs are done, we spin up the VM and login with `User` & `Password` == `msfadmin`. 

First order of business: load correct keyboard

<img width="963" height="200" alt="2026-04-03-00:25:29" src="https://github.com/user-attachments/assets/a800052c-9b4d-4910-9d24-573d5cf7399d" />


Now we can make the new space we created earlier available to the system -->

## Resize Workflow
1. Identify what needs to be changed

<img width="1521" height="780" alt="2026-04-03-00:41:03" src="https://github.com/user-attachments/assets/ca9c247f-2d53-46b2-b356-fc91c7567216" />



2. Modify partitions using the `fdisk` utility

<img width="1595" height="952" alt="2026-04-03-00:54:25" src="https://github.com/user-attachments/assets/6c2bd2a7-e8ed-44dd-9ba8-b7ff3310cfed" />

<img width="1592" height="451" alt="2026-04-03-00:56:09" src="https://github.com/user-attachments/assets/91ca75ef-b2a1-471a-a562-07879c7e55e0" />


3. Reboot the the system
4. Resize the "physical" volume
5. Extend the root LV to use all available free space
6. Expand the filesystem to utilize this newly provided space


<img width="1762" height="840" alt="2026-04-03-01:05:50" src="https://github.com/user-attachments/assets/bada8892-3117-418e-afaf-90e83653283f" />


<img width="1284" height="161" alt="2026-04-03-01:06:14" src="https://github.com/user-attachments/assets/0cf4e553-3ae4-4426-b6d9-da4544c53853" />

And there you have it!



-----





# B) Virtual Networking && C) No Lurking!
**Objective**
- Create a virtual network for `kali` & `metasploitable`
- Show with test that
  - The VMs cannot reach the internet
  - The VMs can reach each other


## Option 1
Use default NAT network created and managed by `libvirt`.

When you want to cut the VMs off from the internet, you mess with the networking on the host.



## Option 2
Another option is to create _complete isolation_, without the need to toggle host networking.

We achieve this by creating an `xml` file storing our configuration and applying it using `virsh`:
```bash
$ nvim isolated.xml
```

```xml
<network>
  <name>isolated</name>
  <bridge name='virbr1' stp='on' delay='0'/>
  <ip address="192.168.120.1" netmask="255.255.255.0">
    <dhcp>
      <range start="192.168.120.10" end="192.168.120.254"/>
    </dhcp>
  </ip>
</network>
```

```bash
$ sudo virsh net-define isolated.xml
```
Verify:
- <img width="735" height="136" alt="2026-04-05-16:40:34" src="https://github.com/user-attachments/assets/b5d5ef3f-20c4-4c52-bfa9-63f302dca216" />


Start the network
```bash
$ sudo virsh net-start isolated
```


On `metasploitable` we simply change the network source on the NIC like so:

<img width="1848" height="445" alt="2026-04-05-16:43:57" src="https://github.com/user-attachments/assets/f96f54e6-133a-473b-b291-0604ae568bf7" />


Because we want to be able to access the internet from `kali`, here we don't _change_ the network source, but rather **add** a NIC with the isolated network as the source:

<img width="1481" height="1293" alt="2026-04-05-16:51:10" src="https://github.com/user-attachments/assets/ff1aa66a-78eb-4b24-aa78-e0c2cd5e9702" />


We then prove isolation by doing the following test:
- Ping an outside host
- Ping a machine on the same network

```terminal
$ ping -c 2 1.1.1.1                # <-- Pinging the cloudflare name server

$ sudo nmap -sn 192.168.120.1/24   # <-- Ping scan on the network gives us a list of available hosts

$ ping -c 2 192.168.120.215        # <-- Pinging a host on the same network
```

<img width="1065" height="729" alt="2026-04-05-16:55:52" src="https://github.com/user-attachments/assets/ec004fee-a197-4f5b-bf5b-3cef002f10db" />


### What if we want to access the internet?
We simply switch interfaces!

First we figure out which interface to toggle:

<img width="1269" height="417" alt="2026-04-05-16:57:56" src="https://github.com/user-attachments/assets/8dc46812-a6ab-41b0-ab98-1e5986176915" />

And then activate it:
```bash
$ nmcli device up eth0
```

<img width="1332" height="755" alt="2026-04-05-17:00:10" src="https://github.com/user-attachments/assets/cfed878e-206c-4ed0-b3be-363989871035" />


As we can see from the output of the status command, activating the other interface will disconnect the other:

<img width="809" height="175" alt="2026-04-05-17:00:31" src="https://github.com/user-attachments/assets/e9b4bd53-c29f-4e91-8de0-a16d68befb7c" />



**Help received**
- The ol' reliable `man` pages
- [libvirt](<https://wiki.libvirt.org/VirtualNetworking.html>) documentation







-------






# D) Find the target
**Objective**
- Find metasploitable

## Recon
A quick `ping scan` on the target network will do the trick:
```bash
$ sudo nmap -sn 192.168.120.1/24
```
<img width="983" height="348" alt="2026-04-05-17:24:18" src="https://github.com/user-attachments/assets/0b1057b4-a07e-4b48-af89-8f6f8eee2b3a" />



3 hosts are live on the network: The gateway `.1`, our own machine (kali) `.233` and the suspected target `.215`. 

We confirm it truly is the target by issuing the `curl` command:

<img width="1072" height="765" alt="2026-04-05-17:26:26" src="https://github.com/user-attachments/assets/9cf0d9e2-20da-43a1-893f-df1855e4a454" />



And there you have it, target is confirmed to have an IP-address of `192.168.122.215`.






--------





# E) Scan
**Objective**
- Port scan metasploitable
- Pick 2-3 ports of interest from an attackers perspective
- Explain and analyze results


## Discovery
We perform a very basic port scan in order to find out which ports are open:
```bash
$ sudo nmap -p- -T4 192.168.120.215
```
We use the`-p-` option for all ports and `-T4` for faster execution.


<img width="839" height="872" alt="2026-04-05-17:28:29" src="https://github.com/user-attachments/assets/4a8e4f83-e596-4e73-8a90-d1842ddcd569" />


Personally I would start with ports: `23/telnet`, `22/ssh`, `2049/nfs`, `3306/mysql` and `5432/postgresql`.

I listed a few more than 3 because telnet & SSH go in the same category, as well as mysql & postgre.

What we're targeting:
- 23/22 --> Remote access to the server
- 2049 --> Network File System
- 3306/5432 --> Database

If breached, this is the hackers dream: gaining remote access to the server, reading/manipulating important files and accessing the database.

Now we can do even deeper probing as we know what to look for:
```bash
$ sudo nmap -A -T4 -p 22,23,2049,3306,5432 192.168.120.215
```
<img width="1483" height="787" alt="2026-04-05-18:20:59" src="https://github.com/user-attachments/assets/76cb151b-1e7c-4199-a8a5-2e68946eef52" />

We get information on the OS and versions of the services we're planning to target. This knowledge is useful when searching for vulnerabilities specific to a certain version.






-------






# F) Bonus: Break In and Enter
We have the IP address of the target and know which ports are open, let's get cracking!


I wanted to break in the DB, so I tried to login remotely but got the error `"TLS/SSL: wrong version number"`. So I decided to skip SSL completely by appending the `--skip-ssl` flag. 
It didn't let me pass, so I switched the user from `admin` to `root` and managed to get inside:

<img width="1534" height="519" alt="2026-04-06-14:18:29" src="https://github.com/user-attachments/assets/86334cdd-cce1-40ca-a56b-a4094f9d4667" />



I found the databases:

<img width="754" height="391" alt="2026-04-06-14:24:00" src="https://github.com/user-attachments/assets/eab3bd97-8306-456d-87de-f8cb85675cab" />


And picked the first one => `dvwa` 
```sql
[(none)]> USE dvwa;

[dwva]> SHOW tables;
```
<img width="833" height="274" alt="2026-04-06-14:26:26" src="https://github.com/user-attachments/assets/4452f52b-0771-4fe2-9dad-e849f44d4a01" />

Naturally the `users` table is a high value target, so we want to know all there is to know about this table:
```sql
[dvwa]> SELECT * from users;
```
<img width="1703" height="569" alt="2026-04-06-14:28:28" src="https://github.com/user-attachments/assets/0099c55c-7783-4034-9496-c4fb50a3dd2f" />


Once we've identified all the most important sections, we can then exfiltrate only the parts that matter:

<img width="749" height="334" alt="2026-04-06-14:54:16" src="https://github.com/user-attachments/assets/6ae2d96f-4b25-4f59-9a00-3ca1448a7d68" />


<img width="910" height="333" alt="2026-04-06-15:00:33" src="https://github.com/user-attachments/assets/9150cca3-05f7-4b9c-b89b-8287d43134c9" />

Now we can try to crack the hashes and so on.


But anyway, as we have already have access to the database, might as well do some other mischief:
```sql
[dvwa]> UPDATE users SET password = MD5('superSecret1234!') WHERE user='pablo';
```
<img width="1261" height="90" alt="2026-04-06-15:05:43" src="https://github.com/user-attachments/assets/00bf0a55-ef31-4dae-9b4f-905489c5f136" />

**Explanation:**
- We update a users password with our own so that we can access the web service

Only thing left to do is to type in the users name and updated password on the web page and we're in!

<img width="1343" height="684" alt="2026-04-06-15:06:56" src="https://github.com/user-attachments/assets/ae7cc9d1-572f-47b2-a30b-43649a7f041d" />


<img width="1463" height="460" alt="2026-04-06-15:07:35" src="https://github.com/user-attachments/assets/22f8e7c3-b98b-4b8d-8c2c-e0b64eb61630" />


> [!NOTE]
> If we want to avoid detection and have persistent access, this is definitively not the way to go about it.
>
> I just felt like trying something out here!
>
> I guess you would just try to crack the hashes and use the password that's already in place!


### Why not the admin account?
Because there was no challenge there:

<img width="933" height="599" alt="2026-04-06-15:07:55" src="https://github.com/user-attachments/assets/67e39431-131c-411c-9a0d-1006bd0d166c" />




