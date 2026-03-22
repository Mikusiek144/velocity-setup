# velocity-setup ENGLISH

![Minecraft](https://img.shields.io/badge/Minecraft-Network-green?style=for-the-badge)
![Proxy](https://img.shields.io/badge/Proxy-Velocity-blue?style=for-the-badge)
![Java](https://img.shields.io/badge/Java-17+-orange?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Stable-brightgreen?style=for-the-badge)

> Complete guide to setting up **Velocity** on a VPS or hosting for a Minecraft server network

---

## 📡 Architecture (how it works)

```id="a8k2lm"
Player
  ↓
Velocity (proxy)
  ↓
Lobby / BoxPvP / Survival (backends)
```

Velocity acts as an intermediary — the player connects only to it, and it handles everything else.

---

## 📌 What is Velocity?

**Velocity** is a modern proxy developed by the Paper team.

In practice:

* the player connects to the proxy
* the proxy authenticates the player
* the proxy forwards them to the appropriate server

→ The backend (Paper/Spigot) should never be publicly accessible

---

## 💡 Why use Velocity?

* ✔️ much better performance than BungeeCord
* ✔️ modern security (forwarding + secret)
* ✔️ separated backends (higher security)
* ✔️ supports newer Minecraft versions
* ✔️ actively maintained project

In short: **the standard for modern Minecraft networks**

---

## Requirements

* Java 17+
* VPS or hosting (e.g., Pterodactyl)
* Velocity `.jar`
* Backends (Paper / Purpur / Spigot)

---

## Installation (VPS / hosting)

### File structure

```id="n3p7df"
/server/
  ├── velocity/   ← proxy
  ├── lobby/      ← backend
  ├── boxpvp/     ← backend
```

### Hosting (e.g., Pterodactyl)

* Velocity = separate server
* each backend = separate instance

**Ports:**

* Proxy → `25565` (public)
* Backend → `25566+` (private)

⚠️ Do NOT expose backends publicly

### VPS (Linux)

Starting the proxy:

```bash id="k5v9rt"
screen -S velocity
java -Xms1G -Xmx1G -jar velocity.jar
```

---

## ⚙️ Proxy configuration (Velocity)

File:

```id="x4b1qo"
velocity.toml
```

### Forwarding (MOST IMPORTANT)

```toml id="c7m2zs"
player-info-forwarding-mode = "modern"
```

### What does this actually do?

Velocity forwards player data to the backend:

* UUID
* IP address
* skin

But not blindly — only:

* signed (MAC)
* secured with a secret

→ This prevents anyone from impersonating a player

### Secret (security)

```toml id="p9l6wx"
forwarding-secret-file = "forwarding.secret"
```

* → the file is generated automatically
* → MUST be identical on the backend

### Adding backends

```toml id="t2r8vn"
[servers]
lobby = "IP:25566"
boxpvp = "IP:25567"
```

→ On a VPS → you can use `127.0.0.1`
→ On hosting → use the IP from your control panel

### Startup server

```toml id="h6q3yb"
try = [
  "lobby"
]
```

---

## ⚡ How the connection works (step by step)

This is important, as most issues come from misunderstandings.

### 1. Player connects to the proxy

* enters the server IP
* connects to Velocity
* Velocity performs authentication

### 2. Player data preparation

Velocity prepares:

* UUID
* IP address
* profile (skin)

The data is signed and secured

### 3. Connection to the backend

Velocity connects to the backend and sends the data

Backend:

* checks the `secret`
* if valid → allows the player in
* if not → `not forwarded`

### 4. Redirection

The player is redirected to:

* a server from `try` (e.g., lobby)

---

## ⚙️ Backend configuration (Paper)

### New versions (paper-global.yml)

```yaml id="v1k8dm"
proxies:
  velocity:
    enabled: true
    online-mode: true
    secret: "TU_SECRET"
```

### Older versions (paper.yml)

```yaml id="r5n2qc"
settings:
  velocity-support:
    enabled: true
    online-mode: true
    secret: "TU_SECRET"
```

### server.properties

```properties id="y7m4hs"
online-mode=false
```

→ The proxy handles authentication, so the backend must not do it

### Additionally

```yaml id="u3w9ap"
# spigot.yml
settings:
  bungeecord: false
```

---

## 🔒 Security (MUST HAVE)

If you skip this, the proxy can be bypassed.

* ✔️ firewall (block backend access)
* ✔️ backend accessible only via the proxy
* ✔️ no public IP for backends

---

## Startup

The order matters:

1. Backends (Paper)
2. Velocity
3. Connect via the proxy

---

## Test if it works

* ✔️ You can log in via the proxy
* ✔️ You reach the lobby
* ✔️ `/server boxpvp` works
* ✔️ You CANNOT connect directly to the backend

→ If everything works → your setup is correct

---

## ❌ Most common errors

### Player info forwarding failed

* → incorrect secret

### Disconnected: not forwarded

* → Velocity is not configured on the backend

### Backend works without a proxy

* → no firewall

### Cannot connect to the backend

* → wrong IP or port

---

## 🧰 Quick checklist (debug)

* [ ] secret is identical
* [ ] backend has `online-mode=false`
* [ ] Velocity uses `modern`
* [ ] backend is running
* [ ] port is correct
* [ ] firewall is not blocking the proxy
* [ ] backend is not public

---

## 📎 Additional information

* Velocity does NOT replace the backend — it is only a proxy
* each backend is a separate Minecraft server
* communication between servers requires plugins (e.g., messaging / Redis)

---

## Author

**noisy144 / mikusiek144**

* GitHub: https://github.com/mikusiek144
* Discord: noisy144

## Support

If this guide was helpful:

* leave a star ⭐
* share it
* having an issue? message me on Discord
---
