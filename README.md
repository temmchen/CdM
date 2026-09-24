# CdM-Portal

Verschlüsseltes Kurs-Portal der **Chambre des Métiers** (Brevet de Maîtrise).

- **Live:** https://temmchen.github.io/CdM/
- Drei Kurse: **Module F**, **Module M** und **Technische Mechanik MINT**
- Logins für Studenten (pro Kurs) und Kollegen; Verwaltung nur für den Admin
- Alle Inhalte liegen **AES-256-GCM-verschlüsselt** im Repo — im Klartext ist hier nichts
- **Bewertungen laufen über die CdM-Plattform; Bewertungsdateien gelangen nie ins Portal.**
  Einen Bereich „Noten" gibt es nicht (seit 23.09.2026 abends entfernt, stammte aus der
  Vorlage des Schuljahr-Portals)
- Online sind nur **Skripte, Pruefungen, Aufgaben, Sonstiges** — Examen, Repêchage und
  Organisation bleiben lokal in OneDrive
- Inhalte kommen aus OneDrive: `KI/en cours de travail études/CdM-Dashboard/<Jahr>/<Kurs>/<Modul>/<Bereich>/`
- Veröffentlicht wird immer nur das Jahr aus `aktuelles-jahr.txt`

Technik identisch zum Schuljahr-Portal (PBKDF2-SHA256 600k → AES-256-GCM,
Umschlag-Verfahren, inkrementeller Build).

## Admin-Archiv und CdM-Suche (seit 23.09.2026)

| | |
| --- | --- |
| Datei | `docs/vaults/archiv.enc`, von `build.py` bei jedem Build neu geschrieben — **vor** den Tresoren |
| Inhalt | Link-Listen **aller 31 Jahresordner** (2025-2026 … 2055-2056, auch leere; seit 24.09.2026) und der Portal-Index der CdM-Suche — nur Dateinamen und OneDrive-Weblinks, **keine Inhalte** |
| nicht enthalten | Dateiinhalte; im Portal-Index keine Bewertungs- und Ergebnisdateien der Plattform-Exporte (Evaluations, Résultats, Grilles, Notenlisten) und keine JSON-Rohdaten (`portal_ausschluss` und `portal_arten_ausschluss` in `suche.json`, feste Namensmuster in `suche_index.export_portal`). Die Jahrgangs-Links (Bereiche und Organisation) nutzen seit 24.09.2026 **genau dieselbe Regel** wie der Portal-Index: `suche_index.portal_ausgeschlossen` (Art „Bewertung", JSON, Ordner-Wortstämme, Namensmuster wie result/cand_/passw) — die Grilles/CSV in `2025-2026/…/Examen/` bleiben so rein lokal |
| Schutz | gzip + AES-256-GCM **ausschließlich unter dem Admin-Schlüssel**; Profs, Kurse und Lese-Zugang können es nicht öffnen |
| Anzeige | nur für den Admin: **Schuljahr-Auswahl** im Kopf mit allen Jahren (★ aktiv · Archiv · kommend, mit Dateizahl) — je Jahr dieselbe Ansicht: Kurs-Karten, Organisation, Hauptverzeichnisse, „Dateien dieses Schuljahres in OneDrive" (aufklappbar), **„Jahresunabhängig"** und die CdM-Suche für dieses Jahr; im aktiven Jahr unter den Kacheln „Nur in OneDrive" (Examen/Repêchage mit Dateien, Organisation, Hauptverzeichnisse, Jahresdateien, Jahresunabhängig). Die Links öffnen nur mit der OneDrive-Anmeldung des Admins |
| Jahresregel | Das Jahr steht im **Pfad**: Jahresordner (`2026-2027/…`), sonst das Jahr im Wurzelnamen (`CdM 2025-2026`). Alles andere ist **jahresunabhängig** (`New CdM/Organisation` mit Rechnungen, Dashboard-`Skripte` …) und erscheint in **jedem** Jahr; README/LIESMICH/`aktuelles-jahr.txt` zählen nicht. Das geschätzte Feld `j` des Index wird dafür bewusst nicht benutzt |
| ↻ Aktualisieren | Knopf neben der Schuljahr-Auswahl, für alle angemeldeten Rollen: lädt `index.json`, Manifeste, Archiv und Zugangsliste **frisch von GitHub** (`?t=…` am Browser- und Pages-Zwischenspeicher vorbei), ohne Abmelden; Ansicht, Klasse und Schuljahr bleiben. Direkt nach einer eigenen Änderung in der Web-Verwaltung holt er `index.json` über die GitHub-API (Pages braucht 1–2 min). Neue Dateien aus OneDrive kommen erst mit dem nächsten 🚀 ins Portal |
| Entsperren | Jeder Mac-Build zieht neue Salze — eine ältere Admin-Sitzung öffnet dann Zugangsliste und Archiv nicht mehr. Statt „ab- und neu anmelden" erscheint oben die Karte **„Portal wurde neu veröffentlicht"**: einmal das Admin-Passwort eingeben (nicht gespeichert). Bis dahin sperrt die Verwaltung „Klasse/Kollege anlegen, Passwort würfeln, Zugang entfernen" — sonst entstünden Einträge unter einem veralteten Schlüssel. Ein Netzfehler beim Archiv zeigt keine Entsperr-Karte, sondern „Archiv fehlt" |
| Voraussetzung | Browser mit `DecompressionStream` (aktueller Safari/Chrome) |
| Fingerabdruck | `index.json["archiv"]["fp"]` — Hash über Jahrgangs-Links und Portal-Index, ohne Datum (`stand`, seit 24.09.2026 minutengenau) und ohne maschinenabhängige Felder |
| öffentlich | Im Klartext von `index.json["archiv"]` stehen die Namen der Jahresordner, die Zahl der Index-Dateien, `stand` und `fp` — keine Datei- oder Personennamen |

Den Suchindex erneuert `build.py` über `suche_index.py` aus dem Dashboard-Ordner.
Scheitert das Archiv (OneDrive-Suchwurzel fehlt, Index länger als 90 s gesperrt), bricht
`build.py` ab, **bevor** etwas in `docs/vaults` geschrieben wird.

## Veröffentlichen (`veroeffentlichen.py`)

- holt vor dem Build die LaTeX-Skripte mit `sync_inhalte.py` aus dem Dashboard-Ordner
  (bis 23.09.2026 abends wurde das Skript im Jahresordner gesucht — der Sync lief nie)
- vergleicht den Archiv-Fingerabdruck (`build.archiv_fingerprint`) mit
  `.letzter-stand.json["archiv"]`; bei Abweichung wird auch ohne Änderung in den
  Online-Bereichen gebaut und gepusht („… Archiv/CdM-Suche erneuert"); gemerkt wird der
  Wert des tatsächlich gebauten Archivs
- bricht ab, wenn der Fingerabdruck nicht berechenbar ist, obwohl `suche_index.py`
  vorhanden ist — dann wird nichts veröffentlicht
- kein Noten-Schritt mehr (seit 23.09.2026 abends): `baue_noten_dashboard.py` wird nicht
  aufgerufen; ein alter `Noten/`-Ordner im aktiven Jahr wird nur gemeldet („ℹ️ … Datei(en) in
  Noten-Ordnern — bleiben offline") und nie hochgeladen (`build.py`: `NIE_HOCHLADEN = "Noten"`
  als Sicherheitsnetz). Schlussmeldung: „(Examen, Repêchage und Organisation bleiben wie
  immer offline.)"
- `--erzwingen` nur noch, wenn sich allein der Code geändert hat
- `--pruefen` (Portal-Wächter): `OFFEN` oder `NICHTS-ZU-TUN`; zählt auch Archiv-Änderungen
  und ausstehende LaTeX-PDFs (`sync_inhalte.py --trocken`); baut dabei den lokalen
  Suchindex neu

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
