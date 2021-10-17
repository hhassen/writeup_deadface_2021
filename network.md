## Network Traffic Analysis
For this part, I used Wireshark to open the PCAP file.

### Monstrum ex Machina
**Task:**
Our person on the "inside" of Ghost Town was able to plant a
packet sniffing device on Luciafer's computer. Based on our
initial analysis, we know that she was attempting to hack a
computer in Lytton Labs, and we have some idea of what she
was doing, but we need a more in-depth analysis. This is where
YOU come in.

We need YOU to help us analyze the packet capture. Look for
relevant data to the potential attempted hack.

To gather some information on the victim, investigate the
victim's computer activity. The "victim" was using a search
engine to look up a name. Provide the name with standard
capitalization: `flag{Jerry Seinfeld}`.

To solve this call, I filtered http traffic and I looked for different requests made to find the requests related to search engine. I found a request with this name : Charles Geschickter. That's it.

![http traffic wireshark](https://raw.githubusercontent.com/hhassen/writeup_deadface_2021/main/images/monstrum_ex_machina.png)


### Release the Crackin'
**Task:** Luciafer cracked a password belonging to the victim. Submit the flag as `flag{password}`.

First, I had no idea about what to do. So, I unlocked 2 hints. One of them said that the password was transferred in plain using `non secure` protocol. The other hint, said that when the password was typed correctly, a plain text is returned showing that connection is ok.

So, I guessed that might be FTP protocol. I searched deeply and I found that Luciafer tried to crack the FTP password by using a lot of passwords until one of them works. I scrolled down until I found the connection established and I looked for the last password sent.

![FTP password wireshark](https://raw.githubusercontent.com/hhassen/writeup_deadface_2021/main/images/tcp_passwd.png)

Now as I solved it, I found a faster way. That's to filter for successful FTP response (230) : `ftp.response.code == 230`, then follow the TCP stream (Right click and select **Follow | TCP Stream**) of that communication to find the password entered.

### Luciafer, You Clever Little Devil
**Task:** Luciafer gains access to victim's computer by using the cracked password. What is the packet number of the response by victim's system, saying the access is granted ?

As I solved the previous challenge, this becomes easy. It's just the packet number of the 230 FTP response.

### The Sum Of All Fears

**Task:** Players will search for binaries with identical names in the packet capture file, but different extensions: lytton-crypt.exe and lytton-crypt.bin. They need to extract these and take the md5 has of them.

I searched the file names using the `ftp` filter. Then, I followed the TCP stream for each file (Right click and select **Follow | TCP Stream**). To extract the file content from the TCP stream, select **RAW** for the format, then **Save As**.