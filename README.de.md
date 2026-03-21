# velocity-setup DEUTSCH

![Minecraft](https://img.shields.io/badge/Minecraft-Network-green?style=for-the-badge)
![Proxy](https://img.shields.io/badge/Proxy-Velocity-blue?style=for-the-badge)
![Java](https://img.shields.io/badge/Java-17+-orange?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Stable-brightgreen?style=for-the-badge)

> Vollständige Anleitung zur Konfiguration von **Velocity** auf einem VPS / Hosting für ein Minecraft-Servernetzwerk

---

## 📡 Architektur (so funktioniert es)

```
Spieler
  ↓
Velocity (Proxy)
  ↓
Lobby / BoxPvP / Survival (Backends)
```

Velocity fungiert als Vermittler, d. h. der Spieler verbindet sich nur mit ihm, und er kümmert sich um den Rest.

---

## 📌 Was ist Velocity?

**Velocity** ist ein moderner Proxy, der vom Paper-Team entwickelt wurde.

In der Praxis:

* Der Spieler verbindet sich mit dem Proxy
* Der Proxy authentifiziert den Spieler
* Der Proxy leitet ihn an den entsprechenden Server weiter

→ Das Backend (Paper/Spigot) sollte niemals öffentlich zugänglich sein.

---

## 💡 Warum lohnt es sich, Velocity zu nutzen?

- ✔️ deutlich bessere Leistung als BungeeCord  
- ✔️ moderne Sicherheitsmaßnahmen (Forwarding + Secret)  
- ✔️ Backends sind getrennt (höhere Sicherheit)  
- ✔️ unterstützt neue Minecraft-Versionen  
- ✔️ aktiv weiterentwickeltes Projekt  

Kurz gesagt: **der Standard in aktuellen Minecraft-Netzwerken**

---

## Anforderungen

* Java 17+
* VPS oder Hosting (z. B. Pterodactyl)
* Velocity `.jar`
* Backends (Paper / Purpur / Spigot)

---

## Installation (VPS / Hosting)

### Dateistruktur

```
/server/
  ├── velocity/   ← Proxy
  ├── lobby/      ← Backend
  ├── boxpvp/     ← Backend
```

### Hosting (z. B. Pterodactyl)

* Velocity = separater Server
* Jedes Backend = separate Instanz

**Ports:**

* Proxy → `25565` (öffentlich)
* Backend → `25566+` (privat)

⚠️ Backends dürfen NICHT öffentlich zugänglich sein

### VPS (Linux)

Proxy starten:

```bash
screen -S velocity
java -Xms1G -Xmx1G -jar velocity.jar
```

---

## ⚙️ Proxy-Konfiguration (Velocity)

Datei:

```
velocity.toml
```

### Weiterleitung (AM WICHTIGSTEN)

```toml
player-info-forwarding-mode = „modern“
```

### Was bewirkt das eigentlich?

Velocity übermittelt dem Backend die Daten des Spielers:

* UUID
* IP
* Skin

Aber nicht „auf das Wort“ – sondern:

* signiert (MAC)
* durch ein Secret gesichert

→ Dadurch kann sich niemand als der Spieler ausgeben

### Secret (Sicherheit)

```toml
forwarding-secret-file = „forwarding.secret“
```

- → Die Datei wird automatisch generiert
- → MUSS im Backend identisch sein

### Backends hinzufügen

```toml
[servers]
lobby = „IP:25566“
boxpvp = „IP:25567“
```

→ Wenn du einen VPS hast → kannst du `127.0.0.1` verwenden
→ Wenn du Hosting nutzt → verwende die IP aus dem Control Panel

### Startserver

```toml
try = [
  „lobby“
]
```

---

## ⚡ So funktioniert die Verbindung (Schritt für Schritt)

Das ist wichtig, da die meisten Probleme auf Missverständnisse zurückzuführen sind.

### 1. Der Spieler verbindet sich mit dem Proxy

* gibt die Server-IP ein
* gelangt zu Velocity
* Velocity führt die Authentifizierung durch

### 2. Erstellen der Spielerdaten

Velocity bereitet vor:

* UUID
* IP
* Profil (Skin)

Die Daten sind signiert und gesichert.

### 3. Verbindung zum Backend

Velocity verbindet sich mit dem Backend und sendet die Daten.

Backend:

* überprüft `secret`
* wenn OK → lässt den Spieler herein
* wenn NEIN → `not forwarded`

### 4. Weiterleitung

Der Spieler gelangt zu:

* einem Server mit `try` (z. B. Lobby)

## ⚙️ Backend-Konfiguration (Paper)

### Neue Versionen (paper-global.yml)

```yaml
proxies:
  velocity:
    enabled: true
    online-mode: true
    secret: „TU_SECRET“
```

### Ältere Versionen (paper.yml)

```yaml
settings:
  velocity-support:
    enabled: true
    online-mode: true
    secret: „TU_SECRET“
```

### server.properties

```properties
online-mode=false
```

→ Der Proxy übernimmt die Authentifizierung, d. h. das Backend darf dies nicht tun

### Zusätzlich

```yaml
# spigot.yml
settings:
  bungeecord: false
```

---

## 🔒 Sicherheitsmaßnahmen (MUST HAVE)

Wenn du dies nicht tust, wird jemand den Proxy umgehen.

- ✔️ Firewall (Sperrung der Backends)
- ✔️ Backend nur für den Proxy zugänglich
- ✔️ Keine öffentliche IP-Adresse der Backends

---

## Starten

Die Reihenfolge ist wichtig:

1. Backends (Paper)
2. Velocity
3. Du loggst dich über den Proxy ein

---

## Testen, ob es funktioniert

- ✔️ Du kannst dich über den Proxy einloggen
- ✔️ Du gelangst in die Lobby
- ✔️ `/server boxpvp` funktioniert
- ✔️ Du kannst dich NICHT direkt auf dem Backend anmelden

→ Wenn alles funktioniert → hast du alles richtig konfiguriert

---

## ❌ Häufigste Fehler

### Player info forwarding failed

- → falscher Secret


### Disconnected: not forwarded

- → Backend hat kein Velocity in der Konfiguration


### Backend läuft ohne Proxy

- → keine Firewall


### Keine Verbindung zum Backend

- → falsche IP / Port

---

## 🧰 Schnelle Checkliste (Debug)

* [ ] Secret ist identisch
* [ ] Backend hat `online-mode=false`
* [ ] Velocity hat `modern`
* [ ] Backend läuft
* [ ] Port ist korrekt
* [ ] Firewall blockiert den Proxy nicht
* [ ] Backend ist nicht öffentlich

---

## 📎 Weitere Informationen

* Velocity ersetzt das Backend NICHT – es ist nur ein Proxy
* Jedes Backend ist ein separater Minecraft-Server
* Die Kommunikation zwischen den Servern erfordert Plugins (z. B. Messaging / Redis)

---

## Autor

**noisy144 / mikusiek144**

* GitHub: https://github.com/mikusiek144
* Discord: noisy144

## Support

Wenn dir dieser Leitfaden geholfen hat:

*  Gib dem Beitrag eine Bewertung
*  Teile ihn
*  Hast du ein Problem? Schreib uns auf Discord

---
