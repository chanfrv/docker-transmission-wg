# Docker - Transmission

Transmission running in a docker container with `docker-compose`. The goal is to have transmission run behind a vpn while the system runs with the normal network.

## 1. Docker compose config

Example directory `~/Transmission`:
```
.
├── docker-compose.yml
└── wg/
```

In `docker-compose.yml`:
- Replace `/path/to/downloads` with for example `/home/\<user>/Transmission/downloads`
- Replace `/path/to/watch` for example `/home/\<user>/Transmission/watch`

## 2. Wireguard config

In `~/Transmission/wg/` generate the keys with `wg genkey | tee privatekey | wg pubkey > publickey` and add the public key in the vpn client.

```
.
├── docker-compose.yml
└── wg/
    ├── privatekey
    └── publickey
```

Sample wireguard config file to place in `/opt/docker/volumes/wireguard-transmission/config/wg0.conf` to fill with the vpn provider info:

```
[Interface]
PrivateKey = <my-pubkey>    # host private key in wg/privatekey
Address = <vpn-my-ip>       # host IP assigned by the vpn provider
DNS = <vpn-my-dns>          # DNS assigned by the vpn provider

[Peer]
PublicKey = <vpn-privkey>               # vpn server public key
Endpoint = <vpn-address>:<vpn-port>     # vpn server address
AllowedIPs = 0.0.0.0/0
```

## 3. Launch the container

```
cd ~/Transmission
docker-compose up -d
```

## 4. Launch transmission client

- Use `ip a show dev docker0` to get the container local ip.
- Download `transmission-gtk` or `transmission-qt`
- Edit > Change Session > Connect to remote session
    - Host: \<container-ip>
    - Port: 9091

## 5. Quality control

### Container IP and geolocation

Should display the home public ip and geolocation:
```
curl ipinfo.io
```

Should display the vpn public ip and geolocation:
```
docker exec transmission sh -c 'curl ipinfo.io'
```

### Network disabled in case of vpn outage

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
