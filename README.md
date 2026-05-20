# meshtastic-bridge v0.1.0 — linxdot install bundle

Pre-built container + compose + hotplug rule for running
[project 08 NAFCO Meshtastic USB Receiver](https://github.com/livinghuang/08_nafco_sqc485i_meshtastic_usb_receiver)
on a linxdot device, with MQTT publish to the existing ChirpStack
mosquitto broker.

## What's in this bundle

| File | Purpose |
| --- | --- |
| `meshtastic-bridge-image.tar.gz` | Pre-built docker image (aarch64, ~150 MiB unpacked) |
| `docker-compose.meshtastic.yml` | Compose v1-compatible service definition |
| `etc-hotplug.d-usb-30-sqc485i` | OpenWrt hotplug rule for SQC485I stable symlink |
| `install.sh` | One-shot installer (run as root on linxdot) |
| `README.md` | This file |

## Target host

- linxdot device (aarch64, OpenWrt + BusyBox + Docker 20.10+)
- legacy `docker-compose` v1 installed
- `kmod-usb-acm` installed
- chirpstack-docker stack already running (creates the
  `chirpstack-docker_default` network this service joins)

## Install

```sh
scp meshtastic-bridge-v0.1.0-linxdot-aarch64.tar.gz root@<linxdot-ip>:/tmp/
ssh root@<linxdot-ip>
cd /tmp
tar -xzf meshtastic-bridge-v0.1.0-linxdot-aarch64.tar.gz
cd meshtastic-bridge-v0.1.0
sh install.sh
```

After install:
- UI: `http://<linxdot-ip>:9090/`
- MQTT topics: `meshtastic/<gateway>/{packet/<from_id>,status,diag/<kind>}` on `<linxdot-ip>:1883`

## What it does

```
SQC485I (Meshtastic firmware)
  ↓ USB Serial
linxdot
  └─ docker container: meshtastic-bridge
       ├─ FastAPI + WebSocket UI on :9090
       └─ paho-mqtt publisher → mosquitto:1883 (same docker network as ChirpStack)
```

Real-time decoded NAFCO Modbus payload (addr / func / regs) shown on
both the web UI and the MQTT topic.

## Restart safely

`docker restart` triggers a fast USB DTR transition that ESP32-C3
native USB-CDC handles badly — Meshtastic handshake will time out on
the next start. Use:

```sh
docker stop meshtastic-bridge && sleep 2 && docker start meshtastic-bridge
```

## Source

Source repository (private): https://github.com/livinghuang/29_linxdot_meshtastic
