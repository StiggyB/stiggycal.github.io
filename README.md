# Tagebuch

Eine eigenständige, selbst-gehostete **Progressive Web App (PWA)** als persönliches Tagebuch.
Daten bleiben **lokal auf dem Gerät** (IndexedDB) – kein Backend, keine Accounts, kein Tracking.
Optional synchronisiert ein **verschlüsselter Online-Speicher** (privates GitHub-Repository)
mehrere Geräte – hochgeladen wird dabei ausschließlich Ciphertext (siehe unten).

Portiert aus dem ursprünglichen Claude-Artifact (`tagebuch.jsx`), aber komplett unabhängig:
`window.storage` → **IndexedDB**, plus Manifest, Service-Worker und Icons für die Installation.

## Funktionen

- **Passwortschutz mit Ende-zu-Ende-Verschlüsselung** (siehe unten)
- Monatskalender mit Kategorie-Punkten pro Tag
- **Wochenansicht** (Agenda) – umschaltbar Monat/Woche
- **Monats-Statistik** – Kacheln, Balken pro Tag, Stimmungsverlauf, Verteilung
  nach Kategorie sowie **Zähler pro Tages-Markierung** (wie oft kam welcher
  Buchstabe bzw. welches Symbol im Monat vor – auch in der Jahresansicht fürs Jahr)
- **Jahres-Heatmap** – Pixeljahr in der Statistik (umschaltbar Monat/Jahr),
  einfärbbar nach Einträgen oder Stimmung; Tippen auf einen Tag öffnet ihn
- Tag-Detail: Einträge anlegen, bearbeiten, löschen
- **„An diesem Tag"** – Rückblick in der Tagesansicht auf Einträge vom selben
  Datum vor 1/3/6 Monaten und in früheren Jahren
- **Stimmung pro Tag** (5 Stufen, Emoji) – in der Tagesansicht antippen, klein
  im Kalender/Wochenliste sichtbar, Verlauf in der Statistik
- **Schreibserie** – 🔥-Anzeige der aktuellen Serie (Kalender-Untertitel und Tagesansicht)
- **Wischen** (Swipe) für vorigen/nächsten Tag, Monat bzw. Woche
- Frei konfigurierbare Kategorien (Name + Farbe), **per Drag sortierbar**;
  Kategorie-Chips füllen die Bildschirmbreite (mehrere Reihen)
- **Tages-Markierungen W · H · S · B · 🍾 · 🍓** – feste Buchstaben und
  Emoji-Symbole ohne Text-Eintrag, pro Tag **mehrfach zählbar** (in der
  Tagesansicht +/−); im Kalenderfeld klein und entsprechend oft wiederholt
  angezeigt (z. B. „W W 🍾"). Weitere Symbole lassen sich in `index.html`
  in der Konstante `MARKERS` ergänzen (beliebige Emojis)
- Volltext-Suche über alle Einträge
- **Online-Speicher (optional)** – Ende-zu-Ende-verschlüsselte Synchronisation
  über ein privates GitHub-Repository; automatischer Abgleich zwischen Geräten
  inklusive Zusammenführung (Details unten)
- **JSON-Export/-Import** – Export enthält Kategorien, Markierungen, Stimmungen
  und Einträge; ältere Exporte (auch das ursprüngliche Claude-Artifact-Format
  `{ categories, entries }`) lassen sich weiterhin importieren
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
- **Kein eigener Server.** Alles bleibt auf dem Gerät – außer du richtest den optionalen
  Online-Speicher ein; auch dann verlässt nur Verschlüsseltes das Gerät.
- **Jeder Neustart/Reload verlangt das Passwort** (der Schlüssel lebt nur im Speicher).
  Zusätzlich: manuelles „Sperren" im Menü und **Auto-Sperre nach 5 Minuten** im Hintergrund.
- **Passwort ändern** verschlüsselt alle Einträge mit dem neuen Schlüssel neu.

> **Kein Passwort-Reset.** Ist das Passwort weg, sind die Einträge unwiederbringlich.
> Der (unverschlüsselte) JSON-Export ist dein Backup und Rettungsanker – mach ihn regelmäßig.

Warum ist der öffentliche Quelltext kein Problem? Die Sicherheit steckt **nicht** im
Geheimhalten des Codes, sondern im passwortabgeleiteten Schlüssel (Prinzip von Kerckhoffs).
Voraussetzung ist eine sichere Herkunft der App – deshalb **HTTPS** (GitHub Pages liefert das).

## Online-Speicher / Sync zwischen Geräten (optional)

Ohne Einrichtung bleibt alles wie gehabt rein lokal. Mit dem Online-Speicher hält die App
mehrere Geräte (z. B. Handy und Desktop) automatisch synchron – über ein **privates
GitHub-Repository**, das dir gehört. Es braucht keinen weiteren Dienst und kein Backend.

### Einrichtung (einmalig, ~2 Minuten)

1. Auf github.com ein **neues privates Repository** anlegen (z. B. `tagebuch-daten`).
2. Einen **Fine-grained Personal Access Token** erstellen – am schnellsten direkt über
   <https://github.com/settings/personal-access-tokens/new>. (Zu Fuß: **Avatar oben
   rechts → Settings** – die *Profil*-Einstellungen, nicht die des Repositories – dann
   in der linken Leiste **ganz unten** *Developer settings → Personal access tokens →
   Fine-grained tokens*.) Als Zugriff **nur dieses Repository** auswählen und als
   einzige Berechtigung **Contents: Read and write** setzen. Den Token (`github_pat_…`)
   sofort kopieren – er wird nur einmal angezeigt.
3. In der App: **Menü → „Online-Speicher einrichten"** – Repository
   (`benutzername/tagebuch-daten`), Token und dein Tagebuch-Passwort eintragen.
4. Auf jedem weiteren Gerät dieselben Angaben eintragen – vorhandene lokale Einträge
   werden dabei mit dem Online-Stand **zusammengeführt**, nichts wird überschrieben.

### Wie synchronisiert wird

- Automatisch **beim Entsperren**, **kurz nach jeder Änderung** und beim Wechsel
  zurück in die App; manuell über „Jetzt synchronisieren". Offline geschriebene
  Einträge werden nachgereicht, sobald wieder Netz da ist.
- Konflikte löst die App selbst: Einträge werden **pro Eintrag** zusammengeführt
  (bei gleichzeitiger Bearbeitung gewinnt die jüngste Version), Löschungen gelten
  auf allen Geräten, Stimmung und Markierungen werden **pro Tag** abgeglichen.
- Ein Cloud-Symbol im Kopfbereich zeigt an, dass der Sync aktiv ist (rot = Fehler,
  Details unter „Online-Speicher").

### Sicherheit des Online-Speichers

- Im Repository liegt **eine einzige Datei (`tagebuch-sync.json`) mit Ciphertext** –
  verschlüsselt mit einem zufälligen 256-Bit-Sync-Schlüssel (AES-GCM). GitHub sieht
  **keine Klartexte**, auch keine Kategorienamen.
- Der Sync-Schlüssel selbst liegt nur „verpackt" in der Datei: verschlüsselt mit einem
  aus deinem **Tagebuch-Passwort** abgeleiteten Schlüssel (PBKDF2-SHA-256). Deshalb kann
  jedes Gerät mit dem Passwort beitreten – und ohne Passwort niemand.
- Der GitHub-Token wird nur lokal gespeichert (verschlüsselt wie deine Einträge) und
  **niemals hochgeladen**. Läuft er ab, erneuerst du ihn unter „Online-Speicher".
- Die App besteht auf einem **privaten** Repository. Praktischer Nebeneffekt:
  Jeder Sync ist ein Git-Commit – du hast automatisch eine Versionshistorie.
- „Verbindung trennen" stoppt nur den Abgleich dieses Geräts; die Datei im Repository
  bleibt (und kann dort jederzeit gelöscht werden).

> Passwort geändert? Die App verpackt den Sync-Schlüssel automatisch neu und lädt das
> beim nächsten Sync hoch. Andere Geräte synchronisieren einfach weiter; nur ein **neues**
> Gerät braucht beim Beitritt das aktuelle Passwort.

## Datensicherheit / Backup

Der Gerätespeicher kann vom Browser geleert werden (Speicherdruck, „Website-Daten löschen").
Die App fragt daher `navigator.storage.persist()` an – **sichere deine Einträge trotzdem
regelmäßig über Menü → „Als Datei sichern (JSON)"**. Der Export ist unverschlüsselt (damit
importierbar/portabel) – behandle die Datei entsprechend vertraulich.

Ohne Online-Speicher heißt lokal: **pro Gerät** – der Weg zwischen Geräten führt dann über
Export/Import. Mit eingerichtetem Online-Speicher gleichen sich die Geräte automatisch ab;
der JSON-Export bleibt trotzdem dein Backup für den Fall der Fälle (z. B. Passwort vergessen
schützt auch der Sync nicht – die Online-Kopie ist genauso verschlüsselt).
