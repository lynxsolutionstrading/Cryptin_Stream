# Cryptin Stream

Statischer WebSocket-Viewer für den Cryptin Game-Streamer. Wird über GitHub Pages bereitgestellt; verbindet sich zu einem auf deinem PC laufenden Python-Streamer (siehe `Cryptin Dev/streamer/`) über einen Cloudflare Tunnel.

**Live:** https://lynxsolutionstrading.github.io/Cryptin_Stream/

## Architektur

```
[Game Client] -> [streamer (Python, lokal)] -> [Cloudflare Tunnel] -> [GitHub Pages Viewer (this repo)]
                                                                            ^
                                                                            |
                                                                       Browser
```

Der Viewer ist 100% statisch. GitHub Pages serviert nur HTML/CSS/JS, keinen Server. Frames und Input fliessen direkt zwischen Browser und deinem Tunnel-Endpoint, nicht durch GitHub.

## Setup (einmalig)

### 1. Cloudflared installieren

```powershell
winget install --id Cloudflare.cloudflared
```

Alternative: [Direktdownload](https://github.com/cloudflare/cloudflared/releases) und in `PATH` legen.

### 2. Auth-Token setzen

Auf deinem PC, vor Streamer-Start:

```powershell
$env:CRYPTIN_STREAM_TOKEN = "ein-langer-zufaelliger-string"
```

Der Token muss zwischen Streamer (Server) und Viewer (Browser) übereinstimmen. Ohne Token akzeptiert der Server nur Verbindungen von `127.0.0.1`.

## Starten (jedes Mal)

### 1. Streamer starten

```powershell
$env:CRYPTIN_STREAM_TOKEN = "ein-langer-zufaelliger-string"
cd "C:\Users\Computer\Desktop\Cryptin Dev\streamer"
py launcher.py
```

### 2. Tunnel öffnen

In einem zweiten Fenster:

```powershell
cloudflared tunnel --url http://localhost:8083
```

Cloudflared gibt eine zufällige URL aus, z.B.:
```
Your quick Tunnel has been created! Visit it at: https://abc-def-ghi.trycloudflare.com
```

### 3. Im Browser öffnen

```
https://lynxsolutionstrading.github.io/Cryptin_Stream/?ws=wss://abc-def-ghi.trycloudflare.com/ws&token=ein-langer-zufaelliger-string
```

Oder die Settings-Maske der Seite nutzen — der Browser merkt sich Werte in `localStorage`.

## Sicherheit

Der Streamer leitet Maus + Tastatur an dein Spiel weiter. **Wer die Tunnel-URL und den Token hat, kann deinen PC fernsteuern.**

- Token muss lang und zufällig sein (nicht `change-me-please`).
- Tunnel-URLs sind quasi-geheim, aber sollten nicht öffentlich geteilt werden.
- Im Viewer kannst du jederzeit auf **View-only: ON** klicken, dann werden lokale Mausklicks/Tasten nicht mehr ans Spiel geschickt.
- Server-seitig komplett deaktivieren via:
  ```powershell
  $env:CRYPTIN_STREAM_ALLOW_INPUT = "0"
  ```
  Dann ist der Stream pure Anzeige; lokale Verbindungen auf `127.0.0.1` bleiben unbeschränkt.

## Deployment auf GitHub Pages

1. Repo `Cryptin_Stream` auf GitHub erstellen (public).
2. Diesen Ordner pushen:
   ```powershell
   cd "C:\Users\Computer\Desktop\Cryptin_Stream"
   git init
   git add .
   git commit -m "initial: static stream viewer"
   git branch -M main
   git remote add origin https://github.com/lynxsolutionstrading/Cryptin_Stream.git
   git push -u origin main
   ```
3. GitHub → Repo → **Settings → Pages**:
   - Source: **Deploy from a branch**
   - Branch: **main** / **/ (root)**
   - Save.
4. Nach ca. 1-2 Min läuft die Seite unter `https://lynxsolutionstrading.github.io/Cryptin_Stream/`.

## Troubleshooting

| Symptom | Ursache |
|---|---|
| `disconnected (1006)` direkt nach Connect | Token falsch ODER Tunnel zeigt nicht auf den richtigen Port |
| Schwarzer Canvas, "connected" aber keine Frames | Game läuft nicht / Streamer findet `metin2client.exe` nicht |
| Schlechte Latenz | Cloudflare Tunnel + JPEG sind ~150-300 ms. Für lokal (gleiches WLAN) lieber direkt `http://<lan-ip>:8083` |
| GitHub Pages zeigt 404 | Pages noch nicht deployed; Settings → Pages prüfen |
