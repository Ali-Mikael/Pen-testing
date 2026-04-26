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
- Not only can you use ffuf to discover hidden directories, but also for fuzzing `GET` and `POST` requests. Just specify what to FUZZ <-- this works again as the keyword!
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
If for some reason you don't want to install locally, the live target can be found [here](<http://ffuf.me/>). 

Launching the vulnerable app requires `docker`, make sure you have it! Further installation instructions in the README for the [ffufme](<https://github.com/BuildHackSecure/ffufme>) repo.

It goes like this:
```bash
$ git clone https://github.com/BuildHackSecure/ffufme.git
$ cd ffufme

# Docker searches the current directory for a dockerfile, builds the image and assigns it a tag=ffufme
$ sudo docker build -t ffufme .

# Run a container from the image, bind local port 80 to containers port 80:
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
> (It's symlinked in /usr/share/wordlists)




# C) Basic Content Discovery
The URL we're working with is `http://localhost/`. We `curl` the URL to get the landing page and find the following:

<img width="1224" height="457" alt="2026-04-25-10:57:32" src="https://github.com/user-attachments/assets/fbb641a5-7578-4d04-a18f-560a794e18e9" />

We're doing the first task, so the **target** URL becomes:
```http
http://localhost/cd/basic
```

Let's FUZZ it and see if we can find anything **useful**:
```bash
$ ffuf -w /user/share/wordlists/dirb/common.txt -u http://localhost/cd/basic/FUZZ -mc all -c -v -fc 404
```
If you've been paying attention you should know the parameters used. As a newcomer option we use the `-fc` flag to filter out `404 not found` responses, this give us a very clean, uncluttered output:

<img width="1784" height="837" alt="2026-04-25-11:05:41" src="https://github.com/user-attachments/assets/ca43edcb-b924-441d-8ca4-0989110839cd" />


Only thing left to do is to collect the hidden data:

<img width="1150" height="252" alt="2026-04-25-11:08:50" src="https://github.com/user-attachments/assets/3bf44dca-9485-4832-b5b4-ad7b287bb249" />




# D) Content Discovery With Recursion
**What's the next URL we can try?**
=> We recall that all the URLs contain `cd`, so we `grep` the `curl` output:

<img width="1397" height="409" alt="2026-04-25-11:13:49" src="https://github.com/user-attachments/assets/e0952e90-1c3a-440d-bb23-f43901b6ae58" />


Okay, next target: 
```http
http://localhost/cd/recursion
```

> [!TIP]
> Keep another terminal **window** or **tab** to recall all the options with `man ffuf` or `ffuf --help`.
>
> Now you can keep crafting your command same time you're reading!

Our attack looks like this:
```bash
$ ffuf -w /user/share/wordlists/dirb/common.txt -recursion -u http://localhost/cd/recursion/FUZZ -mc all -c -v -fc 404
```
We basically just added the `-recursion` flag to the previous command and changed the URL, pretty self explanatory! If you don't have any idea what `recursion` means I suggest you google or ask a friend.

## The result
Notice how the recursion progresses:

<img width="1745" height="1013" alt="2026-04-25-12:03:55" src="https://github.com/user-attachments/assets/a4c5fbc2-8a3a-462f-84c8-d3977ece4b47" />

## Loot
<img width="930" height="132" alt="2026-04-25-12:06:33" src="https://github.com/user-attachments/assets/66a8584f-89a9-4886-8d1e-514947744e3a" />





# E) Content Discovery With File Extensions
**Target:**
```http
http://localhost/cd/ext
```

The first run gave us a single directory:

<img width="1449" height="218" alt="2026-04-25-12:16:34" src="https://github.com/user-attachments/assets/46503f5e-2f19-4ee4-b73b-7aa0a3c6983c" />

But `curl` returns empty handed:

<img width="972" height="184" alt="2026-04-25-12:32:48" src="https://github.com/user-attachments/assets/931b1295-98d1-4331-b72f-b6579096bca4" />

We check out the `man` pages and find out that in order to look for `file extensions` we need to use the `-e` flag.

<img width="1089" height="68" alt="2026-04-25-13:16:13" src="https://github.com/user-attachments/assets/0839fa68-d859-425f-97d5-9b415bd6830e" />

I found `web-extensions.txt` to use in `seclists` (the one we presented earlier). I also noticed a `common.txt` in the same directory, so I changed the original wordlist for that one.

We then use the command:
```bash
$ ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt \
-e /usr/share/seclists/Discovery/Web-Content/web-extensions.txt \
-u http://localhost/cd/ext/logs/FUZZ \
-mc all -c -v -fc 404
```
and get nothing in return:

<img width="1912" height="898" alt="2026-04-25-13:05:25" src="https://github.com/user-attachments/assets/07429018-f530-43a7-b185-89af060d6bb4" />


At this point i'm thinking something's wrong with either the command or wordlists (or both). So back to the man pages and this time I really paid attention to "Comma separated list of extensions".

Our wordlist is **not** comma separated:
<img width="828" height="478" alt="2026-04-25-14:47:22" src="https://github.com/user-attachments/assets/cb2520ef-79ae-4fc2-b82b-040a57d8b06c" />

Maybe that's the problem? So I copied the wordlist to my home directory for modification and used vim to add the commas. By using **vim macros** it becomes very simple.

You start in `normal mode` at the beginning of the file and then execute a few steps, here are the commands used and what they mean:
```bash
qa = Starts the macro 
A = Moves to the end of the line and enters insert mode
, = Insert the comma
esc = Enter normal mode
j = Move one line down
q = Ends the macro

# We did: '$ wc -l web-extensions.txt' before editing so we know the file has 43 lines!

42@a = Executes the macro 42 times
```
The file now:

<img width="673" height="380" alt="2026-04-25-14:55:42" src="https://github.com/user-attachments/assets/d8640528-56f4-4cb8-926c-8acef43e144f" />


We execute the attack command again, this time with the modified file:
<img width="1568" height="452" alt="2026-04-25-14:57:26" src="https://github.com/user-attachments/assets/05b1ad9a-60a9-4072-8fbd-494c18773f78" />

But get nothing in return.... So I went back to modify the file and put everything on **same line** and tried again. Result => Nothing!

Lastly I just copied and pasted the file contents to the option:
```bash
$ ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt -e .asp,.aspx,.bat,.c,.cfm,.cgi,.css,.com,.dll,.exe,.hta,.htm,.html,.inc,.jhtml,.js,.jsa,.json,.jsp,.log,.mdb,.nsf,.pcap,.php,.php2,.php3,.php4,.php5,.php6,.php7,.phps,.pht,.phtml,.pl,.phar,.rb,.reg,.sh,.shtml,.sql,.swf,.txt,.xml \
-u http://localhost/cd/ext/logs/FUZZ \
-mc all -c -v -fc 404
```
And it worked 😂 !

<img width="1909" height="1062" alt="2026-04-25-15:03:34" src="https://github.com/user-attachments/assets/38f53c39-04e1-488e-aa03-2043b27eb38a" />

## Loot
<img width="896" height="133" alt="2026-04-25-15:04:02" src="https://github.com/user-attachments/assets/b81f2025-be1a-48fe-8be2-ab0401054cd3" />

## Pay attention much?
I felt really stupid after I realized in hindsight that we could've just used `.log` as we were fuzzing a **log directory**... 😂

<img width="1901" height="966" alt="2026-04-25-15:07:14" src="https://github.com/user-attachments/assets/3a228e93-357e-4921-bd0e-20160715e609" />

I got too focused on the file extension part that I lost sight of what we were actually targeting..


## Pointless?
Even though we did all of that extra for "nothing" at least we **learned how to fuzz multiple file extensions** at the same time.

By no means was this pointless!




# F) No 404 Status
**Target:**
```http
http://localhost/cd/no404
```

I'm suspecting that we want to filter out all `404` responses, which we have been doing all along, so we just continue like before:
```bash
$ ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt \
-u http://localhost/cd/no404/FUZZ \
-mc all -c -v -fc 404
```
But something strange happened, all of the sudden we're drowning in results:
<img width="700" height="1065" alt="2026-04-25-15:53:27" src="https://github.com/user-attachments/assets/45ffa429-f12b-4ac8-a404-0c2f0da7f2cb" />

I have 5000 scrollback lines in my terminal and couldn't even get to the top....

Many times we want to filter using the **size of the response**, and here we can see why! All of the _fake positives_ have a size of `669`, so we append `-fs 669` to the command (we filter by size=669) and go again:

<img width="1909" height="1014" alt="2026-04-25-15:57:12" src="https://github.com/user-attachments/assets/1ed1abb4-099e-4cd0-94ad-1b791c54a333" />


## Loot?
<img width="866" height="128" alt="2026-04-25-15:58:13" src="https://github.com/user-attachments/assets/177e531b-151b-41b0-90f5-a45d29c5e272" />

It says `Controller does not exist` so I'm not sure if this is the intended target, but we didn't find anything else and at least it doesn't return "page not found".

In contrast we can see what happens when we curl a fake positive:
- `curl http://localhost/cd/no404/registration` =>
- <img width="1326" height="166" alt="2026-04-25-16:00:47" src="https://github.com/user-attachments/assets/a6eddbbf-52f3-4aa7-a52a-6f127ea8785b" />





# G) Param Mining
**Target:**
```http
http://localhost/cd/param
```

Everything begins with basic reconnaissance:
```bash
$ ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt \
-u http://localhost/cd/param/FUZZ \
-mc all -c -v -fc 404
```
<img width="1492" height="364" alt="2026-04-25-16:10:54" src="https://github.com/user-attachments/assets/42ed3042-b839-45d2-a7f6-a7f6112d9583" />

So we need to craft a `GET` request and fuzz the params?

Time to search for a new wordlist:
```bash
$ find . -type f -name '*param*'

# Explained:
# We search the seclists directory for a `file` (-type f),
# and tell find to locate all files with 'param' in their name
```
<img width="1161" height="161" alt="2026-04-25-16:16:35" src="https://github.com/user-attachments/assets/ea667030-3e71-434a-8d1c-b26c650e80bd" />

I found a wordlist to use and looked for clues on how to apply it in the [ffuf wiki](<https://github.com/ffuf/ffuf/wiki>). Didn't really find anything so I went back to the [README](<https://github.com/ffuf/ffuf>) and found this: 

<img width="931" height="296" alt="2026-04-25-16:46:10" src="https://github.com/user-attachments/assets/26deaaf4-46ef-4f0d-8445-307730b3e764" />

I also did a quick recap on parameters [here](<https://apidog.com/articles/http-request-parameters-guide/>).

The command we try then looks like this:
```bash
$ ffuf -w url-params_from-top-55-most-popular-apps.txt -u http://localhost/cd/param/data?FUZZ
```
**Got nothing in return..**

So I modified the command, specifically the last part:
```bash
localhost/cd/param?data=FUZZ
```
This time we get a bunch of fake positives:
<img width="1858" height="1048" alt="2026-04-25-17:12:26" src="https://github.com/user-attachments/assets/bc18790d-b0bb-4529-b3ba-a330510a7a8f" />


I played around with a bunch of different variations without result. Lastly I just reverted the wordlist back to `common.txt` and used the first command we tried like so:
```bash
$ ffuf -w common.txt -u http://localhost/cd/param/data?FUZZ -c
```
We got a hit:

<img width="1612" height="816" alt="2026-04-25-17:08:58" src="https://github.com/user-attachments/assets/6b64249f-4373-4105-ad74-4cb4b84e25ec" />

## Loot
<img width="1181" height="132" alt="2026-04-25-17:09:34" src="https://github.com/user-attachments/assets/58672a74-4d56-4600-9e70-5393cf707b0e" />

## That easy?
Even though it looks like I just tried a few commands and got the result, it's not always that easy.
I tried a **bunch** of different stuff. Here's a snippet from my history 😂 -->

<img width="1908" height="846" alt="2026-04-25-17:20:15" src="https://github.com/user-attachments/assets/aeb50890-044b-4299-8b6f-0f24136219d1" />

I guess you could say I was fuzzing the fuzzer to get it fuzzing... LOL




# H) Rate Limited
**Target:**
```http
http://localhost/cd/rate/
```

You already know the drill! We start off with the basics
```bash
$ ffuf -w common.txt -u http://localhost/cd/rate/FUZZ -mc all -c -v -fc 404
```
and get flooded with `429 Too Many Request` responses:

<img width="1004" height="492" alt="2026-04-25-17:26:16" src="https://github.com/user-attachments/assets/be516b12-a966-4e25-a896-88f1aa4edda5" />

Had to google what the code means and found [this](<https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status/429>). Now that we know the server is `rate limiting` our responses, we go back to the man pages and return with:

<img width="744" height="73" alt="2026-04-25-17:40:02" src="https://github.com/user-attachments/assets/cdde7f42-9a67-4539-8c91-91e30e49209c" />

I started with `-rate 100` and it worked for a few seconds but then we started getting the `429` responses again. So I toned it waaaay down and ended up going with:
```bash
-rate 10
```
8 minutes later we have a hit:

<img width="1829" height="936" alt="2026-04-25-17:53:03" src="https://github.com/user-attachments/assets/76d33df5-6c75-4fb4-adfe-7763bbeb7ba7" />


## Loot
<img width="1244" height="138" alt="2026-04-25-17:55:21" src="https://github.com/user-attachments/assets/676692b4-52a8-492e-98ab-fba2d2e4379f" />





# I) Subdomains - Virtual Host Enumeration
**Target:**
```http
http://localhost
```

Let's find a wordlist for subdomains:

<img width="1248" height="584" alt="2026-04-26-10:58:23" src="https://github.com/user-attachments/assets/d36cccc7-fe57-4ec3-afad-857a6e9f0f4e" />


I again found an example command from the FFUF repo README which I modified to suit my needs and used the command:
```bash
$ ffuf -c -w subdomains-top1million-110000.txt -u http://localhost -H "Host: FUZZ"
```
We got flooded again with a bunch of fake positives, all with the size of `1495` so we append `-fs 1495` and go again:

<img width="1724" height="1041" alt="2026-04-26-11:22:49" src="https://github.com/user-attachments/assets/b1adaf0a-b9a9-474f-a627-584204e020f2" />


This time no results. So I modified to command to `-H "Host: FUZZ.localhost` and followed along in wireshark:

<img width="1152" height="228" alt="2026-04-26-11:37:59" src="https://github.com/user-attachments/assets/ee7d9e34-598c-4a99-a075-330c6fef1dec" />


It seemed like this should work but we still got no results. I played around with fuzzing different parts of the URL and got nothing in return.

<img width="1897" height="888" alt="2026-04-26-11:51:57" src="https://github.com/user-attachments/assets/32fa5be2-d990-412a-9637-f4ff63ba469b" />

## Realization
I was reading [this](<https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Host>) when it hit me. 

<img width="656" height="129" alt="2026-04-26-13:52:36" src="https://github.com/user-attachments/assets/f2b90261-b38b-4a95-9eb7-48fafa837824" />

I can't use localhost in the header, as it's not a valid domain name.


Okay, so I need a domain name. Well I can't really get it as the server doesn't have a valid domain name, this is just a locally spun up container... I thought I could maybe try to use a hostname, but I didn't have that either. My instict said get the containers IP address and then probe it for a valid hostname somehow.

I got the container address:
```bash
# Only one container running so pressing tab a few times after inspect will complete the name!
$ sudo docker inspect heuristic_swirles | grep -i ip
    "IPAddress": "172.17.0.2",
```
I scanned the target using `nmap`, here's a snippet of the results:

<img width="1048" height="257" alt="2026-04-26-14:27:22" src="https://github.com/user-attachments/assets/793a4d84-bfc7-486e-8780-5e6f41913159" />

I didn't really find anything useful.

## My final hail mary before cheating

<img width="986" height="197" alt="2026-04-26-14:49:29" src="https://github.com/user-attachments/assets/cafffb4b-7c9c-4013-a050-66e4b6eec2a7" />

<img width="1905" height="852" alt="2026-04-26-14:50:44" src="https://github.com/user-attachments/assets/aeab0040-2833-4fb5-b606-1e690db55123" />


## Cheating
On my first task I had done
```bash
$ curl http://localhost/cd/basic
```
to see what it returns. I noticed that on the page it had the **example solution**, so I refrained from doing it again in the name of developing problem solving. But now I was getting desparate.

<img width="1900" height="950" alt="2026-04-26-12:24:59" src="https://github.com/user-attachments/assets/159d863b-0930-4d81-a4e3-ecc5ed6b98bd" />

Well shiiiddd, so we just use `ffuf.me` for the host part... 😂 Should've know this.

<img width="1894" height="985" alt="2026-04-26-12:25:59" src="https://github.com/user-attachments/assets/ffd1188a-e743-43c0-a4b0-f6b066cc092d" />

<img width="950" height="125" alt="2026-04-26-14:09:25" src="https://github.com/user-attachments/assets/5bfab3e9-fc64-46db-8825-5838a68db28a" />

## Question
In case we used the live target => <http://ffuf.me/> we would've obviously known the domain name from jump. But how were we supposed to know it now though? 

Like sure I could/should've guessed it's `ffuf.me`, but without any guessing game involved how do we get to this conclusion?


# It works on my machine

<img width="1901" height="1046" alt="2026-04-26-21:12:27" src="https://github.com/user-attachments/assets/c9553024-14e7-4d22-9ab3-d882e5780c6e" />

# Src & Ref
In order of appearance
- <https://github.com/Ali-Mikael/Application-hacking/blob/main/h2-BreakAndFix.md#x2-fuzzing>
- <https://github.com/Ali-Mikael/Application-hacking/blob/main/h2-BreakAndFix.md#c-dirfuzt-1>
- <https://github.com/BuildHackSecure/ffufme>
- <https://github.com/ffuf/ffuf>
- <https://apidog.com/articles/http-request-parameters-guide/>
- <https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status/429>
- <https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Host>


