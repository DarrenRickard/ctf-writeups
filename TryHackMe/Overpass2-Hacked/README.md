# Overpass 2 - Hacked

### Introduction

This CTF assumes that the Overpass production server was hacked. The goal is to figure out how the attacker was able to get in, hack your way back into the server, and identify any artifacts left behind. 

*(Note: This CTF expects you to be familiar with coding/code analysis, using CLI tools, Linux, and Wireshark, but I will explain the best that I can as we walk through this room.)* <br>

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

---

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

### 1. What's the default hash for the backdoor? <br>

We can answer this question by analyzing the code that the attacker used. In Task 1 Question 4, we identified the link to the malicious code that the attacker utilized. If we copy and paste that link in a web browser, we are presented with a github repo. If we open up `main.go`, we can find a variable that is definitely of interest to this question. <br>
![image](https://user-images.githubusercontent.com/85798849/222906092-b2b2bd2d-a7f4-4b53-8101-52d5dbfaa8e3.png) <br>

### 2. What's the hardcoded salt for the backdoor? <br>

If we analyze the code in main.go, we come across a few functions. Pay close attention to the `verifyPass` function. We notice that it accepts 3 different parameters. One of these parameters happens to be `salt`. Let's now find where this function was utilized. Github allows us to click on user-created functions and easily view any references to the function in the code. Alternatively, you could <kbd>Ctrl</kbd> + <kbd>F</kbd> and type in the function name to find references. <br>
![image](https://user-images.githubusercontent.com/85798849/222906747-b619c7f7-58aa-498e-b586-6ac8d6d99df6.png) <br>
We now see how the function was called in the code and the parameters given.

### 3. What was the hash that the attacker used? - go back to the PCAP for this! <br>

If we go back to the PCAP in wireshark and analyze that previous TCP stream displaying the CLI commands that the attacker had used, we can see that the attacker ran the backdoor program with an option `-a` followed by some long text that looks like it could be a hash. <br>
![image](https://user-images.githubusercontent.com/85798849/222907187-0b7f86af-105c-4b67-886c-91188b142dd9.png) <br>


### 4. Crack the hash using rockyou and a cracking tool of your choice. What's the password? <br>

Let's crack this hash using the rockyou.txt password list and the hashcat cracking tool. But first, we should identify what type of hashing algorithm was possibly used. You can copy and paste the hash onto https://hashes.com/en/tools/hash_identifier <br>

Second, let's identify the `hashtype` option we will have to use with hashcat to successfully crack the hash. <br>
We can refer to the man page for hashcat on the terminal or at https://hashcat.net/wiki/doku.php?id=example_hashes to identify the hashtype we must use. <br>

Finally, In our kali terminal, let's run hashcat using the hashtype, hash:salt, and rockyou.txt <br>

`$ hashcat -m <hashtype> -a 0 -o crack.txt 'hash:salt' /usr/share/wordlists/rockyou.txt --force` <br>
`-m` option is to specify the hash type. <br>
`-a` option is to specify the attack mode. <br>
In this case, we are using `-a 0` which specifies "Straight" mode. Which is more commonly known as a Dictionary attack. <br>
`-o` option is to output the results to a specified file. If the file does not exist, it will be created. <br>
`--force` option is just to ignore warnings. <br>

*(Note: Make sure you're using the hash that the attacker used and not the default hash.)*

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

