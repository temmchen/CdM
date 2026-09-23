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
| Inhalt | Link-Listen aller Jahrgänge und der Portal-Index der CdM-Suche — nur Dateinamen und OneDrive-Weblinks, **keine Inhalte** |
| nicht enthalten | Dateiinhalte; im Portal-Index keine Bewertungs- und Ergebnisdateien der Plattform-Exporte (Evaluations, Résultats, Grilles, Notenlisten) und keine JSON-Rohdaten (`portal_ausschluss` und `portal_arten_ausschluss` in `suche.json`, feste Namensmuster in `suche_index.export_portal`). Auch die Jahrgangs-Links lassen Bewertungsdateien weg: `archiv_daten` übernimmt aus den Bereichsordnern keine Datei, die `suche_index.art_aus` als Art „Bewertung" einstuft (dieselbe Regel wie im Portal-Index) — die Grilles/CSV in `2025-2026/…/Examen/` bleiben so rein lokal |
| Schutz | gzip + AES-256-GCM **ausschließlich unter dem Admin-Schlüssel**; Profs, Kurse und Lese-Zugang können es nicht öffnen |
| Anzeige | nur für den Admin: Jahrgangs-Auswahl im Kopf (Archiv-Ansicht) und Karte „CdM-Suche"; die Links öffnen nur mit der OneDrive-Anmeldung des Admins |
| Voraussetzung | Browser mit `DecompressionStream` (aktueller Safari/Chrome) |
| Fingerabdruck | `index.json["archiv"]["fp"]` — Hash über Jahrgangs-Links und Portal-Index, ohne Datum und ohne maschinenabhängige Felder |

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
  `git add docs/index.html build.py && git commit -m "…" && git push`;
  sollen die Tresore mit dem neuen Code neu gebaut werden: `python3 veroeffentlichen.py --erzwingen`
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
