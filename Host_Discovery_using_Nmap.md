When we do a pen test we need to know  what all devices are there /up/alive in the target 77etwork , for that we can use Nmap host discovery option  

Here Nmap uses the ICMP echo requests to check whether the target host is up or not 

An ICMP Echo Request is ==a network packet sent by a device to check if another device on an IP network is online and reachable==

Its just a network packet which check the target device is on  or off

What ever scan u do with Nmap  it better or it compulsory to save those scans for future purposes  , this is because each tools give different scans and we need to compare it , it is always helpful to save these scans 

Now we are going to do a simple Network Range Scan :

We can check our network Subnets using the following commands :

```
ifconfig

ip route show 

ip addr show
```

after getting our network subnet , we can now scan out network range :

```
nmap -sn -oA subnet_scan 192.168.1.0/24 
```

Here this is a simple scanning method which gets all the alive hots with in our network range

![[Pasted image 20260918191218.png]]

Here we have used the flags : 

```
-sn : this is to disable port scan , what nmap does is that only sents ICMP echo requests 

-oA is to save the namp scan results to the file "subnet_scan"
```

Now we are going top add some Linux skills to clean our results 

`namp -sn  -oA subnet_scan 192.168.1.0/24 | grep for | cut -d" " -f5`

![[Pasted image 20260918193001.png]]

==Now our result is more clearer , here  we used the grep command to filter out all the results containing the string **for** , then we cut the lines based on a single space " " , the we printed our the final/5th column using the -f5 command== 

Now imagine if we have a list of IPs that we need to scan , in such scenarios  we can save those IP to a file and we can use that file to scan , we can add some IPs to a file named hosts

![[Pasted image 20260918193710.png]]

We are now going to scan the target using this file :

`namp -sn -oA scan_from_list -iL hosts.txt | grep for | cut -d" " -f5`

![[Pasted image 20260918194120.png]]

Here we have used a list , we  also have an alternative way to specify multiple IP address as well 

`namp -sn -oA multiple_ip_scan 192.168.1.0 192.168.1.1 192.168.1.2 192.168.1.2 | grep for | cut -d" " -f5`

![[Pasted image 20260919033446.png]]

Here we can also specify the IP range in the specific octet

`nmap -sn -oA multiple_ip_scan 192.168.1.0-200 | grep for | cut -d" " -f5`

![[Pasted image 20260919033724.png]]

Now we are going to use a new technique , when using the `-sn` to disable the port scan the nmap will sent **`ICMP echo packets`** , this can we explicitly done using the `-PE `flag , also we can find out what Nmap does in the middle be using the command`--packet-trace `

Nmap have some special character , before sending in the `ICMP echo requests` namp sents out an `ARP ping` packet which in turn receives `ARP reply` , this can be scene using the `--packet-trace` flag , now we can try it out ....

`--packet-trace ` this shows all the packets sent  and received :

`nmap -sn -PE --packet-trace 192.168.1.1`

![[Pasted image 20260919040607.png]]

Now another cool feature from `Nmap` , we can use the `--reason` flag to see why `Nmap` says a particular host is alive or not 

![[Pasted image 20260919041932.png]]

```   
┌──(root㉿kaliVM1)-[~]
└─# nmap -sn -PE --packet-trace --reason -iL hosts.txt 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-18 18:49 -0400
SENT (0.0157s) ARP who-has 192.168.1.9 tell 192.168.1.15
SENT (0.0161s) ARP who-has 192.168.1.16 tell 192.168.1.15
SENT (0.0164s) ARP who-has 192.168.1.8 tell 192.168.1.15
RCVD (0.0168s) ARP reply 192.168.1.8 is-at 18:93:41:D7:EE:5C
RCVD (0.0473s) ARP reply 192.168.1.9 is-at 2A:8C:8C:97:F1:E3
SENT (1.5498s) ARP who-has 192.168.1.16 tell 192.168.1.15
NSOCK INFO [1.6820s] nsock_iod_new2(): nsock_iod_new (IOD #1)
NSOCK INFO [1.6820s] nsock_connect_udp(): UDP connection requested to fe80::1:53 (IOD #1) EID 8
NSOCK INFO [1.6820s] nsock_iod_new2(): nsock_iod_new (IOD #2)
NSOCK INFO [1.6820s] nsock_connect_udp(): UDP connection requested to 192.168.1.1:53 (IOD #2) EID 16
NSOCK INFO [1.6820s] nsock_trace_handler_callback(): Callback: CONNECT SUCCESS for EID 8 [fe80::1:53]
NSOCK INFO [1.6820s] nsock_read(): Read request from IOD #1 [fe80::1:53] (timeout: -1ms) EID 26
NSOCK INFO [1.6820s] nsock_trace_handler_callback(): Callback: CONNECT SUCCESS for EID 16 [192.168.1.1:53]
NSOCK INFO [1.6820s] nsock_read(): Read request from IOD #2 [192.168.1.1:53] (timeout: -1ms) EID 34
NSOCK INFO [1.6820s] nsock_write(): Write request for 42 bytes to IOD #1 EID 43 [fe80::1:53]
NSOCK INFO [1.6820s] nsock_write(): Write request for 42 bytes to IOD #1 EID 51 [fe80::1:53]
NSOCK INFO [1.6820s] nsock_trace_handler_callback(): Callback: WRITE SUCCESS for EID 51 [fe80::1:53]
NSOCK INFO [1.6820s] nsock_trace_handler_callback(): Callback: WRITE SUCCESS for EID 43 [fe80::1:53]
NSOCK INFO [1.7090s] nsock_trace_handler_callback(): Callback: READ SUCCESS for EID 26 [fe80::1:53] (119 bytes)
NSOCK INFO [1.7090s] nsock_read(): Read request from IOD #1 [fe80::1:53] (timeout: -1ms) EID 58
NSOCK INFO [1.7200s] nsock_trace_handler_callback(): Callback: READ SUCCESS for EID 58 [fe80::1:53] (119 bytes)
NSOCK INFO [1.7200s] nsock_read(): Read request from IOD #1 [fe80::1:53] (timeout: -1ms) EID 66
NSOCK INFO [2.2680s] nsock_iod_delete(): nsock_iod_delete (IOD #1)
NSOCK INFO [2.2680s] nevent_delete(): nevent_delete on event #66 (type READ)
NSOCK INFO [2.2680s] nsock_iod_delete(): nsock_iod_delete (IOD #2)
NSOCK INFO [2.2680s] nevent_delete(): nevent_delete on event #34 (type READ)
Nmap scan report for 192.168.1.8
Host is up, received arp-response (0.00035s latency).
MAC Address: 18:93:41:D7:EE:5C (Intel Corporate)
Nmap scan report for 192.168.1.9
Host is up, received arp-response (0.032s latency).
MAC Address: 2A:8C:8C:97:F1:E3 (Unknown)
NSOCK INFO [2.3040s] nsock_iod_new2(): nsock_iod_new (IOD #1)
NSOCK INFO [2.3040s] nsock_connect_udp(): UDP connection requested to fe80::1:53 (IOD #1) EID 8
NSOCK INFO [2.3040s] nsock_iod_new2(): nsock_iod_new (IOD #2)
NSOCK INFO [2.3040s] nsock_connect_udp(): UDP connection requested to 192.168.1.1:53 (IOD #2) EID 16
NSOCK INFO [2.3040s] nsock_trace_handler_callback(): Callback: CONNECT SUCCESS for EID 8 [fe80::1:53]
NSOCK INFO [2.3040s] nsock_read(): Read request from IOD #1 [fe80::1:53] (timeout: -1ms) EID 26
NSOCK INFO [2.3040s] nsock_trace_handler_callback(): Callback: CONNECT SUCCESS for EID 16 [192.168.1.1:53]
NSOCK INFO [2.3040s] nsock_read(): Read request from IOD #2 [192.168.1.1:53] (timeout: -1ms) EID 34
NSOCK INFO [2.3040s] nsock_write(): Write request for 43 bytes to IOD #1 EID 43 [fe80::1:53]
NSOCK INFO [2.3040s] nsock_trace_handler_callback(): Callback: WRITE SUCCESS for EID 43 [fe80::1:53]
NSOCK INFO [2.3360s] nsock_trace_handler_callback(): Callback: READ SUCCESS for EID 26 [fe80::1:53] (120 bytes)
NSOCK INFO [2.3360s] nsock_read(): Read request from IOD #1 [fe80::1:53] (timeout: -1ms) EID 50
NSOCK INFO [2.8660s] nsock_iod_delete(): nsock_iod_delete (IOD #1)
NSOCK INFO [2.8660s] nevent_delete(): nevent_delete on event #50 (type READ)
NSOCK INFO [2.8660s] nsock_iod_delete(): nsock_iod_delete (IOD #2)
NSOCK INFO [2.8660s] nevent_delete(): nevent_delete on event #34 (type READ)
Nmap scan report for 192.168.1.15
Host is up, received localhost-response.
SENT (2.8749s) ICMP [192.168.1.15 > 10.129.2.10 Echo request (type=8/code=0) id=49722 seq=0] IP [ttl=46 id=11164 iplen=28 ]
SENT (2.8752s) ICMP [192.168.1.15 > 10.129.2.11 Echo request (type=8/code=0) id=38483 seq=0] IP [ttl=38 id=32432 iplen=28 ]
SENT (2.8753s) ICMP [192.168.1.15 > 10.129.2.18 Echo request (type=8/code=0) id=13151 seq=0] IP [ttl=52 id=45204 iplen=28 ]
SENT (2.8755s) ICMP [192.168.1.15 > 10.129.2.19 Echo request (type=8/code=0) id=45597 seq=0] IP [ttl=48 id=59467 iplen=28 ]
SENT (2.8757s) ICMP [192.168.1.15 > 10.129.2.20 Echo request (type=8/code=0) id=39451 seq=0] IP [ttl=55 id=54622 iplen=28 ]
SENT (2.8760s) ICMP [192.168.1.15 > 10.129.2.28 Echo request (type=8/code=0) id=54229 seq=0] IP [ttl=48 id=8115 iplen=28 ]
SENT (2.8762s) ICMP [192.168.1.15 > 10.129.2.4 Echo request (type=8/code=0) id=41908 seq=0] IP [ttl=40 id=23393 iplen=28 ]
SENT (3.8761s) ICMP [192.168.1.15 > 10.129.2.10 Echo request (type=8/code=0) id=23819 seq=0] IP [ttl=53 id=18135 iplen=28 ]
SENT (3.8763s) ICMP [192.168.1.15 > 10.129.2.11 Echo request (type=8/code=0) id=63531 seq=0] IP [ttl=51 id=28307 iplen=28 ]
SENT (3.8772s) ICMP [192.168.1.15 > 10.129.2.18 Echo request (type=8/code=0) id=1268 seq=0] IP [ttl=53 id=42894 iplen=28 ]
SENT (3.8774s) ICMP [192.168.1.15 > 10.129.2.19 Echo request (type=8/code=0) id=19490 seq=0] IP [ttl=40 id=45 iplen=28 ]
SENT (3.8778s) ICMP [192.168.1.15 > 10.129.2.20 Echo request (type=8/code=0) id=29806 seq=0] IP [ttl=53 id=10789 iplen=28 ]
SENT (3.8805s) ICMP [192.168.1.15 > 10.129.2.4 Echo request (type=8/code=0) id=2026 seq=0] IP [ttl=59 id=54366 iplen=28 ]
SENT (3.8809s) ICMP [192.168.1.15 > 10.129.2.28 Echo request (type=8/code=0) id=42748 seq=0] IP [ttl=40 id=40683 iplen=28 ]
Nmap done: 11 IP addresses (3 hosts up) scanned in 5.08 seconds

```

Here even though we are telling Nmap to use the ICMP echo request , it just only discovering the host only with ARP requests and replies , so we can disable this by using the flag `--disable-arp-ping`

`nmap -sn -PE -oA apr_disable_scan --reason --disable-arp-ping 192.168.1.1`

![[Pasted image 20260919042537.png]]

![[Pasted image 20260919043005.png]]
