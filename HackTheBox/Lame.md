# Lame Hack The Box - Writeup
## Author As3di0

> # Enumeration
> We start by sending ICMP to check connectivity with the machine, we are going to be using `ping`.
> 
> ```
> PING 10.129.177.242 (10.129.177.242) 56(84) bytes of data.
> 64 bytes from 10.129.177.242: icmp_seq=1 ttl=63 time=44.5 ms
>
> --- 10.129.177.242 ping statistics ---
> 1 packets transmitted, 1 received, 0% packet loss, time 0ms
> rtt min/avg/max/mdev = 44.539/44.539/44.539/0.000 ms
> ```
>
> Now that we now we have connectivity with it, we are going to start enumerating open ports, and running services on that opened ports, we'll be using `Nmap`.
> 
> ```
> Nmap 7.91 scan initiated Thu Oct  7 16:36:38 2021 as: nmap -sC -sV -p21,22,139,445,3632 -oN nmap/targeted 10.129.1.113
> Nmap scan report for 10.129.1.113
> Host is up (0.065s latency).
>
> PORT     STATE SERVICE      VERSION
> 21/tcp   open  ftp          vsftpd 2.3.4
> |_ftp-anon: Anonymous FTP login allowed (FTP code 230)
> | ftp-syst: 
> |   STAT: 
> | FTP server status:
> |      Connected to 10.10.14.108
> |      Logged in as ftp
> |      TYPE: ASCII
> |      No session bandwidth limit
> |      Session timeout in seconds is 300
> |      Control connection is plain text
> |      Data connections will be plain text
> |      vsFTPd 2.3.4 - secure, fast, stable
> |_End of status
> 22/tcp   open  ssh          OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
> | ssh-hostkey: 
> |_  1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
> 139/tcp  open  netbios-ssn  Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
> 445/tcp  open  microsoft-ds Samba smbd 3.0.20-Debian
> 3632/tcp open  distccd      distccd v1 ((GNU) 4.2.4 (Ubuntu 4.2.4-1ubuntu4))
> Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
> 
> Host script results:
> |_clock-skew: mean: 2h01m54s, deviation: 2h49m45s, median: 1m51s
> | smb-os-discovery: 
> |   OS: Unix (Samba 3.0.20-Debian)
> |   Computer name: lame
> |   NetBIOS computer name: 
> |   Domain name: hackthebox.gr
> |   FQDN: lame.hackthebox.gr
> |_  System time: 2021-10-07T11:40:54-04:00
> | smb-security-mode: 
> |   account_used: guest
> |   authentication_level: user
> |   challenge_response: supported
> |_  message_signing: disabled (dangerous, but default)
> |_smb2-time: Protocol negotiation failed (SMB2)
> 
> Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
> # Nmap done at Thu Oct  7 16:39:38 2021 -- 1 IP address (1 host up) scanned in 180.17 seconds
> ```
> 
> We check for publicly available exploits for `vsftpd 2.3.4`, we'll be using `searchsploit`.
>
> ![](/Images/Lame/usermapSearch.png)
> 
> We find this exploit, which is built in python3, that we are going to copy the raw source code and create a file in our local machine, so we can try it with out victim's machine.
> 
> ![](/Images/Lame/usermapRaw.png)
> 
> Now we are going to check how it works and we are going to fill out the arguments it needs in order to work fine, but before we hit it, we are going to start a `nc` listener in the port we are going to be specifying to usermap, so when we launch it, it will hopefully send a reverse shell to it.
> 
> ![](/Images/Lame/usermapCompleto.png)
> 
> And here is what we get back in the listener we have just spawned!
> 
> ![](/Images/Lame/NcCompleto.png)
> 
> We have a user named Makis, that's where the user flag is going to be located, and then the root flag is as usual in `/root/` directory, we `cat`both files to check the flags, and voila!
>
> ![](/Images/Lame/lameFlags.png)
>  
> ### PWNED!
> Thanks for joining!
>
> 
> 
> 
> a
