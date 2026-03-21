# velocity-setup ENGLISH

![Minecraft](https://img.shields.io/badge/Minecraft-Network-green?style=for-the-badge)
![Proxy](https://img.shields.io/badge/Proxy-Velocity-blue?style=for-the-badge)
![Java](https://img.shields.io/badge/Java-17+-orange?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Stable-brightgreen?style=for-the-badge)

> Complete guide to configuring **Velocity** on a VPS / hosting for a Minecraft server network

---

## 📡 Architecture (how it works)

```
Player
  ↓
Velocity (proxy)
  ↓
Lobby / BoxPvP / Survival (backends)
```

Velocity acts as an intermediary—the player connects only to it, and it handles the rest.

---

## 📌 What is Velocity?

**Velocity** is a modern proxy created by the Paper team.

In practice:

* the player connects to the proxy
* the proxy authenticates the player
* the proxy redirects them to the appropriate server

→ The backend (Paper/Spigot) should never be publicly accessible.

---

## 💡 Why use Velocity?

- ✔️ Much better performance than BungeeCord  
- ✔️ Modern security (forwarding + secret)  
- ✔️ Backends are separated (greater security)  
- ✔️ Supports new versions of Minecraft  
- ✔️ Actively developed project  

In short: **the standard in modern Minecraft networks**

---

## Requirements

* Java 17+
* VPS or hosting (e.g., Pterodactyl)
* Velocity `.jar`
* Backends (Paper / Purpur / Spigot)

---

## Installation (VPS / hosting)

### File structure

```
/server/
  ├── velocity/   ← proxy
  ├── lobby/      ← backend
  ├── boxpvp/     ← backend
```

### Hosting (e.g., Pterodactyl)

* Velocity = separate server
* Each backend = separate instance

**Ports:**

* Proxy → `25565` (public)
* Backend → `25566+` (private)

⚠️ Do NOT expose backends publicly

### VPS (Linux)

Starting the proxy:

```bash
screen -S velocity
java -Xms1G -Xmx1G -jar velocity.jar
```

---

## ⚙️ Proxy configuration (Velocity)

File:

```
velocity.toml
```

### Forwarding (MOST IMPORTANT)

```toml
player-info-forwarding-mode = “modern”
```

### What does this actually do?

Velocity forwards player data to the backend:

* UUID
* IP
* skin

But not “just like that”—only:

* signed (MAC)
* protected by a secret

→ This ensures no one can impersonate a player

### Secret (security)

```toml
forwarding-secret-file = “forwarding.secret”
```

- → The file is generated automatically
- → MUST be identical in the backend

### Adding backends

```toml
[servers]
lobby = “IP:25566”
boxpvp = “IP:25567”
```

→ If you have a VPS → you can use `127.0.0.1`
→ If you’re using hosting → use the IP from your control panel

### Startup server

```toml
try = [
  “lobby”
]
```

---

## ⚡ How the connection works (step by step)

This is important because most issues stem from misunderstandings.

### 1. The player connects to the proxy

* enters the server IP
* is directed to Velocity
* Velocity performs authentication

### 2. Creating player data

Velocity prepares:

* UUID
* IP
* profile (skin)

The data is signed and secured.

### 3. Connection to the backend

Velocity connects to the backend and sends the data.

Backend:

* checks the `secret`
* if OK → lets the player in
* if NOT → `not forwarded`

### 4. Redirection

The player is redirected to:

* a server with `try` (e.g., lobby)

## ⚙️ Backend configuration (Paper)

### New versions (paper-global.yml)

```yaml
proxies:
  velocity:
    enabled: true
    online-mode: true
    secret: “TU_SECRET”
```

### Older versions (paper.yml)

```yaml
settings:
  velocity-support:
    enabled: true
    online-mode: true
    secret: “TU_SECRET”
```

### server.properties

```properties
online-mode=false
```

→ The proxy handles authentication, meaning the backend cannot do so

### Additionally

```yaml
# spigot.yml
settings:
  bungeecord: false
```

---

## 🔒 Security (MUST HAVE)

If you don’t do this, someone will bypass the proxy.

- ✔️ firewall (backend blocking)
- ✔️ backend accessible only via the proxy
- ✔️ no public IP for backends

---

## Startup

The order matters:

1. Backends (Paper)
2. Velocity
3. You connect via the proxy

---

## Test if it works

- ✔️ You can log in via the proxy
- ✔️ You reach the lobby
- ✔️ `/server boxpvp` works
- ✔️ You CANNOT access the backend directly

→ If everything works → you have configured it correctly

---

## ❌ Most common errors

### Player info forwarding failed

- → Incorrect secret


### Disconnected: not forwarded

- → backend does not have velocity in its config


### Backend works without a proxy

- → no firewall


### Cannot connect to the backend

- → wrong IP / port

---

## 🧰 Quick checklist (debug)

* [ ] secret is identical
* [ ] backend has `online-mode=false`
* [ ] velocity has `modern`
* [ ] backend is running
* [ ] port is correct
* [ ] firewall is not blocking the proxy
* [ ] backend is not public

---

## 📎 Additional information

* Velocity does NOT replace the backend—it is only a proxy
* Each backend is a separate Minecraft server
* Communication between servers requires plugins (e.g., messaging / Redis)

---

## Author

**noisy144 / mikusiek144**

* GitHub: https://github.com/mikusiek144
* Discord: noisy144

## Support

If this guide was helpful:

*  Leave a star on the repository
*  Share it
*  Having trouble? Message us on Discord

---
