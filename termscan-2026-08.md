# TermScan — Live Barcode Scanner (Browser)

## Ziel
Android-Kamera als Barcode-/QR-Scanner in einer browserbasierten App nutzen — kein natives App-Wrapping, nur Chrome/Edge (kein Fallback für andere Browser nötig).

## Ergebnis
Funktionierender Prototyp: `termscan-live.html`
- Kamerazugriff via `getUserMedia` (Rückkamera, `facingMode: environment`)
- Erkennung via native **`BarcodeDetector`-API** (Shape Detection API) — unterstützt QR, EAN-13/8, Code128/39/93, PDF417, Aztec, Data Matrix, UPC, Codabar, ITF
- Kein Framework, kein Fallback (ZXing o. ä. bewusst weggelassen — Zielgeräte sind Chrome/Edge)
- UI im TERMSCAN-Look (Design-Canvas-Mockup `TermScan-Scanner-standalone.html` war die Vorlage): dunkles Theme, orangener Akzent, Sucherrahmen mit Eckmarkierungen, Scan-Laser-Animation
- Features: Treffer-Flash, Vibration bei Hit, Dedupe (gleicher Code wird 2,5 s lang nicht erneut gemeldet), Ergebnisliste mit Format/Zeitstempel + Copy-Button

**Status: getestet und funktionsfähig** — ein QR-Code wurde erfolgreich live auf dem Handy gescannt.

## Testaufbau (WSL2 + Android-Handy, kabellos)
WSL2 hat kein natives USB-Passthrough, deshalb über **Wireless Debugging** (Android 11+) statt USB-Kabel:

1. `adb` installieren (braucht `sudo`, ging in dieser Umgebung nur über ein externes Terminal, nicht über den Claude-Code-Bash-Zugriff — kein TTY für Passwort):
   ```
   sudo apt-get install -y android-tools-adb
   ```
2. Auf dem Handy: Entwickleroptionen → Wireless-Debugging aktivieren → "Gerät mit Pairing-Code koppeln" → IP:Port + 6-stelliger Code
3. Pairing:
   ```
   adb pair <IP>:<PAIRING_PORT>
   ```
4. Verbinden (zweite IP:Port-Zeile vom Wireless-Debugging-Hauptbildschirm):
   ```
   adb connect <IP>:<CONNECT_PORT>
   ```
5. Port-Reverse einrichten (Handy-`localhost:8000` → WSL2-`localhost:8000`):
   ```
   adb reverse tcp:8000 tcp:8000
   ```
6. Lokalen Server im Projektordner starten:
   ```
   cd ~/projects/claude1 && python3 -m http.server 8000
   ```
7. Auf dem Handy in Chrome öffnen:
   ```
   http://localhost:8000/termscan-live.html
   ```

Getestetes Gerät: Samsung Galaxy Xcover 4s (SM-G398FN).

⚠️ Pairing/Connect/Reverse sind nicht persistent — nach WSL2-Neustart oder wenn das Handy die Wireless-Debugging-Verbindung trennt, müssen Schritt 3–6 wiederholt werden (Pairing-Code ändert sich jedes Mal, `adb connect` + `adb reverse` reichen aber oft ohne erneutes Pairing, falls die Verbindung nur kurz unterbrochen war).

## Offene Ideen (nicht umgesetzt)
- Export der Ergebnisliste als CSV/JSON
- Ton-Feedback bei Treffer (zusätzlich zur Vibration)
- Rückintegration ins Design-Canvas-Mockup (`TermScan-Scanner-standalone.html`), falls das Optik-Vorbild als tatsächliche App weitergebaut werden soll

## Dateien
- `TermScan-Scanner-standalone.html` — ursprüngliches Design-Canvas-Mockup (nur Optik, keine Funktion)
- `termscan-live.html` — funktionierender Live-Scanner (dieser Prototyp)
