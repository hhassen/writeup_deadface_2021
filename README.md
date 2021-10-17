# Writeup for DEADFACE CTF

This weekend (October 17 2021) I participated in DEADFACE CTF 2021 edition. An event brought to by CyberUp, Cyber Hacktics, and United States Air Force veterans in support of National Cyber Security Awareness Month.

I really enjoyed this CTF, especially the accessiblity of challenges for beginners and the smooth graduation of difficulities through different challenges.
I wish I had more time to try all the different challenges proposed.

![logo_deadface_2021](https://raw.githubusercontent.com/hhassen/writeup_deadface/main/images/logo_deadface_2021.png "logo deadface ctf")

## SQL challenges
![https://github.com/hhassen/writeup_deadface/blob/main/sql.md](https://github.com/hhassen/writeup_deadface/blob/main/sql.md)


## Network Traffic Analysis
![https://github.com/hhassen/writeup_deadface/blob/main/network.md](https://github.com/hhassen/writeup_deadface/blob/main/network.md)

## Programming challenges
### The Count
**Task:** We are given a hostname and a port, and we need to find a flag.

First thing to do, a simple netcut to that server `nc code.deadface.io 50000`.
The servers return us a word and wants us to calculate the sum of the word. If a = 0, b = 1, c = 2, etc.. Tell me what the sum of this word is ?
We have to do that in less than 5 seconds.

For that I developed a Python script using pwn library to make socket communication with the server.
```python
#!/usr/bin/env python3

from pwn import *
import re

def count(word):
    sum = 0
    for ch in word:
        sum += ord(ch) - 97
    return sum

r = remote("code.deadface.io", 50000)
while True:
    string = r.recv(1024)
    string = string.decode("utf-8")
    print(string)
    word = re.search("Your word is: (.*)\n", string).group(1)
    print(word)
    print(count(word))
    word_count = count(word)
    r.send(str(word_count).encode())
```

![count program](https://raw.githubusercontent.com/hhassen/writeup_deadface/main/images/count_program.png)


## Exploitation challenges
### Old Devil
**Task:** We are given a ELF file that demands to put the correct demon name in order to print the flag.

First, I have tried `strings` on that file. Nothing special was found.
So, I used `ltrace` to find the `strcmp` function. 
```
ltrace ./demon
```

![ltrace](https://raw.githubusercontent.com/hhassen/writeup_deadface/main/images/ltrace.png)

### Password insecurities
**Task:** We are given a hashed password and we need to crack it.

I used John The Ripper to crack the hashed password using Rockyou dictionnary:
`john --wordlist=/opt/wordlist/rockyou.txt hashed_passwords.txt`

PS : to show previously hacked passwords with John The Ripper :
`john --show hashed_passwords.txt`
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
### You shall not pass
**Task:** We need to crack a password of a user, given her answers to some personal questions.

![Fb Screenshot](https://raw.githubusercontent.com/hhassen/writeup_deadface/main/images/fb_screenshot.png)

**Disclaimer:** I wasn't able to crack the password, but I will share my approach anyways.

First, I put all the personal anwsers in a text file (I used an online OCR to make things faster). I tried to use `john` using that text file dictionnary. But, it didn't work.
Then, I tought to make a new dictionary file that combines words from her personal answers. That's called a **combinator attack**.

Contrary to `hashcat`, `john` doesn't handle combinator attacks within it. I looked for tools to be able to generate the new dictionnary. I found a list of different tools [here](https://miloserdov.org/?p=6032)
Personnaly, I used [princeprocessor](https://github.com/hashcat/princeprocessor) to generate the new dictionnary.

Unfortunately, `john` wasn't able to crack the password again. Maybe, I need to handle uppercase and lowercase to solve that..


**Usefull resource**
[Write up by TheZealOt (creator of ctf)](https://github.com/docsewell/DEADFACE-CTF-2021)