# Tagebuch

Eine eigenständige, selbst-gehostete **Progressive Web App (PWA)** als persönliches Tagebuch.
Daten bleiben **lokal auf dem Gerät** (IndexedDB) – kein Backend, keine Accounts, kein Tracking.

Portiert aus dem ursprünglichen Claude-Artifact (`tagebuch.jsx`), aber komplett unabhängig:
`window.storage` → **IndexedDB**, plus Manifest, Service-Worker und Icons für die Installation.

## Funktionen

- **Passwortschutz mit Ende-zu-Ende-Verschlüsselung** (siehe unten)
- Monatskalender mit Kategorie-Punkten pro Tag
- **Wochenansicht** (Agenda) – umschaltbar Monat/Woche
- **Monats-Statistik** – Kacheln, Balken pro Tag, Verteilung nach Kategorie
- Tag-Detail: Einträge anlegen, bearbeiten, löschen
- **Wischen** (Swipe) für vorigen/nächsten Tag, Monat bzw. Woche
- Frei konfigurierbare Kategorien (Name + Farbe), **per Drag sortierbar**;
  Kategorie-Chips füllen die Bildschirmbreite (mehrere Reihen)
- **Tages-Markierungen W · H · S · B** – feste Buchstaben ohne Text-Eintrag,
  pro Tag in der Tagesansicht umschaltbar, klein unten im Kalenderfeld angezeigt
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

## Sicherheit / Login

Beim ersten Start legst du ein **Passwort** fest. Daraus wird per **PBKDF2-SHA-256**
(210 000 Iterationen) ein Schlüssel abgeleitet, mit dem alle Einträge und Kategorien
**AES-GCM-256-verschlüsselt** in der lokalen Datenbank liegen.

Das bedeutet konkret:

- **Das Passwort wird nie gespeichert** – weder im Code noch in der Datenbank. Abgelegt
  werden nur ein zufälliger Salt und ein Prüf-Token (verschlüsselt).
- **Die Daten liegen nur verschlüsselt vor.** Wer die Datenbank oder das Gerät ausliest,
  sieht ohne Passwort nur Zufallsbytes.
- **Kein Server, kein Endpunkt „von außen".** Alles bleibt auf dem Gerät.
- **Jeder Neustart/Reload verlangt das Passwort** (der Schlüssel lebt nur im Speicher).
  Zusätzlich: manuelles „Sperren" im Menü und **Auto-Sperre nach 5 Minuten** im Hintergrund.
- **Passwort ändern** verschlüsselt alle Einträge mit dem neuen Schlüssel neu.

> **Kein Passwort-Reset.** Ist das Passwort weg, sind die Einträge unwiederbringlich.
> Der (unverschlüsselte) JSON-Export ist dein Backup und Rettungsanker – mach ihn regelmäßig.

Warum ist der öffentliche Quelltext kein Problem? Die Sicherheit steckt **nicht** im
Geheimhalten des Codes, sondern im passwortabgeleiteten Schlüssel (Prinzip von Kerckhoffs).
Voraussetzung ist eine sichere Herkunft der App – deshalb **HTTPS** (GitHub Pages liefert das).

## Datensicherheit / Backup

Der Gerätespeicher kann vom Browser geleert werden (Speicherdruck, „Website-Daten löschen").
Die App fragt daher `navigator.storage.persist()` an – **sichere deine Einträge trotzdem
regelmäßig über Menü → „Als Datei sichern (JSON)"**. Der Export ist unverschlüsselt (damit
importierbar/portabel) – behandle die Datei entsprechend vertraulich.

Lokal heißt: **pro Gerät.** Kein automatischer Sync zwischen Handy und Desktop – der Weg
dorthin führt über Export/Import.
