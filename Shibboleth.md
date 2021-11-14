# WriteUp For Shibboleth from HackTheBox - Author As3di0

## enumeration

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
