# OpenVPN with Docker

**the original project - [jpetazzo/dockvpn](https://github.com/jpetazzo/dockvpn)** and it has its own [automatic build on dockerhub](https://hub.docker.com/r/jpetazzo/dockvpn/).


## Setting up SSH (optional)

Connect to VPS via ssh

```
ssh -l root <ip-address>
```

Setting up new user
```bash
adduser <username>
echo '<username> ALL=(ALL:ALL) ALL' >> /etc/sudoers
```

Adding ssh pubkey to authorized_keys file
```bash
su <username>
mkdir $HOME/.ssh
echo <pubkey> >> $HOME/.ssh/authorized_keys
```

Changing default sshd config file and restart service

```bash
sudo sed -i 's/PubkeyAuthentication no/PubkeyAuthentication yes/g' /etc/ssh/sshd_config
sudo sed -i 's/PasswordAuthentication yes/PasswordAuthentication no/g' /etc/ssh/sshd_config
sudo sed -i 's/PermitRootLogin yes/PermitRootLogin no/g' /etc/ssh/sshd_config

sudo systemctl restart ssh.service
```

Test connection with

```
ssh -l <username> <ip-address>
```

## Requirements

[Docker installation](https://docs.docker.com/engine/install/ubuntu/)

Then add current user to docker group for non root access

```bash
sudo groupadd docker
sudo usermod -aG docker egor
newgrp docker
```

## Starting up VPN + Pihole


```bash
export PIHOLE_PASS=...

docker network create vpn_network --subnet 192.168.1.0/24

# run pihole
docker run \
    --network vpn_network --ip 192.168.1.199 \
    --dns "127.0.0.1" --dns "8.8.8.8" \
    -e TZ="Asia/Yekaterinburg" -e WEBPASSWORD=$PIHOLE_PASS \
    -v "etc-pihole:/etc/pihole" -v "etc-dnsmasq:/etc/dnsmasq.d" \
    -d \
    --name pihole pihole/pihole

# run openvpn
docker run \
    --network vpn_network  \
    -p 1194:1194/udp -p 443:443/tcp \
    -e DNS="192.168.1.199" \
    -v "openvpn:/etc/openvpn" \
    --privileged -d \
    --name openvpn hexlify/vpn
```

To add new client

```bash
docker run -v "openvpn:/etc/openvpn" -p 8081:8081 hexlify/vpn serveconfig <clien_name>
```

## Firewall configuration (optional)

```bash
sudo iptables -A INPUT -p tcp --destination-port 22 -j ACCEPT
sudo iptables -A INPUT -p udp --destination-port 1194 -j ACCEPT
sudo iptables -A INPUT -p tcp --destination-port 443 -j ACCEPT
sudo iptables -I INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -I INPUT -i lo -j ACCEPT
sudo iptables -A INPUT -p tcp --destination-port 8081 -j ACCEPT

# dropping all other packets
sudo iptables -P INPUT DROP
```
