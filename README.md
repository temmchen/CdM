# CdM-Portal

Verschlüsseltes Kurs-Portal der **Chambre des Métiers** (Brevet de Maîtrise).

- **Live:** https://temmchen.github.io/CdM/
- Drei Kurse: **Module F**, **Module M** und **Technische Mechanik MINT**
- Logins für Studenten (pro Kurs) und Kollegen; Verwaltung nur für den Admin
- Alle Inhalte liegen **AES-256-GCM-verschlüsselt** im Repo — im Klartext ist hier nichts
- **Bewertungen laufen über die CdM-Plattform; Bewertungsdateien gelangen nie ins Portal.**
  Einen Bereich „Noten" gibt es nicht (seit 23.09.2026 abends entfernt, stammte aus der
  Vorlage des Schuljahr-Portals)
- **Auf GitHub liegen NUR die Skripte für die Studierenden** (seit 01.11.2026). Prüfungen,
  Aufgaben, Sonstiges, Examen, Repêchage, Korrekturen, Berichte, Ergebnisse und Organisation
  bleiben lokal in OneDrive (Mac Studio, MacBook Pro) und erscheinen nur im lokalen
  CdM-Dashboard (Kachel „Korrekturen", CdM-Suche). Es gibt keine internen Tresore und kein
  Admin-Archiv mehr.
- Inhalte kommen aus OneDrive: `KI/en cours de travail études/CdM-Dashboard/<Jahr>/<Kurs>/<Modul>/<Bereich>/`
- Veröffentlicht wird immer nur das Jahr aus `aktuelles-jahr.txt`

Technik identisch zum Schuljahr-Portal (PBKDF2-SHA256 600k → AES-256-GCM,
Umschlag-Verfahren, inkrementeller Build).

## Nur Skripte online (seit 01.11.2026)

| | |
| --- | --- |
| Tresore | je Klasse genau einer: `skripte` (Kurs, Lese-Zugänge, Profs, Admin). Der frühere Tresor `intern` (Prüfungen, Aufgaben, Sonstiges) wird nicht mehr gebaut |
| Admin-Archiv | abgeschaltet. `build.py` löscht ein altes `docs/vaults/archiv.enc`; `archiv_fingerprint()` liefert fest `"abgeschaltet"`, damit `veroeffentlichen.py` nicht bei jedem Lauf neu baut |
| Lokale Bereiche | `Pruefungen`, `Aufgaben`, `Sonstiges` werden beim Build nur gemeldet („🔒 … bleiben lokal") |
| Portal-Seite | Upload nur nach „Skripte"; „Klasse anlegen" erzeugt nur einen Skripte-Tresor; die Karte „CdM-Suche" erscheint nur, wenn `index.json` noch ein Archiv nennt |
| Einsicht Korrekturen | ausschließlich lokal im CdM-Dashboard (`korrekturen.json`, `korrekturen.py`) |
| Historie | Am 01.11.2026 bereinigt: alle alten Stände von `docs/vaults` (interne Tresore, `archiv.enc`) sind aus der Git-Historie entfernt, die Code-Historie blieb. Sicherung vorher: `~/Documents/CdM-Portal-Sicherungen/CdM-Portal-vor-Bereinigung-2026-11-01.bundle` (nur lokal) |
| Zweiter Mac | Eine ältere Kopie wird vor jedem Pull angeglichen: `_Portal-Setup/historie_angleichen.py` (sichert als Bundle, dann `reset --hard origin/main`), aufgerufen vom Portal-Wächter und von „CdM veröffentlichen.command“; `veroeffentlichen.py` bricht bei alter Historie ab |

## Veröffentlichen (`veroeffentlichen.py`)

- holt vor dem Build die LaTeX-Skripte mit `sync_inhalte.py` aus dem Dashboard-Ordner
  (bis 23.09.2026 abends wurde das Skript im Jahresordner gesucht — der Sync lief nie)
- `BEREICHE_ONLINE = ["Skripte"]`; nur Änderungen dort lösen einen Build aus
- der Archiv-Fingerabdruck ist fest `"abgeschaltet"` (kein Archiv mehr)
- kein Noten-Schritt mehr (seit 23.09.2026 abends): `baue_noten_dashboard.py` wird nicht
  aufgerufen; ein alter `Noten/`-Ordner im aktiven Jahr wird nur gemeldet („ℹ️ … Datei(en) in
  Noten-Ordnern — bleiben offline") und nie hochgeladen (`build.py`: `NIE_HOCHLADEN = "Noten"`
  als Sicherheitsnetz). Die Schlussmeldung sagt, dass nur die Skripte online sind.
- `--erzwingen` nur noch, wenn sich allein der Code geändert hat
- `--pruefen` (Portal-Wächter): `OFFEN` oder `NICHTS-ZU-TUN`; zählt Skripte-Änderungen
  und ausstehende LaTeX-PDFs (`sync_inhalte.py --trocken`)

## Hinweise für Änderungen am Code

- Der Web-Upload schreibt über die Konstante `REPO` in `docs/index.html` — seit
  23.09.2026 korrekt `temmchen/CdM` (vorher fälschlich `temmchen/schuljahr-portal`).
- `veroeffentlichen.py` committet nur `docs/vaults`. Änderungen an `docs/index.html`
  oder `build.py` separat committen:
  `git add -- docs/index.html build.py && git commit -m "Code: …" && git push`;
  sollen die Tresore mit dem neuen Code neu gebaut werden: `python3 veroeffentlichen.py --erzwingen`.
  **Nie `git add docs`** (sonst gehen die ungetrackten `docs/*.bak-*` online) und **keine
  Commit-Nachricht mit „Portal:" oder „Portal-Upload" beginnen** — solche Commits hält
  `veroeffentlichen.py` auf dem anderen Mac für Browser-Aktionen und fragt dann [j/N].
- Seit 24.09.2026: `veroeffentlichen.py` lädt `build.py` erst nach dem `git pull`
  (`archiv_stand`), damit beide Macs nach einem Code-Update denselben Fingerabdruck rechnen;
  `build.py` gibt einer **geänderten** Datei eine **neue** Datei-ID (der Browser speichert
  Dateien dauerhaft zwischen — mit gleicher ID sähe man still die alte Fassung).
- Vorlagen-Reste bereinigt (23.09.2026 spätabends, Commit `64b2af5`): `verwaltung.py` legt
  für neue Klassen `Skripte`, `Pruefungen`, `Examen`, `Repechage`, `Aufgaben`, `Sonstiges`
  an (keine `Noten`-/`Referentiels`-Ordner), seine Texte und die `PASSWOERTER.md`-Vorlage
  sprechen nicht mehr von Noten; `docs/index.html` sagt beim Kollegen-Anlegen „sieht
  Skripte, Prüfungen, Aufgaben und Sonstiges aller Klassen"; der Docstring von
  `veroeffentlichen.py` nennt Schritt 6 nur noch als Hinweis. `build.py` behält
  `NIE_HOCHLADEN = "Noten"` als Sicherheitsnetz.
- Référentiels gibt es beim CdM nicht (`sync_inhalte.py` im Dashboard lehnt `Referentiels`
  als Ziel ab). Aus der Vorlage übrig und derzeit wirkungslos: der Référentiel-Code in
  `docs/index.html` und `web_einsammeln.py` sowie `"Referentiels"` in `BEREICHE_ONLINE`
  von `veroeffentlichen.py` — `build.py` liest keinen solchen Ordner.
