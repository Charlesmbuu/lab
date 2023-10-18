# # KIOPTRIX-LEVEL-1-Walkthrough 

This is a walkthrough from the point of view of a penetration tester of kioptrix level 1 machine found in vulhub.com

We will be searching for possible bugs and vulnerabilites to exploit in the machine.





First, we will have to find what is the IP address of the machine. We can do so by using either nmap or netdiscover. Here I used netdiscover with sudo privileges;
![netdiscover](https://github.com/Charlesmbuu/lab/assets/95209529/8580538c-e427-4414-a25e-293e8cbeb3b7)

Our focus will be on the last two Ips, where we will run them on Nmap
![netdiscover-results](https://github.com/Charlesmbuu/lab/assets/95209529/d3f99a5e-25a9-46fa-958d-3f7b5d0eddb2)







-----------------------------------------------------------------------------
>NMAP
```
sudo nmap -sS -A -D 192.1.1.1 192.168.116.130
```
-sS to run a silent scan so that we cannot be detected by the blue team, by the means of a three-way handshake. 

-A to enable OS detection, version detection, script scanning, and traceroute all ports. 

-D to decoy the real IP with a fake IP.


![nmap-scan-result](https://github.com/Charlesmbuu/lab/assets/95209529/84105bea-f2ef-4a54-b592-a045e381b75f)

There are very interesting ports here; 22 which is the SSH, port 80 which is HTTP website running ( apache 1.3.20 ), 139 smb file sharing protocol with Samba and 443 https Apache server.


I think it’s a good start to investigate what is on the website.

![http-webpage](https://github.com/Charlesmbuu/lab/assets/95209529/26d2f654-bf16-422d-95c4-c901f3b69997)

We have found it running an Apache; We knew this earlier, and on a RedHat Linux Machine.
Let’s try to enumerate vulnerabilities using Nikto;
```
nikto -h https://example.org
```



Let’s focus on SSL which may allow a remote shell, for the scope of this machine.

![nikto-scan](https://github.com/Charlesmbuu/lab/assets/95209529/a7a24b49-c080-46fe-a4ce-f2aa8630e85c)

----------------------------------------------------------------------------------------------



> Directory Busting


We’ll try to enumerate subfolders with either ( dirb, dirbuster, gobuster, ffuf )
I’ll use ffuf

This can be simply done by typing;
```
ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u <IP Address>/FUZZ
```

![ffuf-scan](https://github.com/Charlesmbuu/lab/assets/95209529/9f1c133c-5be5-4e08-b0db-3bf462bb6d89)

----------------------------------------------------------------------------------------------
On running this tool against our target machine, unfortunately it gives back zero results; meaning that there are no directories out there.

![ffuf-results](https://github.com/Charlesmbuu/lab/assets/95209529/92a86006-e514-45fd-a659-bfbf6fb223ba)


----------------------------------------------------------------------------------------------





> SMB(SAMBA) ENUMERATION

My favorites: Enumerating 139 smb port with Metasploit


Try to know what version the smb is running on, search in Metasploit for smb auxiliary modules used for enumeration. Focusing on the results and we found that one that searches for version. 

For me is 97; or you can copy and paste the selection.
![smb-search-msf](https://github.com/Charlesmbuu/lab/assets/95209529/1ec9add5-467c-463b-b994-9732fac3896d)



See options to see needed inputs;
![smb-msf-req-inputs](https://github.com/Charlesmbuu/lab/assets/95209529/c61081b2-807b-43e1-9aef-cb786937a28b)


Assign RHOSTS (Remote Hosts) and start enumeration. You can change the threads for speed.
```
set rhosts <IP Address>
```


To RUN, just type “exploit” or “run”. 


As you see, it runs on samba 2.2.1a. We can search for vulnerabilities in this version.
![smb-version-result](https://github.com/Charlesmbuu/lab/assets/95209529/46e44431-2f90-4885-9afd-0b4308bd57a3)


Searching for vulnerabilities in samba: Samba 2.2.1a
We know that the machine runs on Linux, so 2 is the one.
![samba-2 2-msf-search](https://github.com/Charlesmbuu/lab/assets/95209529/5a3ca34a-1079-4886-8d8b-08a81dd972d9)



Set the RHOSTS and run;

![Samba2 2-search](https://github.com/Charlesmbuu/lab/assets/95209529/531ab7a3-56e0-47ba-aabb-dfd8b6fca2c6)


It keeps on throwing “Meterpreter session _ is not valid and will be closed”
![meterpreter-session-closed](https://github.com/Charlesmbuu/lab/assets/95209529/9012d885-89be-4a53-a96e-eaa6cc89f7a4)



The current payload used is staged, let’s try a non-staged payload…

![setting-to-nonstaged](https://github.com/Charlesmbuu/lab/assets/95209529/d49f529c-7fff-4822-b001-d57466debb1a)

> -------------------------------------------------------------------------------------------

exploit

> -------------------------------------------------------------------------------------------

Congrats! You are now ROOT

![root](https://github.com/Charlesmbuu/lab/assets/95209529/b33c1b5b-39be-4a83-aec0-8f9bfcedcf28)

> -------------------------------------------------------------------------------------------
