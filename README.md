# OpenVPN for Docker

**the original project - [jpetazzo/dockvpn](https://github.com/jpetazzo/dockvpn)** and it has its own [automatic build on dockerhub](https://hub.docker.com/r/jpetazzo/dockvpn/).


Quick instructions:

```bash
CID=$(docker run -d --privileged -p 1194:1194/udp -p 443:443/tcp hexlify/vpn)
docker run -t -i -p 8080:8080 --volumes-from $CID hexlify/vpn serveconfig <client-name>
```