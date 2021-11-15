# Shibboleth WriteUp - Author As3di0

Exploiting Zabbix 5.0 and mysql.



## Enumeration

First of all we need to `ping` the machine to see if it's alive.
```
root@as3diosMachine:~# ping -c 1 10.129.99.35
PING 10.129.99.35 (10.129.99.35) 56(84) bytes of data.
64 bytes from 10.129.99.35: icmp_seq=1 ttl=63 time=38.8 ms

--- 10.129.99.35 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 38.839/38.839/38.839/0.000 ms
root@as3diosMachine:~# 

```


Now we are going to search for open ports in the server with `Nmap`.

```
root@as3diosMachine:~/Desktop/HTB/Labs/Shibboleth# nmap -sCV -p80 10.129.99.35 -oN nmap/targeted
Starting Nmap 7.91 ( https://nmap.org ) at 2021-11-14 23:42 GMT
Nmap scan report for 10.129.99.35
Host is up (0.036s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.41
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Did not follow redirect to http://shibboleth.htb/
Service Info: Host: shibboleth.htb

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.97 seconds
```


So we have a web server, running `Apache 2.4.41`, and we have a dns to add to the `/etc/hosts`.

In this moment before running the browser to check the web, I'm starting to enumerate directories and possible virtual hosts in
this machine, so we are going to use `wfuzz` and `gobuster` to do this :


>            |------| Gobuster Directory Search |------|

```

root@as3diosMachine:~/Desktop/HTB/Labs/Shibboleth# gobuster dir -u http://shibboleth.htb/ -w /usr/share/wordlists/dirbuster/directory-list-
2.3-medium.txt -t 50
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://shibboleth.htb/
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/11/14 23:47:28 Starting gobuster in directory enumeration mode
===============================================================
/assets               (Status: 301) [Size: 317] [--> http://shibboleth.htb/assets/]
/forms                (Status: 301) [Size: 316] [--> http://shibboleth.htb/forms/] 
Progress: 3525 / 220561 (1.60%)                                                   ^C
[!] Keyboard interrupt detected, terminating.
                                                                                   
===============================================================
2021/11/14 23:47:45 Finished
===============================================================
```

We have found `/assets` and `/forms`, which is going to give us nothing but headaches. But good to try and see errors, even a possible Directory Transversal,
which I have been inable to exploit. But the deal comes here:


>            |------| Wfuzz Virtual Host Discovery |------|


