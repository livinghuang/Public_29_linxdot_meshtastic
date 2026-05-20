# meshtastic-bridge — linxdot install bundle

Pre-built container + compose + hotplug rule for running
[project 08 NAFCO Meshtastic USB Receiver](https://github.com/livinghuang/08_nafco_sqc485i_meshtastic_usb_receiver)
on a linxdot device, with MQTT publish to the existing ChirpStack
mosquitto broker.

> [繁體中文版本見下方](#中文版本) ↓

---

## What's in each release bundle

| File | Purpose |
| --- | --- |
| `meshtastic-bridge-image.tar.gz` | Pre-built docker image (aarch64) |
| `docker-compose.meshtastic.yml` | Compose v1-compatible service definition |
| `etc-hotplug.d-usb-30-sqc485i` | OpenWrt hotplug rule for SQC485I stable symlink |
| `install.sh` | One-shot installer (run as root on linxdot) |
| `README.md` | Per-release notes |

Latest: <https://github.com/livinghuang/Public_29_linxdot_meshtastic/releases/latest>

## Target host requirements

- linxdot device, **aarch64** (ARM64), OpenWrt + BusyBox
- Docker 20.10+ with legacy `docker-compose` v1
- `kmod-usb-acm` installed (default on the standard linxdot image)
- chirpstack-docker stack already up (provides the
  `chirpstack-docker_default` network + `mosquitto` service name)
- A SQC485I (ESP32-C3 + SX1262 + Meshtastic firmware) ready to plug
  into a USB host port

## Install — Linux / macOS

```sh
# 1. Download the release bundle (replace VERSION with e.g. v0.1.0)
curl -LO https://github.com/livinghuang/Public_29_linxdot_meshtastic/releases/download/VERSION/meshtastic-bridge-VERSION-linxdot-aarch64.tar.gz

# 2. Copy to linxdot
scp meshtastic-bridge-VERSION-linxdot-aarch64.tar.gz root@<linxdot-ip>:/tmp/

# 3. SSH in and install
ssh root@<linxdot-ip>
cd /tmp
tar -xzf meshtastic-bridge-VERSION-linxdot-aarch64.tar.gz
cd meshtastic-bridge-VERSION
sh install.sh
```

## Install — Windows PowerShell

Windows 10 (1809+) and Windows 11 ship OpenSSH (`ssh` + `scp`) by
default, so the exact same flow works in PowerShell. The four commands
after `scp` run on linxdot, not Windows — they require nothing extra.

```powershell
# 0. (one time, if scp/ssh not present) Settings → Apps → Optional features → Add "OpenSSH Client"

# 1. Download the bundle into your current directory (or use the browser)
Invoke-WebRequest -Uri "https://github.com/livinghuang/Public_29_linxdot_meshtastic/releases/download/VERSION/meshtastic-bridge-VERSION-linxdot-aarch64.tar.gz" -OutFile meshtastic-bridge-VERSION-linxdot-aarch64.tar.gz

# 2. Copy to linxdot
scp meshtastic-bridge-VERSION-linxdot-aarch64.tar.gz root@<linxdot-ip>:/tmp/

# 3. SSH in and install
ssh root@<linxdot-ip>
cd /tmp
tar -xzf meshtastic-bridge-VERSION-linxdot-aarch64.tar.gz
cd meshtastic-bridge-VERSION
sh install.sh
```

Or — copy + install + exit in one PowerShell block:

```powershell
$ver    = "v0.1.0"
$bundle = "meshtastic-bridge-$ver-linxdot-aarch64.tar.gz"
$dir    = "meshtastic-bridge-$ver"
$ip     = "192.168.0.85"   # ← replace with your linxdot IP
scp $bundle root@${ip}:/tmp/
ssh root@$ip "cd /tmp && tar -xzf $bundle && cd $dir && sh install.sh"
```

### Windows gotchas

- **OpenSSH not found** → Settings → Apps → Optional features → Add "OpenSSH Client" (or use PuTTY/WinSCP).
- **Password prompt every time** → drop your public key into `C:\Users\<you>\.ssh\id_ed25519.pub`, then `ssh-copy-id root@<linxdot-ip>` (Windows 11 PowerShell has this; older Win10 may need a manual append into `~/.ssh/authorized_keys` on linxdot).
- **Path separators** — PowerShell accepts both `\` and `/`; the `:/tmp/` part of the `scp` target is the **remote** path on linxdot and is always `/`.
- **Defender / SmartScreen** may flag `scp`; allow and continue.

## After install

- Web UI: `http://<linxdot-ip>:9090/`
- MQTT (anonymous): `mosquitto_sub -h <linxdot-ip> -t 'meshtastic/#' -v`
- Health: `curl http://<linxdot-ip>:9090/api/health`
- Logs: `ssh root@<linxdot-ip> 'docker logs -f meshtastic-bridge --tail 100'`

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

- Private source repo: <https://github.com/livinghuang/29_linxdot_meshtastic>
- Upstream NAFCO USB receiver: <https://github.com/livinghuang/08_nafco_sqc485i_meshtastic_usb_receiver>

---

## 中文版本

把 [08 NAFCO Meshtastic USB Receiver](https://github.com/livinghuang/08_nafco_sqc485i_meshtastic_usb_receiver) 原封不動 deploy 到 linxdot 上，加掛 MQTT publisher 把 mesh 事件同步送到 linxdot 既有的 ChirpStack mosquitto broker。本 repo 只負責**派發已 build 好的安裝包**，每次 release 一份 aarch64 docker image + compose + hotplug + 一鍵安裝腳本。

### 每個 release bundle 內容

| 檔案 | 用途 |
| --- | --- |
| `meshtastic-bridge-image.tar.gz` | 預 build 好的 docker image（aarch64） |
| `docker-compose.meshtastic.yml` | compose v1 兼容的 service 定義 |
| `etc-hotplug.d-usb-30-sqc485i` | OpenWrt hotplug rule，建 SQC485I 穩定 symlink |
| `install.sh` | 一鍵安裝腳本（在 linxdot 上 root 執行） |
| `README.md` | 該版本說明 |

最新 release：<https://github.com/livinghuang/Public_29_linxdot_meshtastic/releases/latest>

### 目標主機需求

- linxdot device，**aarch64**（ARM64），OpenWrt + BusyBox
- Docker 20.10+ 與 legacy `docker-compose` v1
- 已裝 `kmod-usb-acm`（標準 linxdot image 已內含）
- 主 chirpstack-docker stack 已起來（提供 `chirpstack-docker_default` 網路與 `mosquitto` service name）
- 一顆燒好 Meshtastic 韌體的 SQC485I（ESP32-C3 + SX1262），準備接到 USB

### 安裝 — Linux / macOS

```sh
# 1. 下載安裝包（把 VERSION 換成例如 v0.1.0）
curl -LO https://github.com/livinghuang/Public_29_linxdot_meshtastic/releases/download/VERSION/meshtastic-bridge-VERSION-linxdot-aarch64.tar.gz

# 2. 推到 linxdot
scp meshtastic-bridge-VERSION-linxdot-aarch64.tar.gz root@<linxdot-ip>:/tmp/

# 3. SSH 進去解壓 + 跑 install
ssh root@<linxdot-ip>
cd /tmp
tar -xzf meshtastic-bridge-VERSION-linxdot-aarch64.tar.gz
cd meshtastic-bridge-VERSION
sh install.sh
```

### 安裝 — Windows PowerShell

Windows 10（1809 以後）與 Windows 11 內建 OpenSSH（`ssh` + `scp`），同一份流程 PowerShell 直接跑得起來。`scp` 之後的指令是在 **linxdot 那台 OpenWrt 上**執行，跟 Windows 端無關。

```powershell
# 0.（如果 ssh / scp 不存在）設定 → 應用程式 → 選用功能 → 加入「OpenSSH 用戶端」

# 1. 下載安裝包到目前目錄（或自己用瀏覽器抓）
Invoke-WebRequest -Uri "https://github.com/livinghuang/Public_29_linxdot_meshtastic/releases/download/VERSION/meshtastic-bridge-VERSION-linxdot-aarch64.tar.gz" -OutFile meshtastic-bridge-VERSION-linxdot-aarch64.tar.gz

# 2. 推到 linxdot
scp meshtastic-bridge-VERSION-linxdot-aarch64.tar.gz root@<linxdot-ip>:/tmp/

# 3. SSH 進去解壓 + 跑 install
ssh root@<linxdot-ip>
cd /tmp
tar -xzf meshtastic-bridge-VERSION-linxdot-aarch64.tar.gz
cd meshtastic-bridge-VERSION
sh install.sh
```

或者：copy + install 一次完成：

```powershell
$ver    = "v0.1.0"
$bundle = "meshtastic-bridge-$ver-linxdot-aarch64.tar.gz"
$dir    = "meshtastic-bridge-$ver"
$ip     = "192.168.0.85"   # ← 改成你的 linxdot IP
scp $bundle root@${ip}:/tmp/
ssh root@$ip "cd /tmp && tar -xzf $bundle && cd $dir && sh install.sh"
```

### Windows 常見坑

- **找不到 OpenSSH** → 設定 → 應用程式 → 選用功能 → 加入「OpenSSH 用戶端」；或改用 PuTTY/WinSCP。
- **每次都跳密碼** → 把 public key 放 `C:\Users\<你>\.ssh\id_ed25519.pub`，跑 `ssh-copy-id root@<linxdot-ip>`（Windows 11 PowerShell 有；舊 Win10 需要手動把 key 貼到 linxdot 的 `~/.ssh/authorized_keys`）。
- **路徑分隔符** — PowerShell `\` 跟 `/` 都接受；`scp` 目標 `:/tmp/` 那段是 **linxdot 上的路徑**，永遠用 `/`。
- **Defender / SmartScreen** 偶爾擋 `scp`；放行繼續。

### 安裝完成後

- Web UI：`http://<linxdot-ip>:9090/`
- 訂閱 MQTT（anonymous）：`mosquitto_sub -h <linxdot-ip> -t 'meshtastic/#' -v`
- 健康狀態：`curl http://<linxdot-ip>:9090/api/health`
- Logs：`ssh root@<linxdot-ip> 'docker logs -f meshtastic-bridge --tail 100'`

### 整體架構

```
SQC485I（Meshtastic 韌體）
  ↓ USB Serial
linxdot
  └─ docker container: meshtastic-bridge
       ├─ FastAPI + WebSocket UI 在 :9090
       └─ paho-mqtt publisher → mosquitto:1883（跟 ChirpStack 同個 docker network）
```

NAFCO Sender 透過 LoRa 送來的 Modbus 封包（addr / func / regs）會即時出現在 Web UI 與 MQTT topic 上。

### 重啟正確姿勢

`docker restart` 會觸發 ESP32-C3 native USB-CDC 的快速 DTR 切換、把下次 Meshtastic handshake 撞壞。請用：

```sh
docker stop meshtastic-bridge && sleep 2 && docker start meshtastic-bridge
```

### 程式碼來源

- Private 主 repo：<https://github.com/livinghuang/29_linxdot_meshtastic>
- Upstream NAFCO USB receiver：<https://github.com/livinghuang/08_nafco_sqc485i_meshtastic_usb_receiver>
