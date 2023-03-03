# Overpass 2 - Hacked

### Introduction

This CTF assumes that the Overpass production server was hacked. The goal is to figure out how the attacker was able to get in, hack your way back into the server, and identify any artifacts left behind. 

*(Note: This CTF expects you to be familiar with using CLI tools, Linux, and Wireshark, but I will explain the best that I can as we walk through this room.)* <br>

## Task 1 - *Forensics - Analyse the PCAP*

### 1. What was the URL of the page they used to upload a reverse shell?

So we know that the attacker has uploaded some kind of payload to a web server. Knowing it's some type of interaction with a web server, we can safely assume that the protocol we should filter for in Wireshark is HTTP. <br>

*(Note: HTTP transmits data in plaintext; however, in the real world, you are more likely to run into the encrypted version known as HTTPS as it is far more secure and keeps data private.)* <br>
![image](https://user-images.githubusercontent.com/85798849/222154541-e10bdb04-8dd1-46c6-8779-fb970c6bb297.png)
Now that we've filtered for HTTP traffic, we now see an interesting POST request which contains the URL of the page that the attacker uploaded the payload to. <br>

*(Note: An HTTP POST request is used to send data to a server to create/update a resource.)*

---

### 2. What payload did the attacker use to gain access?
*(Note: A payload refers to malicious programs/code an attacker can use to steal, damage, or gain unauthorized access to data/systems.)* <br>

To find this payload, let's refer to the POST request packet we found earlier. If we right-click the packet and **Follow -> TCP Stream**, we are now presented with the entire TCP conversation to include the plaintext data. <br>
![image](https://user-images.githubusercontent.com/85798849/222185577-96666b95-4be3-4688-9c41-a303e297f837.png)


---

### 3. What password did the attacker use to privesc?

In the last task, we were able to find and observe the payload. If we analyze the payload, it shows that the reverse shell will connect to port 4242. So let's filter for port 4242 to get the unnecessary traffic out of the way using `tcp.port == 4242` <br>
Now, let's go ahead and **Follow -> TCP Stream** on any of the packets. <br>
![image](https://user-images.githubusercontent.com/85798849/222744414-e8ebb640-d184-4e01-b20d-a0641de9983a.png) <br>
We can now easily identify the password that the attacker used to privilege escalate. <br>

*(Note: The PSH flag in the TCP header informs the receiving host that the data should be pushed to the receiving application immediately. In this case, the data being pushed to the application would be the CLI commands that the attacker is entering.)* <br>

---

### 4. How did the attacker establish persistence? 

In the last task, we opened up the TCP stream showing us all of the commands that the attacker had used. <br>
In case you've forgotten, we can filter for `tcp.port == 4242` and **Follow -> TCP Stream** on any of the packets to display the attackers commands. <br>
*Hint: Attackers can establish persistence by creating a backdoor.*

### 5. Using the fasttrack wordlist, how many of the system passwords were crackable? 

In the same TCP stream, we can see that the attacker has dumped the /etc/shadow file. <br>
*(Note: The shadow file is a text file in Linux, accessible only by the root user, that contains usernames, hashed passwords, and other user properties.)* <br>

Copy the shadow file contents and save it in a new text file on your kali machine. Now we can invite our friend "John the Ripper" to try and crack these hashed passwords. <br>
*(Note: John the Ripper is a popular open source password cracking tool)*

```console
# Crack passwords using fasttrack.txt
kali@kali:~/thm/overpass2$ sudo john -wordlist=/usr/share/wordlists/fasttrack.txt shadow.txt
# Show cracked passwords
kali@kali:~/thm/overpass2$ sudo john --show shadow.txt
```
You should now be presented with the cracked hashes! 

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

