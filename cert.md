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






