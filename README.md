# claude1

> Agenten-Repo Claude Code

## TermScan — Live Barcode-/QR-Scanner (PWA)

Browserbasierter Barcode-/QR-Scanner für Android, kein natives App-Wrapping. Nutzt die
Kamera direkt im Browser und schreibt jeden Treffer in eine Supabase-Tabelle.

**Live:** https://anchje.github.io/claude1/termscan-live.html

### Funktionsweise
- Kamerazugriff via `getUserMedia` (Rückkamera, `facingMode: environment`)
- Erkennung über die native **`BarcodeDetector`-API** (Shape Detection API) —
  unterstützt QR, EAN-13/8, Code128/39/93, PDF417, Aztec, Data Matrix, UPC, Codabar, ITF
- Kein Framework, kein Fallback (ZXing o. ä. bewusst weggelassen — Zielgeräte sind
  Chrome/Edge auf Android)
- UI im TERMSCAN-Look: dunkles Theme, orangener Akzent, Sucherrahmen mit
  Eckmarkierungen, Scan-Laser-Animation
- Treffer-Flash, Vibration bei Hit, Dedupe (gleicher Code wird 2,5 s lang nicht erneut
  gemeldet), Ergebnisliste mit Format/Zeitstempel + Copy-Button

**Status: produktiv im Einsatz** — gehostet auf GitHub Pages, als PWA installierbar,
Ergebnisse werden live nach Supabase gesynct. Getestet auf einem Samsung Galaxy
Xcover 4s (SM-G398FN) in Chrome.

### Dateien
| Datei | Zweck |
|---|---|
| `termscan-live.html` | Der eigentliche Scanner (Kamera, Erkennung, UI, Supabase-Sync) |
| `index.html` | Redirect auf `termscan-live.html`, damit die Pages-Root-URL funktioniert |
| `manifest.json` | PWA-Manifest (Name, Icons, `start_url`, Theme-Farben) |
| `sw.js` | Service Worker — cached App-Shell für Offline-Start |
| `icon.svg` | App-Icon (TERMSCAN-Look, dunkler Hintergrund + orangener Rahmen) |
| `TermScan-Scanner-standalone.html` | Ursprüngliches Design-Canvas-Mockup (nur Optik, keine Funktion — war Vorlage für `termscan-live.html`) |
| `termscan-2026-08.md` | Notizen aus der lokalen Testphase (WSL2 + adb, vor dem Umzug auf GitHub Pages) |

### Hosting: GitHub Pages
- Repo ist **public** (Voraussetzung für kostenlose GitHub Pages)
- Settings → Pages → Source: **Deploy from a branch** → `main` / `/ (root)`
- HTTPS von GitHub Pages ist Voraussetzung für Kamerazugriff (`getUserMedia`
  funktioniert nur in einem "secure context" — HTTPS oder `localhost`)
- Deploy passiert automatisch bei jedem Push auf `main`, kein Build-Schritt nötig
  (reines statisches HTML/JS)

### Als App installieren (PWA)
Auf dem Handy in Chrome den Live-Link öffnen → Menü → **"Zum Startbildschirm
hinzufügen"**. Dank `manifest.json` + `sw.js` startet die App danach wie eine
native App (eigenes Icon, kein Browser-Chrome, App-Shell wird offline gecacht).

### Supabase-Backend
Jeder gescannte Code wird per `supabase-js` (CDN) in die Tabelle `public.scans`
geschrieben. In der UI zeigt jede Ergebniszeile den Sync-Status ("speichert…" →
"gespeichert ✓" / "nicht gespeichert").

- Projekt-URL: `https://qiqjywcyqcrccgbocbdq.supabase.co`
- Im Frontend verwendeter Key: der öffentliche **publishable key** (`sb_publishable_...`)
  — das ist kein Secret und darf im Client-Code stehen

**Schema:**
```sql
create table public.scans (
  id bigint generated always as identity primary key,
  value text not null,
  format text,
  created_at timestamptz not null default now()
);

alter table public.scans enable row level security;

create policy "Allow public insert" on public.scans
  for insert
  to anon
  with check (true);

create policy "Allow public select" on public.scans
  for select
  to anon
  using (true);
```

RLS ist aktiv, `anon` darf nur **Insert + Select** — kein Update/Delete. Da die
Tabelle öffentlich lesbar ist, ist das für ein internes/kleines Tool ok; bei
breiterer Nutzung sollte hier noch ein Zugriffsschutz (z.B. Supabase Auth oder
Shared Secret) ergänzt werden.

⚠️ Free-Tier-Projekte pausieren nach Inaktivität automatisch. Nach dem
Reaktivieren im Dashboard ("Restore project") kann die Datenbank noch ein paar
Minuten im Read-only-Modus hängen, bevor Schema-Änderungen (DDL) wieder
möglich sind.

### Lokales Testen (alte Methode, vor GitHub Pages)
Für die frühe Testphase (bevor HTTPS-Hosting eingerichtet war) wurde lokal über
WSL2 + Android Wireless Debugging getestet — Details dazu in
`termscan-2026-08.md`. Seit dem Umzug auf GitHub Pages ist das nicht mehr nötig,
der Live-Link funktioniert direkt auf jedem Gerät mit Chrome/Edge.

### Offene Ideen (nicht umgesetzt)
- Export der Ergebnisliste als CSV/JSON
- Ton-Feedback bei Treffer (zusätzlich zur Vibration)
- Zugriffsschutz für die Supabase-Tabelle (Auth oder Shared Secret), falls das
  Tool über den internen Kreis hinaus genutzt wird
