# Docker - Transmission with Wireguard

Transmission running in a docker container with `docker-compose`. The goal is to have transmission run behind a vpn while the system runs with the normal network.

## 1. Docker compose config

In `docker-compose.yml`:
- Replace `/path/to/downloads` with for example `/home/<user>/Transmission/downloads` or somewhere with disk space.
- Replace `/path/to/watch` with for example `/home/<user>/Transmission/watch`.

## 2. Wireguard config

#### _Keys:_

In a directory `~/.wg/`:
- Generate the keys with `wg genkey | tee privatekey | wg pubkey > publickey`.
- Change the rights with `chmod 600 privatekey`.
- Add the public key to the vpn client key manager.

#### _Sample wireguard config file:_

```
[Interface]
PrivateKey = <my-privatekey>    # host private key in ~/.wg/privatekey
Address = <vpn-my-ip>           # host IP assigned by the vpn provider
DNS = <vpn-dns>                 # DNS assigned by the vpn provider

[Peer]
PublicKey = <vpn-publickey>             # vpn server public key
Endpoint = <vpn-hostname>:<vpn-port>    # vpn server address
AllowedIPs = 0.0.0.0/0
```

Once completed the file is placed in `/opt/docker/volumes/wireguard-transmission/config/wg0.conf`.

## 3. Launch the container

```
docker-compose up -d
```

## 4. Launch transmission client

- Use `ip a show dev docker0` to get the container local ip.
- Download `transmission-gtk` or `transmission-qt`.
- Edit > Change Session > Connect to remote session
    - Host: \<container-ip>
    - Port: 9091

## 5. Quality control

#### _Container IP and geolocation:_

Should display the home public ip and geolocation:
```
curl ipinfo.io
```

Should display the vpn server public ip and geolocation:
```
docker exec transmission sh -c 'curl ipinfo.io'
```

#### _Network disabled in case of vpn outage:_

Stop the vpn container:
```
docker stop wireguard-transmission
```

Requesting the ip info should result in an error:
```
docker exec transmission sh -c 'curl ipinfo.io'
```

Clean restart:
```
docker-compose down
docker-compose up -d
```
