# velocity-setup POLSKI

![Minecraft](https://img.shields.io/badge/Minecraft-Network-green?style=for-the-badge)
![Proxy](https://img.shields.io/badge/Proxy-Velocity-blue?style=for-the-badge)
![Java](https://img.shields.io/badge/Java-17+-orange?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Stable-brightgreen?style=for-the-badge)

> Kompletny poradnik konfiguracji **Velocity** na VPS lub hostingu dla sieci serwerów Minecraft

---

## 📡 Architektura (jak to działa)

```id="m4k9pz"
Gracz
  ↓
Velocity (proxy)
  ↓
Lobby / BoxPvP / Survival (backendy)
```

Velocity działa jako pośrednik — gracz łączy się tylko z nim, a on zajmuje się resztą.

---

## 📌 Czym jest Velocity?

**Velocity** to nowoczesne proxy stworzone przez zespół Paper.

W praktyce:

* gracz łączy się z proxy
* proxy autoryzuje gracza
* proxy przekierowuje go na odpowiedni serwer

→ Backend (Paper/Spigot) nigdy nie powinien być dostępny publicznie

---

## 💡 Dlaczego warto używać Velocity?

* ✔️ dużo lepsza wydajność niż BungeeCord
* ✔️ nowoczesne zabezpieczenia (forwarding + secret)
* ✔️ oddzielone backendy (większe bezpieczeństwo)
* ✔️ wsparcie dla nowych wersji Minecraft
* ✔️ aktywnie rozwijany projekt

Krótko: **standard w nowoczesnych sieciach Minecraft**

---

## Wymagania

* Java 17+
* VPS lub hosting (np. Pterodactyl)
* Velocity `.jar`
* Backendy (Paper / Purpur / Spigot)

---

## Instalacja (VPS / hosting)

### Struktura plików

```id="q7n2vd"
/server/
  ├── velocity/   ← proxy
  ├── lobby/      ← backend
  ├── boxpvp/     ← backend
```

### Hosting (np. Pterodactyl)

* Velocity = osobny serwer
* każdy backend = osobna instancja

**Porty:**

* Proxy → `25565` (publiczny)
* Backend → `25566+` (prywatne)

⚠️ Backendów NIE należy wystawiać publicznie

### VPS (Linux)

Uruchomienie proxy:

```bash id="c1z8rt"
screen -S velocity
java -Xms1G -Xmx1G -jar velocity.jar
```

---

## ⚙️ Konfiguracja proxy (Velocity)

Plik:

```id="x6k3pw"
velocity.toml
```

### Forwarding (NAJWAŻNIEJSZE)

```toml id="d9v5hs"
player-info-forwarding-mode = "modern"
```

### Co to właściwie robi?

Velocity przekazuje backendowi dane gracza:

* UUID
* adres IP
* skin

Ale nie „na słowo”, tylko:

* podpisane (MAC)
* zabezpieczone przy użyciu secret

→ Dzięki temu nikt nie może podszyć się pod gracza

### Secret (zabezpieczenie)

```toml id="p2w7jn"
forwarding-secret-file = "forwarding.secret"
```

* → plik generuje się automatycznie
* → MUSI być identyczny w backendzie

### Dodanie backendów

```toml id="t5r1bx"
[servers]
lobby = "IP:25566"
boxpvp = "IP:25567"
```

→ Na VPS możesz użyć `127.0.0.1`
→ Na hostingu użyj IP z panelu

### Serwer startowy

```toml id="k8m4yu"
try = [
  "lobby"
]
```

---

## ⚡ Jak działa połączenie (krok po kroku)

To ważne, bo większość problemów wynika z nieporozumień.

### 1. Gracz łączy się z proxy

* wpisuje IP serwera
* trafia do Velocity
* Velocity przeprowadza autoryzację

### 2. Tworzenie danych gracza

Velocity przygotowuje:

* UUID
* adres IP
* profil (skin)

Dane są podpisane i zabezpieczone

### 3. Połączenie z backendem

Velocity łączy się z backendem i przesyła dane

Backend:

* sprawdza `secret`
* jeśli poprawny → wpuszcza gracza
* jeśli nie → `not forwarded`

### 4. Przekierowanie

Gracz zostaje przekierowany na:

* serwer z listy `try` (np. lobby)

---

## ⚙️ Konfiguracja backendu (Paper)

### Nowe wersje (paper-global.yml)

```yaml id="v3n9dq"
proxies:
  velocity:
    enabled: true
    online-mode: true
    secret: "TU_SECRET"
```

### Starsze wersje (paper.yml)

```yaml id="r1k6js"
settings:
  velocity-support:
    enabled: true
    online-mode: true
    secret: "TU_SECRET"
```

### server.properties

```properties id="b7c2xm"
online-mode=false
```

→ Autoryzację przejmuje proxy, więc backend nie może jej wykonywać

### Dodatkowo

```yaml id="y4p8lw"
# spigot.yml
settings:
  bungeecord: false
```

---

## 🔒 Zabezpieczenia (MUST HAVE)

Jeśli tego nie ustawisz, proxy może zostać ominięte.

* ✔️ firewall (blokada backendów)
* ✔️ backend dostępny tylko przez proxy
* ✔️ brak publicznego dostępu do backendów

---

## Uruchamianie

Kolejność ma znaczenie:

1. Backendy (Paper)
2. Velocity
3. Łączysz się przez proxy

---

## Test działania

* ✔️ możesz wejść przez proxy
* ✔️ trafiasz na lobby
* ✔️ `/server boxpvp` działa
* ✔️ NIE możesz wejść bezpośrednio na backend

→ Jeśli wszystko działa → konfiguracja jest poprawna

---

## ❌ Najczęstsze błędy

### Player info forwarding failed

* → niepoprawny secret

### Disconnected: not forwarded

* → Velocity nie jest skonfigurowane w backendzie

### Backend działa bez proxy

* → brak firewalla

### Brak połączenia z backendem

* → niepoprawny IP lub port

---

## 🧰 Szybka checklista (debug)

* [ ] secret jest identyczny
* [ ] backend ma `online-mode=false`
* [ ] Velocity używa `modern`
* [ ] backend działa
* [ ] port jest poprawny
* [ ] firewall nie blokuje proxy
* [ ] backend nie jest publiczny

---

## 📎 Dodatkowe informacje

* Velocity NIE zastępuje backendu - to tylko proxy
* każdy backend to osobny serwer Minecraft
* komunikacja między serwerami wymaga pluginów (np. messaging / Redis)

---

## Autor

**noisy144 / mikusiek144**

* GitHub: https://github.com/mikusiek144
* Discord: noisy144

## Wsparcie

Jeśli poradnik Ci pomógł:

* zostaw gwiazdkę ⭐
* udostępnij go
* masz problem? napisz na Discord
---
