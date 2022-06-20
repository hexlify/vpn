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

sudo systemctl restart sshd
```

Test connection with

```
ssh -l <username> <ip-address>
```

## Firewall configuration (optional)

```bash
```


## Starting up VPN


```bash
CID=$(docker run -d --privileged -p 1194:1194/udp -p 443:443/tcp hexlify/vpn)
docker run -t -i -p 8080:8080 --volumes-from $CID hexlify/vpn serveconfig <client-name>
```