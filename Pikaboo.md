# Pikaboo - Hack The Box - Writeup - Author As3di0

> Enumeration
>> First we need to check if machine is alive, we are using `ping`.

>> ```
>> root@as3diosMachine:~/Desktop/HTB/Labs/Pikaboo# ping -c 1 10.129.198.34
>> PING 10.129.198.34 (10.129.198.34) 56(84) bytes of data.
>> 64 bytes from 10.129.198.34: icmp_seq=1 ttl=63 time=33.4 ms
>>
>> --- 10.129.198.34 ping statistics ---
>> 1 packets transmitted, 1 received, 0% packet loss, time 0ms
>> rtt min/avg/max/mdev = 33.367/33.367/33.367/0.000 ms
>> root@as3diosMachine:~/Desktop/HTB/Labs/Pikaboo# 
>> ```
>>
>> Now we now it's alive so we are going to start enumerating opened ports in the machine with `nmap`.


