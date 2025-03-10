# Wireguard

Install wireguard on a client:
```shell
sudo apt install -y wireguard
```

Generate key pairs on a client:
```shell
sudo -i
cd /etc/wireguard
wg genkey | tee privatekey | wg pubkey > publickey
exit
```

## Server

Setup firewall:
```shell
sudo apt install iptables -y iptables-persistent
sudo nano /etc/iptables/rules.v4
```

```shell
*filter
:INPUT DROP [0:0]
:FORWARD DROP [0:0]
:OUTPUT ACCEPT [0:0]

# Allow established connections
-A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# Allow SSH
-A INPUT -p tcp --dport 22 -j ACCEPT

# Allow ICMP
-A INPUT -p icmp -j ACCEPT
-A FORWARD -p icmp -j ACCEPT

# Allow loopback interface
-A INPUT -i lo -j ACCEPT
-A OUTPUT -o lo -j ACCEPT

# Drop all other
-A INPUT -j DROP
-A FORWARD -j DROP

COMMIT
```

```shell
sudo iptables-restore < /etc/iptables/rules.v4
```

Set this to enabled ip forwarding and disable ipv6 with the following commands:
```shell
# /etc/sysctl.conf
net.ipv4.ip_forward = 1
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
```

> [!WARNING]
> Make sure this will be applied after reboot!

```shell
sudo nano /etc/sysctl.conf
sudo sysctl -p
```

This is the config at `/etc/wireguard/wg0.conf`:
```ini                                                                            
[Interface]
Address = 10.0.0.1/24
ListenPort = 51820
PrivateKey = 
PostUp = iptables -I INPUT -p udp --dport 51820 -j ACCEPT; iptables -I FORWARD -i wg0 -o ens3 -j ACCEPT; iptables -I FORWARD -i ens3 -o wg0 -j ACCEPT; iptables -I FORWARD -i wg0 -o wg0 -j ACCEPT; iptables -t nat -I POSTROUTING -o ens3 -j MASQUERADE
PostDown = iptables -D INPUT -p udp --dport 51820 -j ACCEPT; iptables -D FORWARD -i wg0 -o ens3 -j ACCEPT; iptables -D FORWARD -i ens3 -o wg0 -j ACCEPT; iptables -D FORWARD -i wg0 -o wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o ens3 -j MASQUERADE

# Gateway Client
[Peer]
PublicKey = 
AllowedIPs = 10.0.0.2/32, 192.168.10.0/24

# Normal Client
[Peer]
PublicKey = 
AllowedIPs = 10.0.0.3/32
```

Activate the wireguard service
```shell
sudo systemctl enable wg-quick@wg0
sudo systemctl start wg-quick@wg0
```

## Local Gateway Client

Setup firewall:
```shell
sudo apt install iptables -y iptables-persistent
sudo nano /etc/iptables/rules.v4
```

```shell
*filter
:INPUT DROP [0:0]
:FORWARD DROP [0:0]
:OUTPUT ACCEPT [0:0]

# Allow established connections
-A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# Allow SSH
-A INPUT -p tcp --dport 22 -j ACCEPT

# Allow ICMP
-A INPUT -p icmp -j ACCEPT
-A FORWARD -p icmp -j ACCEPT

# Allow loopback interface
-A INPUT -i lo -j ACCEPT
-A OUTPUT -o lo -j ACCEPT

# Drop all other
-A INPUT -j DROP
-A FORWARD -j DROP

COMMIT
```

```shell
sudo iptables-restore < /etc/iptables/rules.v4
```

Set this to enabled ip forwarding and disable ipv6 with the following commands:
```shell
# /etc/sysctl.conf
net.ipv4.ip_forward = 1
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
```

> [!WARNING]
> Make sure this will be applied after reboot!

```shell
sudo nano /etc/sysctl.conf
sudo sysctl -p
```

This is the config at `/etc/wireguard/wg0.conf`:
```ini
[Interface]
PrivateKey = 
Address = 10.0.0.2/32
PostUp = iptables -I FORWARD -i wg0 -o ens3 -j ACCEPT; iptables -I FORWARD -i ens3 -o wg0 -j ACCEPT; iptables -I FORWARD -i wg0 -o wg0 -j ACCEPT; iptables -t nat -I POSTROUTING -o ens3 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -o ens3 -j ACCEPT; iptables -D FORWARD -i ens3 -o wg0 -j ACCEPT; iptables -D FORWARD -i wg0 -o wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o ens3 -j MASQUERADE

# Server
[Peer]
PublicKey = 
AllowedIPs = 10.0.0.0/24
Endpoint = 
PersistentKeepalive = 25
```

Activate the wireguard service
```shell
sudo systemctl enable wg-quick@wg0
sudo systemctl start wg-quick@wg0
```

Optionally you can add a route to your local router to route to allow accessing the vpn clients from your local network.
Just set the Local Gateway Client as gateway for the `10.0.0.0/24` network.


## Normal Client

```ini
[Interface]
PrivateKey = 
Address = 10.0.0.3/32
DNS = 192.168.10.1

# Server
[Peer]
PublicKey = 
AllowedIPs = 10.0.0.0/24, 192.168.10.0/24
Endpoint = 
PersistentKeepalive = 25

```

### Workaround for sysctl
```shell
sudo crontab -e
```

```shell
@reboot /sbin/sysctl --load=/etc/sysctl.conf
```