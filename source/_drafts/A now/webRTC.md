# WebRTC



## coturn

- docker-compose.yml

```yaml
version: '3'
services:
  coturn_server:
    image: instrumentisto/coturn:4.5.1
    restart: always
    network_mode: "host"
    volumes:
      - ./turnserver.conf:/etc/coturn/turnserver.conf
```

- turnserver.conf

```conf
# TURN server name and realm
realm=a
server-name=b

# Use fingerprint in TURN message
fingerprint

# IPs the TURN server listens to
listening-ip=0.0.0.0

# External IP-Address of the TURN server
external-ip=42.192.50.232

# Main listening port
listening-port=3478

# Further ports that are open for communication
min-port=10000
max-port=20000

# Log file path
log-file=/var/log/turnserver.log

# Enable verbose logging
verbose

# Specify the user for the TURN authentification
user=test:test123

# Enable long-term credential mechanism
lt-cred-mech

# SSL certificates
#cert=/etc/letsencrypt/live/<DOMAIN>/cert.pem
#pkey=/etc/letsencrypt/live/<DOMAIN>/privkey.pem

# 443 for TURN over TLS, which can bypass firewalls
#tls-listening-port=443
```



### 验证Coturn的可用性

- [webrtc.github.io](https://webrtc.github.io/samples/src/content/peerconnection/trickle-ice/)

#### 验证stun
输入 stun:x.x.x.x:3478
得到结果

```
Time	Component	Type	Foundation	Protocol	Address	Port	Priority
0.002	1	host	886443856	udp	10.80.1.131	49469	126 | 32542 | 255
0.104	1	host	2052453280	tcp	10.80.1.131	9	90 | 32542 | 255
0.288	1	srflx	2643034245	udp	112.27.203.124	49469	100 | 32542 | 255
0.312	                                        Done
0.313
```

看到srflx后面就是你的电脑的外网IP，表示打洞成功。

#### 验证turn
输入 turn:x.x.x.x:3478,username:,password
得到结果

```
0.003	1	host	886443856	udp	10.80.1.131	55831	126 | 32542 | 255
0.104	1	host	2052453280	tcp	10.80.1.131	9	90 | 32542 | 255
0.534	1	srflx	2643034245	udp	112.27.203.124	55831	100 | 32542 | 255
0.614	1	relay	3676437432	udp	x.x.x.x	56631	2 | 32542 | 255
0.878	                                       Done
0.880
```
看到relay后面就是你的服务器的外网IP，表示可以使用Coturn的turn服务器进行转发。同时也可以看见srflx，这说明了turn服务是stun的一个拓展，turn和stun是包含的关系。





## pion/ion-sfu

- docker run

```shell
$ docker run -d -p 7000:7000 -p 5000-5020:5000-5020/udp pionwebrtc/ion-sfu:v1.7.7-jsonrpc
```





## 参考文章

- [How to set up and configure your own TURN server using Coturn (gabrieltanner.org)](https://gabrieltanner.org/blog/turn-server)

- [Building a WebRTC video and audio Broadcaster in Golang using ION-SFU, and media devices (gabrieltanner.org)](https://gabrieltanner.org/blog/broadcasting-ion-sfu)

- [What, Why and How | WebRTC for the Curious](https://webrtcforthecurious.com/docs/01-what-why-and-how/)

  

