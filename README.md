# Tagebuch

Eine eigenständige, selbst-gehostete **Progressive Web App (PWA)** als persönliches Tagebuch.
Daten bleiben **lokal auf dem Gerät** (IndexedDB) – kein Backend, keine Accounts, kein Tracking.

Portiert aus dem ursprünglichen Claude-Artifact (`tagebuch.jsx`), aber komplett unabhängig:
`window.storage` → **IndexedDB**, plus Manifest, Service-Worker und Icons für die Installation.

## Funktionen

- Monatskalender mit Kategorie-Punkten pro Tag
- Tag-Detail: Einträge anlegen, bearbeiten, löschen
- Frei konfigurierbare Kategorien (Name + Farbe)
- Volltext-Suche über alle Einträge
- **JSON-Export/-Import** – Format identisch zum Claude-Artifact (`{ categories, entries }`),
  d. h. alte Exporte lassen sich hier importieren
- Spracheingabe (sofern vom Browser unterstützt)
- Offline-fähig, installierbar („Zum Startbildschirm hinzufügen")
- Barrierefrei: sichtbarer Fokus, Tap-Ziele ≥ 44 px, `prefers-reduced-motion` respektiert

## Kein Build nötig

Das ist bewusst eine **einzelne `index.html`** mit Vanilla-JS – kein npm, kein Bundler,
keine Laufzeit-Abhängigkeit. Einfach die Dateien statisch ausliefern.

```
index.html              # die komplette App
manifest.webmanifest    # PWA-Manifest
service-worker.js       # Offline-Cache (App-Shell)
icons/                  # 192/512 px, normal + maskable
```

## Lokal testen

Ein Service-Worker braucht `http(s)://` (nicht `file://`). Beliebiger statischer Server:

```bash
python3 -m http.server 8000
# dann http://localhost:8000 öffnen
```

## Deployment

### GitHub Pages (dieses Repo)
Repo-Einstellungen → **Pages** → Branch `main`, Ordner `/ (root)`.
Die App liegt dann unter `https://<user>.github.io/stiggycal.github.io/`.
Alle Pfade sind **relativ**, funktioniert also auch im Unterordner.

### Netlify Drop (Alternative, ~1 Min.)
Den Projektordner auf <https://app.netlify.com/drop> ziehen. Fertig – inkl. HTTPS.

> HTTPS ist Pflicht für PWA/Service-Worker. GitHub Pages und Netlify liefern das automatisch.

## Datensicherheit

Der Gerätespeicher kann vom Browser geleert werden (Speicherdruck, „Website-Daten löschen").
Die App fragt daher `navigator.storage.persist()` an – **sichere deine Einträge trotzdem
regelmäßig über Menü → „Als Datei sichern (JSON)"**. Das ist zugleich dein Umzugsweg.

Lokal heißt: **pro Gerät.** Kein automatischer Sync zwischen Handy und Desktop – der Weg
dorthin führt über Export/Import.
