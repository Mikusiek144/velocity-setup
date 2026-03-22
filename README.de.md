# velocity-setup DEUTSCH

![Minecraft](https://img.shields.io/badge/Minecraft-Network-green?style=for-the-badge)
![Proxy](https://img.shields.io/badge/Proxy-Velocity-blue?style=for-the-badge)
![Java](https://img.shields.io/badge/Java-17+-orange?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Stable-brightgreen?style=for-the-badge)

> Vollständige Anleitung zur Einrichtung von **Velocity** auf einem VPS oder Hosting für ein Minecraft-Servernetzwerk

---

## 📡 Architektur (so funktioniert es)

```id="w1v9kq"
Spieler
  ↓
Velocity (Proxy)
  ↓
Lobby / BoxPvP / Survival (Backends)
```

Velocity fungiert als Vermittler, das heißt: Der Spieler verbindet sich nur mit dem Proxy, und dieser übernimmt den Rest.

---

## 📌 Was ist Velocity?

**Velocity** ist ein moderner Proxy, der vom Paper-Team entwickelt wurde.

In der Praxis:

* Der Spieler verbindet sich mit dem Proxy
* Der Proxy authentifiziert den Spieler
* Der Proxy leitet ihn an den entsprechenden Server weiter

→ Das Backend (Paper/Spigot) sollte niemals öffentlich zugänglich sein

---

## 💡 Warum lohnt es sich, Velocity zu nutzen?

* ✔️ deutlich bessere Leistung als BungeeCord
* ✔️ moderne Sicherheitsmechanismen (Forwarding + Secret)
* ✔️ getrennte Backends (höhere Sicherheit)
* ✔️ Unterstützung neuer Minecraft-Versionen
* ✔️ aktiv weiterentwickeltes Projekt

Kurz gesagt: **der Standard für moderne Minecraft-Netzwerke**

---

## Anforderungen

* Java 17+
* VPS oder Hosting (z. B. Pterodactyl)
* Velocity `.jar`
* Backends (Paper / Purpur / Spigot)

---

## Installation (VPS / Hosting)

### Dateistruktur

```id="x9c2lm"
/server/
  ├── velocity/   ← Proxy
  ├── lobby/      ← Backend
  ├── boxpvp/     ← Backend
```

### Hosting (z. B. Pterodactyl)

* Velocity = separater Server
* Jedes Backend = eigene Instanz

**Ports:**

* Proxy → `25565` (öffentlich)
* Backend → `25566+` (privat)

⚠️ Backends dürfen NICHT öffentlich zugänglich sein

### VPS (Linux)

Proxy starten:

```bash id="t7n4pe"
screen -S velocity
java -Xms1G -Xmx1G -jar velocity.jar
```

---

## ⚙️ Proxy-Konfiguration (Velocity)

Datei:

```id="g3p8zr"
velocity.toml
```

### Weiterleitung (AM WICHTIGSTEN)

```toml id="k6q1as"
player-info-forwarding-mode = "modern"
```

### Was bedeutet das?

Velocity übermittelt dem Backend die Spielerdaten:

* UUID
* IP-Adresse
* Skin

Aber nicht ungesichert, sondern:

* signiert (MAC)
* durch ein Secret geschützt

→ Dadurch kann sich niemand als Spieler ausgeben

### Secret (Sicherheit)

```toml id="m2b7cx"
forwarding-secret-file = "forwarding.secret"
```

* → Die Datei wird automatisch generiert
* → MUSS im Backend identisch sein

### Backends hinzufügen

```toml id="z8v5hd"
[servers]
lobby = "IP:25566"
boxpvp = "IP:25567"
```

→ Bei VPS kannst du `127.0.0.1` verwenden
→ Bei Hosting nutze die IP aus dem Control Panel

### Startserver

```toml id="r4n6yu"
try = [
  "lobby"
]
```

---

## ⚡ So funktioniert die Verbindung (Schritt für Schritt)

Das ist wichtig, da die meisten Probleme durch Missverständnisse entstehen.

### 1. Der Spieler verbindet sich mit dem Proxy

* gibt die Server-IP ein
* gelangt zu Velocity
* Velocity führt die Authentifizierung durch

### 2. Erstellung der Spielerdaten

Velocity bereitet vor:

* UUID
* IP-Adresse
* Profil (Skin)

Die Daten sind signiert und gesichert

### 3. Verbindung zum Backend

Velocity verbindet sich mit dem Backend und übermittelt die Daten

Backend:

* überprüft das `secret`
* wenn korrekt → Spieler wird zugelassen
* wenn nicht → `not forwarded`

### 4. Weiterleitung

Der Spieler wird weitergeleitet zu:

* einem Server aus `try` (z. B. Lobby)

---

## ⚙️ Backend-Konfiguration (Paper)

### Neue Versionen (paper-global.yml)

```yaml id="y5k9td"
proxies:
  velocity:
    enabled: true
    online-mode: true
    secret: "TU_SECRET"
```

### Ältere Versionen (paper.yml)

```yaml id="p1w7nr"
settings:
  velocity-support:
    enabled: true
    online-mode: true
    secret: "TU_SECRET"
```

### server.properties

```properties id="c8j2qx"
online-mode=false
```

→ Die Authentifizierung übernimmt der Proxy, daher darf das Backend dies nicht tun

### Zusätzlich

```yaml id="l3v6hs"
# spigot.yml
settings:
  bungeecord: false
```

---

## 🔒 Sicherheitsmaßnahmen (MUST HAVE)

Wenn du das nicht einstellst, kann der Proxy umgangen werden.

* ✔️ Firewall (Backends blockieren)
* ✔️ Backend nur für den Proxy erreichbar
* ✔️ Keine öffentliche IP der Backends

---

## Starten

Die Reihenfolge ist wichtig:

1. Backends (Paper)
2. Velocity
3. Verbindung über den Proxy herstellen

---

## Testen, ob alles funktioniert

* ✔️ Login über den Proxy möglich
* ✔️ Weiterleitung in die Lobby funktioniert
* ✔️ `/server boxpvp` funktioniert
* ✔️ Direktes Verbinden mit dem Backend ist NICHT möglich

→ Wenn alles passt → ist die Konfiguration korrekt

---

## ❌ Häufigste Fehler

### Player info forwarding failed

* → falsches Secret

### Disconnected: not forwarded

* → Velocity nicht im Backend konfiguriert

### Backend läuft ohne Proxy

* → keine Firewall eingerichtet

### Keine Verbindung zum Backend

* → falsche IP oder falscher Port

---

## 🧰 Schnelle Checkliste (Debug)

* [ ] Secret ist identisch
* [ ] Backend hat `online-mode=false`
* [ ] Velocity nutzt `modern`
* [ ] Backend läuft
* [ ] Port ist korrekt
* [ ] Firewall blockiert den Proxy nicht
* [ ] Backend ist nicht öffentlich

---

## 📎 Weitere Informationen

* Velocity ersetzt das Backend NICHT – es ist nur ein Proxy
* Jedes Backend ist ein eigener Minecraft-Server
* Kommunikation zwischen Servern erfordert Plugins (z. B. Messaging / Redis)

---

## Autor

**noisy144 / mikusiek144**

* GitHub: https://github.com/mikusiek144
* Discord: noisy144

## Support

Wenn dir dieser Leitfaden geholfen hat:

* Gib einen Stern ⭐
* Teile ihn
* Hast du ein Problem? Schreib auf Discord
---
