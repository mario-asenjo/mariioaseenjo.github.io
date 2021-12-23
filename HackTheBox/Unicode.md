# Unicode - HackTheBox Writeup
## Author As3di0

> ## Token Key Modification and .....

> # Enueration
> We start sending a ICMP packet to the machine with `ping`.
> ```bash
> PING 10.129.180.130 (10.129.180.130) 56(84) bytes of data.
> 64 bytes from 10.129.180.130: icmp_seq=1 ttl=63 time=41.2 ms
>
> --- 10.129.180.130 ping statistics ---
> 1 packets transmitted, 1 received, 0% packet loss, time 0ms
> rtt min/avg/max/mdev = 41.185/41.185/41.185/0.000 ms
> ```
> So we know the machine is alive. We can start enumerating open ports in the machine, we do this with `nmap`.
>
> ```bash
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
> Now we know there is a web server running under port 80 running `nginx 1.18.0`, so we are going to enumerate that now, as nmap is telling me the title of the web, I'm going to add `hackmedia.htb` to my `/etc/hosts`.
> 
> ![](/Images/Unicode/etcHostHackmedia.png)
> 
> This is the home page for the web server....
> 
> ![](/Images/Unicode/hackmediaHomePage.png)
>
> We are going to register a new account so we can check what kind of content and access we have on the server.
> In order to do this we are going to click `Register` from the upper menu in the home page.
>
> ![](/Images/Unicode/hackmediaRegistro.png)
> 
> Inmediately after creating a new account, we'll be redirected to the `/login` page, where we'll rewrite the credentials we have just created in order to complete login.
> 
> ![](/Images/Unicode/hackmediaLogin.png)
> 
> After login, we'll be redirected to `/dashboard` , where we can check the things we can do to interactuate with the server.
> 
> ![](/Images/Unicode/hackmediaDashboard.png)
> 
> In this case we have three valid options, "Pricing, Upload and Logout".
>
> ![](/Images/Unicode/hackmediaOpciones.png)
> 
> We can check manually each of the options, getting the next sites as a response of each of the options.
>  
> ![](/Images/Unicode/hackmediaPricing.png)
> 
> ![](/Images/Unicode/hackmediaUpload.png)
> 
> I've tried everything with the upload page, but nothing works. I guess the server isn't executing the file I'm uploading so this is nonesense.
> So suddenly I figured out that we have a cookie! Actually a JWT, so we are going to check it out.
> 
> ![](/Images/Unicode/hackmediaCookie.png)
> 
> This is out cookie, whick translated into readable json is....
> 
> ![](/Images/Unicode/jwtDefaultCookie.png) 
>
> So as we can see, we have several fields, but we have special interest in jku and the keys down below (in blue).
> 
> If we search for jku we find this statement...
>> The “jku” (Json Web Key Set URL) Header Parameter is a URI that refers to a resource for a set of JSON-encoded public keys, one of which corresponds to the key used to digitally sign the JWS (JSON Web Signature).
>
> So this jky is pointing somewhere in hackmedia's server, if we try to reach this resource we get the following....
> 
> ![](/Images/Unicode/staticJwks.png)
>
>
