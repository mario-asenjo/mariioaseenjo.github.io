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
