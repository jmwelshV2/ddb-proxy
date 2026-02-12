# ddb-proxy (Modified)

This proxy allows communication with D&D Beyond (DDB) for integration into **Foundry VTT** via **DDB Importer**.

## Features

Provides backend functionality for:

* Characters
* Spells
* Items
* Monsters
* Campaigns

> **Note:** This is a cut down MVP implementation meant for individual use. It does **not** implement caching and should not be run as a public service for others.

---

## Requirements

* **Node.js 18+**
* **npm** (included with Node)
* Foundry VTT with DDB Importer

> Yarn is **optional**. You can use **npm** instead.

---

## Install

From the project directory:

```bash
npm install
```

(If you prefer Yarn and have it installed, `yarn install` also works.)

---

## Run

This modified version supports configurable host/port via environment variables:

* `HOST` (default: `127.0.0.1`)
* `PORT` (default: `3000`)

### Run locally (local machine only)

```bash
node index.js
```

Binds to `127.0.0.1:3000` and is only reachable from the same machine.

### Run on LAN (reachable from other devices on your network)

```bash
HOST=0.0.0.0 PORT=3000 node index.js
```

Binds to all IPv4 interfaces, enabling LAN/WLAN access.

---

## Verify it is running

Open in a browser or use curl:

```bash
curl http://YOUR_IP:3000/ping
```

Expected response:

```text
pong
```

Examples:

* Local: `http://localhost:3000/ping`
* LAN: `http://192.168.1.50:3000/ping`

---

## Configure Foundry VTT / DDB Importer

### DDB Importer v3.1.26+

Enable **custom proxy** in the settings menu and set the endpoint address to:

```text
http://YOUR_IP:3000
```

> **Important:** Do **not** include a trailing slash.
>
> ✅ `http://example.com:3000`
>
> ❌ `http://example.com:3000/`

### Older versions (manual settings)

In Foundry’s developer console (or browser dev console if hosting remotely):

```js
game.settings.set("ddb-importer", "custom-proxy", true);
game.settings.set("ddb-importer", "api-endpoint", "http://YOUR_IP:3000");
```

Example (local):

```js
game.settings.set("ddb-importer", "custom-proxy", true);
game.settings.set("ddb-importer", "api-endpoint", "http://localhost:3000");
```

To revert to MrPrimate’s proxy:

```js
game.settings.set("ddb-importer", "api-endpoint", "https://proxy.ddb.mrprimate.co.uk");
game.settings.set("ddb-importer", "custom-proxy", false);
```

---

## Windows + WSL2 Notes

If you run the proxy inside **WSL2**, you may need Windows firewall and/or portproxy rules for LAN access.

### 1) Install Node + npm in WSL

```bash
sudo apt update
sudo apt install -y nodejs npm
```

Verify:

```bash
node -v
npm -v
```

### 2) Install dependencies in WSL

```bash
cd /mnt/c/Users/<your-user>/ddb-proxy
npm install
```

### 3) Start the proxy in WSL

```bash
HOST=0.0.0.0 PORT=3000 node index.js
```

Leave this terminal open.

### 4) Allow inbound port on Windows Firewall (PowerShell Admin)

Open **Windows PowerShell as Administrator** (not WSL) and run:

```powershell
New-NetFirewallRule -DisplayName "ddb-proxy 3000" -Direction Inbound -Action Allow -Protocol TCP -LocalPort 3000
```

### 5) If LAN still cannot reach it (WSL2 portproxy)

Get the WSL IP (run inside WSL):

```bash
ip addr show eth0 | grep 'inet ' | awk '{print $2}' | cut -d/ -f1
```

Then in **PowerShell (Admin)**:

```powershell
netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=3000 connectaddress=<WSL_IP> connectport=3000
```

Verify:

```powershell
netsh interface portproxy show all
```

Remove (if needed):

```powershell
netsh interface portproxy delete v4tov4 listenaddress=0.0.0.0 listenport=3000
```

---

## Ubuntu / Debian Linux (Native Server)

### Install Node + npm

```bash
sudo apt update
sudo apt install -y nodejs npm
```

### Install dependencies

```bash
npm install
```

### Run (LAN)

```bash
HOST=0.0.0.0 PORT=3000 node index.js
```

### Firewall (UFW, if enabled)

```bash
sudo ufw allow 3000/tcp
sudo ufw status
```

---

## Router Port Forwarding (Optional)

If you forward `3000` from the internet to your proxy host:

```text
WAN:3000 -> ProxyHostLANIP:3000
```

⚠️ **Warning:** This exposes an HTTP service to the public internet.

---

## TLS / HTTPS (Strongly Recommended when not purely local)

If you are not running purely local, run the proxy behind SSL/TLS (e.g., **Caddy**).

Example `Caddyfile`:

```caddy
yourdomain.com {
  reverse_proxy 127.0.0.1:3000
}
```

Then use:

```text
https://yourdomain.com
```

---

## Troubleshooting

### `Cannot find module 'express'`

You didn’t install dependencies in the environment you’re running:

```bash
npm install
```

### `ss -lntp | grep :3000` shows nothing

The process isn’t running (or you pressed `Ctrl+C`). Start it again and keep it running.

### LAN request times out

Usually firewall/WSL2 port forwarding:

* Windows: add firewall rule and possibly `netsh portproxy`
* Linux: allow `3000/tcp` in UFW/firewalld

### Connection refused

The service is reachable but not listening on that interface/port. Ensure you started with:

```bash
HOST=0.0.0.0 PORT=3000 node index.js
```

---

## Support

No support is provided for this software.
