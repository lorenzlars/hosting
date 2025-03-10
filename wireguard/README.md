# Wireguard

Install wireguard on a client:
```shell
sudo apt install -y wireguard
```

Generate key pairs on a client:
```shell
wg genkey | tee privatekey | wg pubkey > publickey
```

Setup firewall:
```shell
sudo apt install -y ufw
sudo ufw allow 22
sudo ufw allow 51820
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw enable
```

## Server

Set `net.ipv4.ip_forward=1` to enabled ip forwarding with the following commands:
```shell
sudo nano /etc/sysctl.conf
sudo sysctl -p
sudo systemctl start sysctl
```

This is the config at `/etc/wireguard/wg0.conf`:
```ini                                                                            
[Interface]
Address = 10.0.0.1/24
ListenPort = 51820
PrivateKey = 
PostUp = iptables -I FORWARD -i wg0 -o ens3 -j ACCEPT; iptables -I FORWARD -i ens3 -o wg0 -j ACCEPT; iptables -t nat -I POSTROUTING -o ens3 -j MASQUERADE; iptables -I FORWARD -i wg0 -o wg0 -j ACCEPT
PostDown = iptables -D FORWARD -i wg0 -o ens3 -j ACCEPT; iptables -D FORWARD -i ens3 -o wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o ens3 -j MASQUERADE; iptables -D FORWARD -i wg0 -o wg0 -j ACCEPT

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
systemctl restart wg-quick@wg0
```

## Local Gateway Client

Set `net.ipv4.ip_forward=1` to enabled ip forwarding with the following commands:
```shell
sudo nano /etc/sysctl.conf
sudo sysctl -p
sudo systemctl start sysctl
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
systemctl restart wg-quick@wg0
```

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