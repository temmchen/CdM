# CdM-Portal

Verschlüsseltes Kurs-Portal der **Chambre des Métiers** (Brevet de Maîtrise).

- **Live:** https://temmchen.github.io/CdM/
- Drei Kurse: **Module F**, **Module M** und **Technische Mechanik MINT**
- Logins für Studenten (pro Kurs) und Kollegen; Verwaltung nur für den Admin
- Alle Inhalte liegen **AES-256-GCM-verschlüsselt** im Repo — im Klartext ist hier nichts
- **Noten sind niemals online** (bleiben lokal in OneDrive)
- Inhalte kommen aus OneDrive: `KI/en cours de travail études/CdM-Dashboard/<Jahr>/<Kurs>/<Modul>/<Bereich>/`
- Veröffentlicht wird immer nur das Jahr aus `aktuelles-jahr.txt`

Technik identisch zum Schuljahr-Portal (PBKDF2-SHA256 600k → AES-256-GCM,
Umschlag-Verfahren, inkrementeller Build).

## Admin-Archiv und CdM-Suche (seit 23.09.2026)

| | |
| --- | --- |
| Datei | `docs/vaults/archiv.enc`, von `build.py` bei jedem Build neu geschrieben |
| Inhalt | Link-Listen aller Jahrgänge und der Dateiindex der CdM-Suche — nur Dateinamen und OneDrive-Weblinks, **keine Inhalte** |
| Schutz | gzip + AES-256-GCM **ausschließlich unter dem Admin-Schlüssel**; Profs, Kurse und Lese-Zugang können es nicht öffnen |
| Anzeige | nur für den Admin: Jahrgangs-Auswahl im Kopf (Archiv-Ansicht) und Karte „CdM-Suche"; die Links öffnen nur mit der OneDrive-Anmeldung des Admins |
| Voraussetzung | Browser mit `DecompressionStream` (aktueller Safari/Chrome) |

Den Suchindex erneuert `build.py` über `suche_index.py` aus dem Dashboard-Ordner.

## Hinweise für Änderungen am Code

- Der Web-Upload schreibt über die Konstante `REPO` in `docs/index.html` — seit
  23.09.2026 korrekt `temmchen/CdM` (vorher fälschlich `temmchen/schuljahr-portal`).
- `veroeffentlichen.py` committet nur `docs/vaults`. Änderungen an `docs/index.html`
  oder `build.py` separat committen:
  `git add docs/index.html build.py && git commit -m "…" && git push`
