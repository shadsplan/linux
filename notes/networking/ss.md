# ss

[ref](https://www.youtube.com/watch?v=phY8Q7Woxsw)

Important, helps you keep an eye on what's connected to your linux server (open ports, open sockets, what's listening, etc.).

 - Port: A unique number given to an application that establishes a network connection. 
 - Socket: A combination of an IP address and a port number.

 ``` bash

# Show all tcp and udp sockets with process info, no DNS resolution, and all states (to cover both listening and established connections)
shad@linux:~/linux/notes/networking$ ss -tupna
Netid         State           Recv-Q          Send-Q                   Local Address:Port                     Peer Address:Port          Process                                             
udp           UNCONN          0               0                           127.0.0.54:53                            0.0.0.0:*                                                                 
udp           UNCONN          0               0                        127.0.0.53%lo:53                            0.0.0.0:*                                                                 
udp           UNCONN          0               0                        10.0.0.4%eth0:68                            0.0.0.0:*                                                                 
udp           UNCONN          0               0                            127.0.0.1:323                           0.0.0.0:*                                                                 
udp           UNCONN          0               0                                [::1]:323                              [::]:*                                                                 
tcp           LISTEN          0               4096                           0.0.0.0:22                            0.0.0.0:*                                                                 
tcp           LISTEN          0               1024                         127.0.0.1:39739                         0.0.0.0:*              users:(("code-7d842fb85a",pid=3020,fd=9))          
tcp           LISTEN          0               4096                     127.0.0.53%lo:53                            0.0.0.0:*                                                                 
tcp           LISTEN          0               4096                        127.0.0.54:53                            0.0.0.0:*                                                                 
tcp           ESTAB           0               0                            127.0.0.1:39739                       127.0.0.1:57258          users:(("code-7d842fb85a",pid=3020,fd=11))         
tcp           ESTAB           0               0                             10.0.0.4:42548                   140.82.113.21:443            users:(("node",pid=1388,fd=34))                    
tcp           ESTAB           0               308                           10.0.0.4:22                       108.28.[REDACTED]:54483                                                             
tcp           ESTAB           0               0                            127.0.0.1:57258                       127.0.0.1:39739                                                             
tcp           LISTEN          0               4096                              [::]:22                               [::]:*                                                                                     

# Above, but only listening sockets
shad@linux:~/linux/notes/networking$ ss -tupnl

# Summary of socket statistics - useful to combine with watch command to monitor in real-time
shad@linux:~/linux/notes/networking$ ss -s
Total: 208
TCP:   13 (estab 8, closed 0, orphaned 0, timewait 0)

Transport Total     IP        IPv6
RAW       1         0         1        
UDP       5         4         1        
TCP       13        12        1        
INET      19        16        3        
FRAG      0         0         0        

# Show only IPv4 sockets, one of them will be the redacted ssh connection above.
shad@linux:~/linux/notes/networking$ ss -4

# Show us only connections related to port 22 (ssh)
shad@linux:~/linux/notes/networking$ ss -tupna '( dport = :22 or sport = :22 )'
Netid       State        Recv-Q       Send-Q             Local Address:Port               Peer Address:Port        Process       
tcp         LISTEN       0            4096                     0.0.0.0:22                      0.0.0.0:*                         
tcp         ESTAB        0            372                     10.0.0.4:22                 108.28.[REDACTED]:54483                     
tcp         LISTEN       0            4096                        [::]:22                         [::]:*                         

# So apache2 doesn't show up here, most likely due to how forked processes work.
# But we run a http server in the background, we can see it:

shad@linux:~$ python3 -m http.server 8080 &
[1] 4006

shad@linux:~$ ss -tupna | grep -E "Netid|8080"
Netid State  Recv-Q Send-Q Local Address:Port     Peer Address:Port Process                                    
tcp   LISTEN 0      5            0.0.0.0:8080          0.0.0.0:*     users:(("python3",pid=4006,fd=3))

# But also, lsof works because it shows processes with open files/sockets:
shad@linux:~$ lsof -i :8080
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
python3 4006 shad    3u  IPv4  33974      0t0  TCP *:http-alt (LISTEN)
```
