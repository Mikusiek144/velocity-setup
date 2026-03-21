# velocity-setup POLSKI
> Kompletny poradnik konfiguracji **Velocity** na VPS / hostingu pod sieć serwerów Minecraft

---

## 📡 Architektura (jak to działa)

```
Gracz
  ↓
Velocity (proxy)
  ↓
Lobby / BoxPvP / Survival (backendy)
```

Velocity działa jako pośrednik czyli gracz łączy się tylko z nim, a on ogarnia resztę.

---

## 📌 Czym jest Velocity?

**Velocity** to nowoczesne proxy stworzone przez team Paper.

W praktyce:

* gracz łączy się do proxy
* proxy autoryzuje gracza
* proxy przekierowuje go na odpowiedni serwer

👉 Backend (Paper/Spigot) nigdy nie powinien być dostępny publicznie.

---

## 💡 Dlaczego warto używać Velocity?

✔️ dużo lepsza wydajność niż BungeeCord
✔️ nowoczesne zabezpieczenia (forwarding + secret)
✔️ backendy są odseparowane (większe bezpieczeństwo)
✔️ wspiera nowe wersje Minecraft
✔️ aktywnie rozwijany projekt

Krótko: **standard w aktualnych sieciach Minecraft**

---

## ⚙️ Wymagania

* Java 17+
* VPS lub hosting (np. Pterodactyl)
* Velocity `.jar`
* Backendy (Paper / Purpur / Spigot)

---

## 📥 Instalacja (VPS / hosting)

### 📁 Struktura plików

```
/server/
  ├── velocity/   ← proxy
  ├── lobby/      ← backend
  ├── boxpvp/     ← backend
```

---

### 🌐 Hosting (np. Pterodactyl)

* Velocity = osobny serwer
* Każdy backend = osobna instancja

**Porty:**

* Proxy → `25565` (publiczny)
* Backend → `25566+` (prywatne)

⚠️ Backendów NIE wystawiasz publicznie

---

### 🖥️ VPS (Linux)

Uruchomienie proxy:

```bash
screen -S velocity
java -Xms1G -Xmx1G -jar velocity.jar
```

---

## ⚙️ Konfiguracja proxy (Velocity)

Plik:

```
velocity.toml
```

---

### 🔑 Forwarding (NAJWAŻNIEJSZE)

```toml
player-info-forwarding-mode = "modern"
```

### 🧠 Co to właściwie robi?

Velocity przekazuje backendowi dane gracza:

* UUID
* IP
* skin

Ale nie „na słowo” — tylko:

* podpisane (MAC)
* zabezpieczone przez secret

→ Dzięki temu nikt nie może się podszyć pod gracza

---

### 🔐 Secret (zabezpieczenie)

```toml
forwarding-secret-file = "forwarding.secret"
```

➡️ Plik wygeneruje się automatycznie
➡️ MUSI być identyczny w backendzie

---

### 🌐 Dodanie backendów

```toml
[servers]
lobby = "IP:25566"
boxpvp = "IP:25567"
```

📌 Jeśli masz VPS → możesz użyć `127.0.0.1`
📌 Jeśli hosting → użyj IP z panelu

---

### 🚪 Serwer startowy

```toml
try = [
  "lobby"
]
```

---

## 🔗 Jak działa podłączenie (krok po kroku)

To jest ważne, bo większość problemów wynika z niezrozumienia.

---

### 1️⃣ Gracz łączy się z proxy

* wpisuje IP serwera
* trafia do Velocity
* Velocity robi autoryzację

---

### 2️⃣ Tworzenie danych gracza

Velocity przygotowuje:

* UUID
* IP
* profil (skin)

Dane są podpisane i zabezpieczone.

---

### 3️⃣ Połączenie z backendem

Velocity łączy się z backendem i wysyła dane.

Backend:

* sprawdza `secret`
* jeśli OK → wpuszcza gracza
* jeśli NIE → `not forwarded`

---

### 4️⃣ Przekierowanie

Gracz trafia na:

* serwer z `try` (np. lobby)

---

## ⚙️ Konfiguracja backend (Paper)

---

### 🆕 Nowe wersje (paper-global.yml)

```yaml
proxies:
  velocity:
    enabled: true
    online-mode: true
    secret: "TU_SECRET"
```

---

### 🧓 Starsze wersje (paper.yml)

```yaml
settings:
  velocity-support:
    enabled: true
    online-mode: true
    secret: "TU_SECRET"
```

---

### 📄 server.properties

```properties
online-mode=false
```

👉 Proxy przejmuje autoryzację czyli backend nie może tego robić

---

### ⚠️ Dodatkowo

```yaml
# spigot.yml
settings:
  bungeecord: false
```

---

## 🔒 Zabezpieczenia (MUST HAVE)

Jeśli tego nie zrobisz to ktoś ominie proxy.

✔️ firewall (blokada backendów)
✔️ backend dostępny tylko dla proxy
✔️ brak publicznego IP backendów

---

## ▶️ Uruchamianie

Kolejność ma znaczenie:

1. Backendy (Paper)
2. Velocity
3. Wchodzisz przez proxy

---

## 🧪 Test czy działa

✔️ możesz wejść przez proxy
✔️ trafiasz na lobby
✔️ `/server boxpvp` działa
✔️ NIE możesz wejść na backend bezpośrednio

→ Jeśli wszystko działa → masz dobrze skonfigurowane

---

## ❌ Najczęstsze błędy

### 🔴 Player info forwarding failed

➡️ zły secret

---

### 🔴 Disconnected: not forwarded

➡️ backend nie ma velocity w configu

---

### 🔴 Backend działa bez proxy

➡️ brak firewalla

---

### 🔴 Nie łączy z backendem

➡️ zły IP / port

---

## 🧰 Szybka checklista (debug)

* [ ] secret jest identyczny
* [ ] backend ma `online-mode=false`
* [ ] velocity ma `modern`
* [ ] backend działa
* [ ] port jest poprawny
* [ ] firewall nie blokuje proxy
* [ ] backend nie jest publiczny

---

## 📎 Dodatkowe informacje

* Velocity NIE zastępuje backendu — to tylko proxy
* Każdy backend to osobny serwer Minecraft
* Komunikacja między serwerami wymaga pluginów (np. messaging / Redis)

---

## 👤 Autor

**noisy144 / mikusiek144**

* GitHub: https://github.com/mikusiek144
* Discord: noisy144

---

## ⭐ Wsparcie

Jeśli poradnik pomógł:

* ⭐ zostaw gwiazdkę na repo
* 🔁 udostępnij dalej
* 💬 masz problem? napisz na Discord

---
