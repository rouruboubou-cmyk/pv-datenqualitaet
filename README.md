# PV-Erzeugungsdaten: Datenqualitätsprüfung und Marktwert

Eigenes Praxisprojekt mit Python, SQL (SQLite) und Plotly.

Das Projekt bildet in vereinfachter Form ab, wie Erzeugungsdaten von PV-Anlagen in einem Energiedatenmanagement verarbeitet werden: Daten kommen an, werden auf Vollständigkeit und Plausibilität geprüft, Lücken werden vorläufig ersetzt, und erst danach werden Energie und Marktwert ausgewertet.

## Ablauf

| Schritt | Inhalt |
|---|---|
| 1. Einrichtung | Bibliotheken und Projektpfade |
| 2. Daten laden | PVGIS-Zeitreihen, Anlagenstammdaten, Kundenzuordnung, Day-Ahead-Preise, Eingangskontrolle |
| 3. Lieferung simulieren | Kopie der Daten mit vier typischen Lieferfehlern |
| 4. Daten prüfen | SQL-Prüfungen (Lücken mit `LAG()`, doppelte Zeitstempel), Abgleich mit Soll-Zeitindex, Plausibilitätsregeln, Tagesbericht |
| 5. Ersatzwerte bilden | vorläufige Ersatzwerte, Validierung gegen die Originaldaten, Grafiken je Fehlerfall |
| 6. Auswerten | Energie, Marktwert und spezifischer Ertrag auf Basis der geprüften Daten (SQL) |
| 7. Berichten | Monatsübersicht, Kundendashboard, Export |

## Geprüfte Fehlerarten

| Fehler | Beispiel | Erkennung |
|---|---|---|
| Fehlende Werte | Hamburg, 15.06.2023, 3 Stunden | Soll-Zeitindex und SQL (`LAG()`) |
| Unplausibel hohe Werte | München, 15.06.2023, 2 Stunden | über 120 % der Nennleistung |
| Doppelter Zeitstempel | Dresden, 03.07.2023 | SQL (`GROUP BY … HAVING`) |
| Erzeugung in der Nacht | Erfurt, 20.08.2023, 3 Stunden | Vergleich mit Nachtstunden der Anlage |

Alle eingebauten Fehler werden erkannt; ein Selbsttest im Notebook prüft das automatisch. Der Tagesbericht ordnet jedem auffälligen Tag eine nächste Aktion zu („Daten beim MSB anfordern“ oder „Prüfung erforderlich“) und berücksichtigt die Zeitumstellung (23- und 25-Stunden-Tage).

## Kundendashboard

Im Notebook lässt sich die Auswertung je Kunde interaktiv filtern: PV-Anlagen, Zeitraum (Von/Bis), Auflösung (stündlich, täglich, monatlich, jährlich) und Kennzahl (Erzeugung oder Marktwert). Die Zeitachse folgt dem gewählten Zeitraum; lange stündliche Zeiträume lassen sich mit einem Schieberegler durchblättern. GitHub zeigt die interaktiven Grafiken nicht an, deshalb hier ein Screenshot (violett München, grün Hamburg):

![Kundendashboard](Bilder/dashboard.png)

## Fehlerfälle aus dem Notebook

**Fehlende Werte (Hamburg, 15.06.2023):** Drei Zeilen fehlen in der Lieferung und werden vorläufig ersetzt. Status: Daten beim MSB anfordern.

![Fehlende Werte Hamburg](Bilder/hamburg.png)

**Unplausible Werte (München, 15.06.2023):** Zwei Werte über 120 % der Nennleistung werden markiert und ersetzt. Status: Prüfung erforderlich.

![Unplausible Werte München](Bilder/muenchen.png)

**Doppelter Zeitstempel (Dresden, 03.07.2023):** Eine Stunde wurde zweimal mit unterschiedlichen Werten geliefert; beide werden verworfen und ersetzt. Status: Daten beim MSB anfordern.

![Doppelter Zeitstempel Dresden](Bilder/dresden.png)

**Erzeugung in der Nacht (Erfurt, 20.08.2023):** Drei Nachtstunden enthalten Erzeugung; die Werte werden auf 0 kWh korrigiert. Status: Prüfung erforderlich.

![Erzeugung in der Nacht Erfurt](Bilder/erfurt.png)

## Monatserzeugung 2023 (geprüfte Daten)

![Monatserzeugung](Bilder/monatserzeugung.png)

## Ergebnisse 2023

| Anlage | Leistung (kWp) | Erzeugung (MWh) | Ertrag (kWh/kWp) | Marktwert (€/MWh) |
|---|---:|---:|---:|---:|
| Dresden | 500 | 510,4 | 1.021 | 72,85 |
| Erfurt | 300 | 314,2 | 1.047 | 73,44 |
| Hamburg | 750 | 736,4 | 982 | 74,24 |
| München | 1.000 | 1.131,0 | 1.131 | 73,46 |

Der durchschnittliche Day-Ahead-Preis 2023 lag bei 95,19 €/MWh. Die Anlagen erzielen rund 77 % davon, weil PV vor allem mittags einspeist, wenn viele Anlagen gleichzeitig erzeugen und die Preise niedrig sind.

**Plausibilisierung:** Der offizielle Jahresmarktwert Solar 2023 lag bei 7,2 ct/kWh (72 €/MWh). Die simulierten Anlagen liegen mit 73–74 €/MWh in derselben Größenordnung.

**Warum nur 2023?** PVGIS stellt stündliche Zeitreihen nur für vergangene Jahre bereit, und bis September 2025 wurden die Day-Ahead-Preise stündlich ermittelt. So passen PV-Werte und Preise ohne Umrechnung zusammen. Das Notebook prüft, ob Preis- und PV-Daten aus demselben Jahr stammen.

## Projektstruktur

```
├── PV_Portfolio_Analytics_VSCode.ipynb   Notebook mit allen Schritten
├── requirements.txt                      benötigte Python-Pakete
├── Bilder/                               Grafiken für diese Übersicht
└── Daten/
    ├── Lastgänge/                        PVGIS-Zeitreihen je Anlage (CSV)
    ├── Preise/                           Day-Ahead-Preise DE-LU (SMARD-Export, CSV)
    └── Stammdaten/                       Kundenzuordnung (CSV, fiktiv)
```

Beim Ausführen legt das Notebook zusätzlich `Daten/Export`, `Daten/Datenqualität` und die Datenbank `PV_Portfolio.db` an.

## Ausführen

```
python -m venv .venv
.venv\Scripts\python.exe -m pip install -r requirements.txt
```

Anschließend das Notebook in VS Code oder Jupyter öffnen, den Kernel `.venv` auswählen und alle Zellen ausführen.

## Grenzen

- Die Erzeugungsdaten sind stündlich und simuliert; reale Messwerte liegen meist als Viertelstundenwerte vor.
- Reale Messwerte kommen über die Marktkommunikation (z. B. MSCONS) mit Marktlokations-IDs und Korrekturlieferungen.
- Schwellenwerte und Nacht-Referenz sind vereinfachte Projektannahmen.
- Die Ersatzwertmethode (Mittelwert aus Vor- und Folgetag) ist bei wechselndem Wetter ungenau und nicht abrechnungsrelevant.
- Der Marktwert ist eine Modellrechnung, kein tatsächlich erzielter Erlös. Für eine reale Erlösberechnung kämen u. a. Marktprämie, § 51 EEG (negative Preise), Direktvermarktungskosten und Redispatch-Ausgleich hinzu (siehe Notebook, Abschnitt 6.3).
- Kunden und Fehlerfälle sind fiktiv.

## Datenquellen

- PV-Erzeugung: PVGIS, © Europäische Union, Joint Research Centre
- Day-Ahead-Preise: Bundesnetzagentur | SMARD.de (CC BY 4.0)
