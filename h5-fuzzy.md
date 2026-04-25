# X) Read/Watch/Listen & Summarize
**Objective**
- A few bullet points per section is enough to summarize
- Add in your own observation, question, idea or comment


# X.1) [Find Hidden Web Directories](<https://terokarvinen.com/2023/fuzz-urls-find-hidden-directories/>)
Summary can be found --> [here](<https://github.com/Ali-Mikael/Application-hacking/blob/main/h2-BreakAndFix.md#x2-fuzzing>).

**My own question**
- How **fast** do you have to **fuzz** to get noticed?


# X.2) Fuff project [README](<https://github.com/ffuf/ffuf/blob/master/README.md>)
- A fast web fuzzer written in Go
- You can no only use ffuf hidden directories, but also for `GET` and `POST` requests. Just specify what to FUZZ <-- this works again as the keyword!
- Use the keyword anywhere in the **URL** `-u`, **headers** `-H` or **POST** data `-d`
- Press `enter` during execution to enter _interactive_ mode and reconfigure your filters etc.

**My own question**
- What's the limit to how fast one can fuzz without disturbing the quantum realm?







--------






# A) Dirfuz-1
**Objective**
- Solve dirfuz-1 from the article you read earlier

**Walkthrough**
- Read all about it --> [here!](<https://github.com/Ali-Mikael/Application-hacking/blob/main/h2-BreakAndFix.md#c-dirfuzt-1>)






--------







# B) Target Practise
**Objective**
- Install the [FuffMe-target](<https://github.com/BuildHackSecure/ffufme>)


## Install
If for some reason you don't want to install, the live target can be found [here](<http://ffuf.me/>). 

Launching the vulnerable app locally requires `docker`, make sure you have it, further installation instructions in the README for the [ffufme](<https://github.com/BuildHackSecure/ffufme>) repo.

It goes like this:
```bash
$ git clone https://github.com/BuildHackSecure/ffufme.git
$ cd ffufme

# Docker searches the current directory for a dockerfile, builds the image and assigns it a tag=ffufme
$ sudo docker build -t ffufme .

# Run container from the image to listen on port 80:
$ sudo docker run -d -p 80:80 ffufme

# Confirm the container is running:
$ sudo docker ps
```

Now that everything is set, let's get FUZZING -->

> [!TIP]
> Kali:`/usr/share/wordlists` contains a bunch of different wordlists you can use!
>
> If you want the `seclists` specifically just `$ sudo apt install seclists`
> 
> When you invoke it you get a top level listing and taken to the directory:
>
> <img width="1292" height="529" alt="2026-04-25-13:11:30" src="https://github.com/user-attachments/assets/086167f8-59f1-49f6-8f01-53afb3e1e76c" />
>
> (It also gets symlinked in /usr/share/wordlists)



# C) Basic Content Discovery
The URL we're working with is `http://localhost/`. We `curl` the URL to get the landing page and find the following:

<img width="1224" height="457" alt="2026-04-25-10:57:32" src="https://github.com/user-attachments/assets/fbb641a5-7578-4d04-a18f-560a794e18e9" />

We're doing the first task, so the target URL becomes:
```terminal
http://localhost/cd/basic
```

Let's FUZZ it and see if we can find anything **useful** (from a hackers perspective):
```terminal
$ ffuf -w /user/share/wordlists/dirb/common.txt -u http://localhost/cd/basic/FUZZ -mc all -c -v -fc 404
```
If you've been paying attention you should know the parameters used. As a newcomer, we use the `-fc` flag to filter out `404 not found` responses, this give us a very clean, uncluttered output:

<img width="1784" height="837" alt="2026-04-25-11:05:41" src="https://github.com/user-attachments/assets/ca43edcb-b924-441d-8ca4-0989110839cd" />


Only thing left to do is to collect the hidden data:

<img width="1150" height="252" alt="2026-04-25-11:08:50" src="https://github.com/user-attachments/assets/3bf44dca-9485-4832-b5b4-ad7b287bb249" />


# d) Content Discovery With Recursion
What's the next URL we can try?
=> We recall that all the URLs contain `cd`, so can `grep` the `curl` output:

<img width="1397" height="409" alt="2026-04-25-11:13:49" src="https://github.com/user-attachments/assets/e0952e90-1c3a-440d-bb23-f43901b6ae58" />


Okay, next target: `http://localhost/cd/recursion`.

> [!TIP]
> Keep another terminal window or tab open where you can quickly check all the options with `man ffuf` or `ffuf --help`.
>
> Then you can keep crafting the attack command at the same time you're reading and gaining insight!

Our attack looks like this:
```terminal
$ ffuf -w /user/share/wordlists/dirb/common.txt -recursion -u http://localhost/cd/recursion/FUZZ -mc all -c -v -fc 404
```
We add the `-recursion` flag to the command, pretty self explanatory!

Here are all the results, you can see how the recursion progresses:

<img width="1745" height="1013" alt="2026-04-25-12:03:55" src="https://github.com/user-attachments/assets/a4c5fbc2-8a3a-462f-84c8-d3977ece4b47" />

### Loot
<img width="930" height="132" alt="2026-04-25-12:06:33" src="https://github.com/user-attachments/assets/66a8584f-89a9-4886-8d1e-514947744e3a" />



# e) Content Discovery With File Extensions
Target:
```
http://localhost/cd/ext
```

The first run gave us a single directory:

<img width="1449" height="218" alt="2026-04-25-12:16:34" src="https://github.com/user-attachments/assets/46503f5e-2f19-4ee4-b73b-7aa0a3c6983c" />

But we cannot access the contents:

<img width="972" height="184" alt="2026-04-25-12:32:48" src="https://github.com/user-attachments/assets/931b1295-98d1-4331-b72f-b6579096bca4" />

We check out the `man` pages and find out that in order to look for file extensions we need to use the `-e` flag.

<img width="1089" height="68" alt="2026-04-25-13:16:13" src="https://github.com/user-attachments/assets/0839fa68-d859-425f-97d5-9b415bd6830e" />

I found a `web-extensions.txt` to use in `**/seclists/Discovery/Web-Content/`, I also noticed a `common.txt` in the same directory, so I changed the original wordlist for that one:
```
$ ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt -e /usr/share/seclists/Discovery/Web-Content/web-extensions.txt -u http://localhost/cd/ext/logs/FUZZ -mc all -c -v -fc 404
```
and got nothing in return:

<img width="1912" height="898" alt="2026-04-25-13:05:25" src="https://github.com/user-attachments/assets/36d56406-312a-4410-b830-f972dbfbf2ab" />



Somethings wrong with either the command or wordlists used.

# f) No 404 Status
# g) Param Mining
# h) Rate Limited
# i) Subdomains - Virtual Host Enumeration

