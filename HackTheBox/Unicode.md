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
> This is our cookie, which translated into readable json is....
> 
> ![](/Images/Unicode/jwtDefaultCookie.png) 

> # Token Modification 
> 
> So as we can see, we have several fields, but we have special interest in jku and the keys down below (in blue).
> 
> If we search for jku we find this statement...
>> The “jku” (Json Web Key Set URL) Header Parameter is a URI that refers to a resource for a set of JSON-encoded public keys, one of which corresponds to the key used to digitally sign the JWS (JSON Web Signature).
>
> So this jku is pointing somewhere in hackmedia's server, if we try to reach this resource we get the following....
> 
> ![](/Images/Unicode/staticJwks.png)
>
> This is telling us that we need to serve our own jku, which will contain our public keys so the server thinks we are admin, as our petition is going to get signed as valid.
> 
> So first, we'll need to create a json web key, and we are going to use [mkjwk](https://mkjwk.org/), which is a JSON Web Key Generator.
>
> ![](/Images/Unicode/jwksCreation.png)
>
> So we have now three different keys, we are going to copy the one on the left side, and we are going to use another resource which is [jwk to pem](https://8gwifi.org/jwkconvertfunctions.jsp).
> Now we paste here the copied values and tell it to convert it into a public and a private key.
>  
> ![](/Images/Unicode/jwkToPem.png)
>
> We click "Submit", and we have generated one public and one private key, for us to be able to sign our json web token.
> 
> This are the results:
> 
> ![](/Images/Unicode/jwkGeneratedKeys.png)
> 
> Now that we have the keys, we are going to insert them into the "Verify signature" section in `Jwt.io` like so:
> 
> ![](/Images/Unicode/jwtMitad.png)
> 
> Now we have signed the token, but we still to change the jku. As hackmedia.htb is the one allow listed address in which the token is going to be verified, we will craft the jku from this address, redirecting it to our machine in which we are going to be hosting a web server serving a jwks.json file
> We are going to save the content of hackmedia.htb/static/jwks.json into a file, and we are going to change the "n" parameter, inserting in it the value from the right column from [mkjwk](https://mkjwk.org/), which is:
> 
> ![](/Images/Unicode/jwkN.png)
> 
>> ’n’ : modulus value for the RSA public key.
>
> This is the crafted file, which we are going to serve with `python3`.
> 
> ![](/Images/Unicode/jwksLocal.png)
> 
> Now we only need to modify the username and the jku to redirect our machine for hackmedia to search for the `jwks.json` file in our attacker's machine.
> 
> This is the crafted json web token we need in order to impersonate admin in the webserver:
> 
> ![](/Images/Unicode/jwtFull.png)
> 
> Now we can copy this new token and insert it into our auth field in the cookies from hackmedia.htb.
> 
> When we do this we refresh the webpage and Boom! 
> 
> ![](/Images/Unicode/hackmediaNewDashboard.png)
>
> As you can see, we get a hit to our python server, getting the jwks.json file we have created, so the token is working correctly, and in our browser we see a new dashboard, with some new options to check.
> 
> There we have it! We are now logged in as admin user, let's check what can we do as admin users, and try to get that initial foothold on the machine!
 
> # Foothold
>
> We notice in the left menu, we have two saved reports, if we click one of them we are going to be redirected to, for example:
>
> ![](/Images/Unicode/monthly1.png)
> 
> As we can see it's searching for some sort of file in the server, we could try to get LFI this way but....
>
> ![](/Images/Unicode/monthlyTryLFI.png)
> ![](/Images/Unicode/monthlyTryFailed.png)
>
> As we can see the server applys input filtering so we are going to search for a way to bypass this filtering.
> Trying common ways like inserting `....//....//....//....//etc/paswdd` doesn't give us the file we want so, I've started searching for information on filtering bypass, and found this resource: [HackTricks Unicode Nomalization Vuln](https://book.hacktricks.xyz/pentesting-web/unicode-normalization-vulnerability).
> Here they tell us that different appearence unicode's characters, mean in binary code the same, so we can tell the browser to search for example for : `︰(U+FE30) in the form of	"︰/︰/︰/etc/passwd"`	and the server will respond with the value for `"../../../etc/passwd"`.
> If we try and hit this, we will be able to see the server contents!
> 
> ![](/Images/Unicode/searchLFI.png)
> ![](/Images/Unicode/LFICompleto.png)
>
> As we can see we get the `/etc/passwd`file from the server checking that the only user with a bash shell is `code`.
> 
> Taking into account that this is a nginx server, and after trying to fuzz the possible LFI files we could access, I checked the default files for nginx and found one that actually worked, `/etc/nginx/sites-available/default`.
> 
> ![](/Images/Unicode/sitesAvailable.png)
> 
> ![](/Images/Unicode/sitesAvailable.png)
> As we can see we have a password change for a user, and a file containing this change, located in `/home/code/coder/db.yaml`, so we are going to check that resource....
> 
> ![](/Images/Unicode/dbYaml.png)
> 
> There we go, we have SQL credentials for user code.
> We could try to SSH that user with that password and actually log into the machine as user code, so that's awesome, now we have a foothold! 
>
> ![](/Images/Unicode/sshConfirm.png)
> 
> # Privilege Escalation
> 
> 
> 
> a
