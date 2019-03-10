# B2 Réseau 2018 - TP2


#### Routage statique


```sh
# net1 à net2 via Router 1
[ervin@client1 ~]$ ip route add 10.2.2.0/24 via 10.2.1.254 dev enp0s8
# net2 à net1 via Router2
[ervin@server1 ~]$ ip route add 10.2.1.0/24 via 10.2.2.254 dev enp0s8
# Router1 à net2 via Router2 
[ervin@router1 ~]$ ip route add 10.2.2.0/24 via 10.2.12.3 dev enp0s9
# Router2 à net1 via Router1
[ervin@router2 ~]$ ip route add 10.2.1.0/24 via 10.2.12.2 dev enp0s9
```

Test de fonctionnement des interfaces

- Client 1 : Ping **Server1**

```sh
[ervin@client1 ~]$ ping -c 4 server1
64 bytes from server1.net2.b2 (10.2.2.10): icmp_seq=1 ttl=62 time=1.39 ms
64 bytes from server1.net2.b2 (10.2.2.10): icmp_seq=2 ttl=62 time=1.19 ms
64 bytes from server1.net2.b2 (10.2.2.10): icmp_seq=3 ttl=62 time=1.09 ms
64 bytes from server1.net2.b2 (10.2.2.10): icmp_seq=4 ttl=62 time=1.20 ms

4 packets transmitted, 4 received, 0% packet loss, time 3012ms
```

- Server 1 : Ping **Client1**

```sh 
[ervin@server1 ~]$ ping -c 4 client1
64 bytes from client1.net1.b2 (10.2.1.10): icmp_seq=1 ttl=62 time=1.36 ms
64 bytes from client1.net1.b2 (10.2.1.10): icmp_seq=2 ttl=62 time=1.37 ms
64 bytes from client1.net1.b2 (10.2.1.10): icmp_seq=3 ttl=62 time=1.21 ms
64 bytes from client1.net1.b2 (10.2.1.10): icmp_seq=4 ttl=62 time=1.92 ms

4 packets transmitted, 4 received, 0% packet loss, time 3017ms
```

- **Router1** ne ping pas **Server1**, car **Server1** qui est dans le réseau **net2** ne connait pas l'adresse du réseau **net12** dans lequel se trouve **Router1** et donc ne peux pas lui répondre. 

---

#### Visualisation du routage avec Wireshark

**Client1** ping **Server1**

- <a href="https://github.com/Ervin11/b2-net-tp2/blob/master/Captures/capture-routage-net12.pcap">Capture pour l'interface qui est dans net12</a>
- <a href="https://github.com/Ervin11/b2-net-tp2/blob/master/Captures/capture-routage-net2.pcap">Capture pour l'interface qui est dans net2</a>

La différence est que pour :
- Le réseau **net12** : **router1** demande qui a l'IP qui correspond à **router2** pour pouvoir rediriger le ping vers l'adresse de **server1**.
- Le réseau **net2** : **router2** demande qui a l'IP qui correspond à **server1** pour transmettre le ping.

---

#### NAT

Pour que curl google.com fonctionne, j'ai édité le fichier /etc/resolv.conf, j'y ai simplement ajouté **nameserver 8.8.8.8**.
- Client 1
```sh
[ervin@client1 ~]$ curl google.com
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.com/">here</A>.
</BODY></HTML>
```
- Server 1
```sh
[ervin@server1 ~]$ curl google.com
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.com/">here</A>.
</BODY></HTML>
```
- Router 1
```sh
[ervin@router1 ~]$ curl google.com
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.com/">here</A>.
</BODY></HTML>
```
- Router 2
```sh
[ervin@router2 ~]$ curl google.com
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.com/">here</A>.
</BODY></HTML>
```
---

#### NTP Server

- Router 1

```sh
[root@router1 ervin] chronyc sources
210 Number of sources = 4
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^* clients0.arcanite.ch          2   6   377    13  +1199us[+1814us] +/-   60ms
^+ pob01.aplu.fr                 2   6   377    17  +1200us[+1812us] +/-   52ms
^+ siberut.ordimatic.net         2   6   377    17  +1861us[+2472us] +/-   55ms
^+ master.servme.fr              3   6   377    16  +1923us[+2536us] +/-   65ms

[root@router1 ervin] chronyc tracking
Reference ID    : 25BB49B2 (clients0.arcanite.ch)
Stratum         : 3
Ref time (UTC)  : Sun Mar 10 22:28:57 2019
System time     : 0.000835854 seconds fast of NTP time
Last offset     : +0.000289100 seconds
RMS offset      : 0.001136065 seconds
Frequency       : 21.455 ppm fast
Residual freq   : +0.124 ppm
Skew            : 7.886 ppm
Root delay      : 0.045474123 seconds
Root dispersion : 0.035563808 seconds
Update interval : 65.3 seconds
Leap status     : Normal

```

- Router 2

```sh

[ervin@router2 ~]$ chronyc sources
210 Number of sources = 1
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^? router1.net12.b2              0   7     0     -     +0ns[   +0ns] +/-    0ns

[ervin@router2 ~]$ chronyc tracking
Reference ID    : 7F7F0101 ()
Stratum         : 10
Ref time (UTC)  : Sun Mar 10 22:30:15 2019
System time     : 0.000000000 seconds fast of NTP time
Last offset     : +0.000000000 seconds
RMS offset      : 0.000000000 seconds
Frequency       : 0.000 ppm slow
Residual freq   : +0.000 ppm
Skew            : 0.000 ppm
Root delay      : 0.000000000 seconds
Root dispersion : 0.000000000 seconds
Update interval : 0.0 seconds
Leap status     : Normal

```

- Server 1

```sh
[ervin@server1 ~]$ chronyc sources
210 Number of sources = 1
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^? router1.net12.b2              0   7     0     -     +0ns[   +0ns] +/-    0ns

[ervin@server1 ~]$ chronyc tracking
Reference ID    : 7F7F0101 ()
Stratum         : 10
Ref time (UTC)  : Sun Mar 10 22:31:10 2019
System time     : 0.000000000 seconds fast of NTP time
Last offset     : +0.000000000 seconds
RMS offset      : 0.000000000 seconds
Frequency       : 0.000 ppm slow
Residual freq   : +0.000 ppm
Skew            : 0.000 ppm
Root delay      : 0.000000000 seconds
Root dispersion : 0.000000000 seconds
Update interval : 0.0 seconds
Leap status     : Normal

```

#### Web server

```sh
[ervin@server1 ~]$ sudo ss -altnp4
State       Recv-Q Send-Q                                            Local Address:Port                                                           Peer Address:Port              
LISTEN      0      128                                                           *:80                                                                        *:*                   users:(("nginx",pid=3743,fd=6),("nginx",pid=3742,fd=6))
LISTEN      0      128                                                           *:22                                                                        *:*                   users:(("sshd",pid=3055,fd=3))
LISTEN      0      100                                                   127.0.0.1:25                                                                        *:*                   users:(("master",pid=3179,fd=13))

```

- Accès au serveur depuis client 2

```sh
[ervin@client2 ~]$ curl server1
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">

<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en">
    <head>
        <title>Test Page for the Nginx HTTP Server on Fedora</title>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
        <style type="text/css">
            /*<![CDATA[*/
            body {
                background-color: #fff;
                color: #000;
                font-size: 0.9em;
                font-family: sans-serif,helvetica;
                margin: 0;
                padding: 0;
            }
            :link {
                color: #c00;
            }
            :visited {
                color: #c00;
            }
            a:hover {
                color: #f50;
            }

```
