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

Unzipping the file leaves us with the `Metasploitable2-linux` directory containing data for creating the VM.

By issuing the `file` command we get additional information on the content:

<img width="1702" height="259" alt="2026-04-02-22:49:46" src="https://github.com/user-attachments/assets/3c82e6d5-ef31-4b64-a4f0-1eb770208983" />


We're mainly concerned with the `Metasploitable.vmdk` file which is the `disk image` for the VM. 
As i'm using `KVM/QEMU` for virtualization we'll start off by converting the `vmdk` image to `qcow`.

**Q:** What's `qcow`? 

**A:** It's a _virtual hard disk_ format used by QEMU!

> [!NOTE]
> I'm not going to give an in-depth explanation on the conversion, as it's outside the scope of this assigment, check out `$ man qemu-img` for more!


Here's the command used to convert a `vmdk` image file to `qcow`
```bash
$ qemu-img convert -p -f vmdk -O qcow2 Metasploitable.vmdk metasploitable2.qcow2
```

I noticed the disk was on the smaller side, so I doubled it before starting up the VM (we still have to make the new space available to the system once inside the VM)

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
- Assign 4MB of RAM along with 2 vCPU's
- Use virtual NAT networking

When the configs are done, we spin up the VM and login with `User` & `Password` == `msfadmin`. 

First order of business: load correct keyboard

<img width="963" height="200" alt="2026-04-03-00:25:29" src="https://github.com/user-attachments/assets/a800052c-9b4d-4910-9d24-573d5cf7399d" />


Now we can make the new space we created earlier available to the system -->

## Resize Workflow
1. Identify what needs to be changed

<img width="1521" height="780" alt="2026-04-03-00:41:03" src="https://github.com/user-attachments/assets/99787c08-cd9d-45be-8b6e-d3434a3eb394" />


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





# B) Virtual Networking
**Objective**
- Create a virtual network for `kali` & `metasploitable`






------





# C) No lurking!
**Objective**
- Show with test that
  - The VMs cannot reach the internet
  - The VMs can reach each other


