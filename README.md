# forward2wireguard

### enable forwarding
```
sysctl -w net.ipv4.ip_forward=1
sysctl -w net.ipv6.conf.all.forwarding=1
```

### default
```
sudo iptables -P FORWARD DROP
```

### add
```
sudo iptables -A FORWARD -i wg1 -o eth0 -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A FORWARD -i eth0 -o wg1 -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# port 80
sudo iptables -A FORWARD -i eth0 -o wg1 -p tcp --syn --dport 80 -m conntrack --ctstate NEW -j ACCEPT
sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j DNAT --to-destination x.y.z.100:80
sudo iptables -t nat -A POSTROUTING -o wg1 -p tcp --dport 80 -d x.y.z.100 -j SNAT --to-source x.y.z.1
```

### del
```
sudo iptables -D FORWARD -i wg1 -o eth0 -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
sudo iptables -D FORWARD -i eth0 -o wg1 -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# port 80
sudo iptables -D FORWARD -i eth0 -o wg1 -p tcp --syn --dport 80 -m conntrack --ctstate NEW -j ACCEPT
sudo iptables -t nat -D PREROUTING -i eth0 -p tcp --dport 80 -j DNAT --to-destination x.y.z.100:80
sudo iptables -t nat -D POSTROUTING -o wg1 -p tcp --dport 80 -d x.y.z.100 -j SNAT --to-source x.y.z.1
```

### info
```
sudo iptables -L
sudo iptables -t nat -L
```

### wireguard
```
[Interface]
...
PostUp = /etc/wireguard/helper/add-nat-routing.sh
PostDown = /etc/wireguard/helper/remove-nat-routing.sh

[Peer]
...

$ chmod -v +x /etc/wireguard/helper/*.sh
```
