# Unicode - HackTheBox Writeup
## Author As3di0

> ## Token Key Modification and .....

> # Enueration
> We start sending a ICMP packet to the machine with `ping`.
> ```
> PING 10.129.180.130 (10.129.180.130) 56(84) bytes of data.
> 64 bytes from 10.129.180.130: icmp_seq=1 ttl=63 time=41.2 ms
>
> --- 10.129.180.130 ping statistics ---
> 1 packets transmitted, 1 received, 0% packet loss, time 0ms
> rtt min/avg/max/mdev = 41.185/41.185/41.185/0.000 ms
> ```
> So we know the machine is alive. We can start enumerating open ports in the machine, we do this with `nmap`.
>
> ```
> # Nmap 7.92 scan initiated Sun Dec  5 13:33:04 2021 as: nmap -sCV -p22,80 -oN nmap/targeted 10.129.96.111
> Nmap scan report for 10.129.96.111
> Host is up (0.072s latency).
> 
> PORT   STATE SERVICE VERSION
> 22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
> | ssh-hostkey: 
> |   3072 fd:a0:f7:93:9e:d3:cc:bd:c2:3c:7f:92:35:70:d7:77 (RSA)
> |   256 8b:b6:98:2d:fa:00:e5:e2:9c:8f:af:0f:44:99:03:b1 (ECDSA)
> |_  256 c9:89:27:3e:91:cb:51:27:6f:39:89:36:10:41:df:7c (ED25519)
> 80/tcp open  http    nginx 1.18.0 (Ubuntu)
> |_http-generator: Hugo 0.83.1
> |_http-server-header: nginx/1.18.0 (Ubuntu)
> |_http-title: Hackmedia
> Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
>
> Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
> # Nmap done at Sun Dec  5 13:33:14 2021 -- 1 IP address (1 host up) scanned in 10.12 seconds
> ```
> Now we know there is a web server running under port 80 running `nginx 1.18.0`, so we are going to enumerate that now.
[Image from Hackmedia's home page](/Images/hackmediaHomePage)
> 
> ![](/Images/etcHostsHackmedia)
> 
