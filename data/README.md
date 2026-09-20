# Daten – Struktur & Zugriff (Single Source of Truth)

> Zweck: Damit jede Session die **historischen Daten zuverlässig findet** und nichts mehr „verloren" geht. Am 20.09.2026 kam es zu einem Fehler, weil eine Messung (Bauchumfang 18.09.) nur auf einem parallelen Branch lag und der Arbeitsstand veraltet war. Diese Datei + die Regeln unten verhindern das künftig.

## ⚠️ Regel Nr. 1 – vor jeder Bearbeitung synchronisieren
Es existieren teils **mehrere parallele `claude/*`-Branches**. Bevor du Werte liest oder schreibst:
1. `git fetch --all`
2. Prüfe, ob Messwerte fehlen: `git log --all --oneline -- data/koerperwerte.csv`
3. Fehlende Messungen aus anderen Branches zusammenführen (mergen), **bevor** du analysierst.
Die **Deadline-/Ziel-Wahrheit** steht in `../CLAUDE.md` und `../Ziele/Primäres-Ziel-Aktiv.md`.

## Kanonische Datenquellen (maschinenlesbar zuerst)
| Datei | Inhalt | Rolle |
|-------|--------|-------|
| `data/koerperwerte.csv` | Alle Körperwaagen- + Bauchumfang-Messungen | **SoT für Körperwerte** – hier zuerst nachsehen |
| `data/training.csv` | Trainings-/Aktivitäts-Einträge (Apple Fitness etc.) | **SoT für Training** |
| `../Körperwerte/Messungen-Tracker.md` | Menschlich lesbare Tabelle + Trend-Analysen | Spiegel/Analyse der CSV (nicht die Rohquelle) |
| `../Tageslog/YYYY-MM-DD.md` | Tages-Check-in (Gewicht, Training, Ernährung, Reflexion) | Tagesdoku |
| `../Ernährung/Essenslog/YYYY-MM-DD.md` | Detailliertes Essenslog + Netto-Defizit | Tagesdoku |
| `../Fortschritt/Fortschrittsübersicht.md` | Aggregierte Fortschritts-/Ampel-Übersicht | Aggregat |

## Schema `koerperwerte.csv`
Kopfzeile (kommagetrennt), **Feld-Index in Klammern** – wichtig fürs Parsen:
1. `Datum` (YYYY-MM-DD)
2. `Gewicht_kg`
3. `Koerperfett_pct`
4. `FettMasse_kg`
5. `FettfreieMasse_kg`
6. `SubkutanesFett_pct`
7. `VisceralFett`
8. `SkelettMuskel_pct`
9. `Muskelmasse_kg`
10. `Knochenmasse_kg`
11. `Koerperwasser_pct`
12. `BMR_kcal`
13. `BMI`
14. `MetabolischesAlter`
15. **`Bauchumfang_cm`** ← der Sixpack-Indikator
16. `Notiz` (Freitext)

### ⚠️ Parsing-Hinweis (häufige Fehlerquelle)
`Notiz` (Feld 16) ist **Freitext und enthält Kommas** → naive `cut -d, -fN` / `$(NF-1)`-Zugriffe liefern **falsche** Werte. Regeln:
- Der **Bauchumfang ist immer Feld 15**, gezählt vom Zeilenanfang (`awk -F, '{print $15}'`), **nicht** von hinten.
- Alles ab dem 16. Komma gehört zur `Notiz`.
- Teilmessungen sind erlaubt: leere Felder = an dem Tag nicht erfasst (z. B. nur Bauchumfang gemessen, ohne Wiegen).

### Konventionen
- Eine Zeile pro Messtag. Nicht gemessene Werte bleiben **leer** (kein Platzhalter, kein erfundener Wert).
- Tage ganz ohne Messung bekommen **keine** CSV-Zeile (nur ggf. eine Notiz im Tracker/Tageslog).
- `Messungen-Tracker.md` wird passend zur CSV gepflegt; bei Abweichung gilt die **CSV**.

## Wichtige Referenzwerte (Stand 20.09.2026)
- Bauchumfang-Verlauf: 87 (12.07.) → 86 (16.07.) → **85,5** (23.07. = Juli-Allzeit-Tief) → **85,5** (18.09.) → **85,0 (20.09. = neues Allzeit-Tief)**.
- Baseline Challenge (08.09.): 67,35 kg / 18,6 % KF. Deadline: 04.10.2026.
