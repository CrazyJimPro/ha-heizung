# ha-heizung – Raum-Heizung als Home-Assistant-Blueprint

Ein Blueprint, aus dem du **pro Raum eine Heizungs-Automation** anlegst. Alles wird in der Oberfläche eingestellt, ohne YAML:

- Heizungsregler des Raums
- ein oder mehrere Fensterkontakte (mit Verzögerung)
- Hauptschalter (z. B. „Heizmodus“)
- Außentemperatur mit Grenzwert
- bis zu **6 Zeitfenster**, jedes mit eigener Uhrzeit, eigenen Wochentagen und eigener Temperatur
- Verhalten außerhalb der Zeitfenster: Regler aus, Absenktemperatur oder nicht eingreifen (Handbetrieb)
- optional: Zeitplan für eine Dashboard-Anzeige in einen Text-Helfer schreiben („heizt noch 2 h“, „ab 17:00 auf 23 °C“)

## So entscheidet die Automation

Bei jedem Auslöser (Fenster auf/zu, Beginn oder Ende eines Zeitfensters, Außentemperatur über/unter dem Grenzwert, Hauptschalter, HA-Start, Regler wieder erreichbar) wird neu bewertet. Die erste Regel, die zutrifft, gewinnt:

| Reihenfolge | Wenn … | dann … |
|---|---|---|
| 1 | Hauptschalter ist aus | Regler aus (oder „nichts tun“, einstellbar) |
| 2 | ein Fenster ist offen | Regler aus |
| 3 | draußen ist es so warm wie der Grenzwert oder wärmer | Regler aus |
| 4 | ein aktives Zeitfenster passt | heizen mit dessen Temperatur |
| 5 | sonst | Regler aus, Absenktemperatur oder nicht eingreifen |

Geht das Fenster wieder zu, gilt sofort wieder, was der Zeitplan gerade vorsieht.

### Handbetrieb („Nicht eingreifen“)

Für Räume ohne festen Zeitplan, deren Regler du von Hand bedienst. Außerhalb der Zeitfenster (oder ganz ohne Zeitfenster) lässt die Automation den Regler in Ruhe. Nur Fenster, Hauptschalter und Außentemperatur greifen ein:

- Geht ein Fenster auf, merkt sich die Automation Modus und Solltemperatur des Reglers und schaltet ihn aus.
- Sind alle Fenster wieder zu, stellt sie den gemerkten Zustand wieder her.
- Hast du den Regler in der Zwischenzeit selbst wieder eingeschaltet, bleibt deine Einstellung.
- Der gemerkte Zustand übersteht keinen Neustart von Home Assistant. Wird HA neu gestartet, während das Fenster offen ist, bleibt der Regler danach aus.
Der Regler bekommt nur dann einen Befehl, wenn sich Modus oder Solltemperatur wirklich ändern. Das schont die Batterien der Thermostate.

## Voraussetzungen

- Home Assistant **2025.4** oder neuer
- pro Raum ein `climate`-Regler und mindestens ein Fenster-/Türkontakt (`binary_sensor`)
- ein Schalter-Helfer (`input_boolean`) als Hauptschalter
- ein Außentemperatur-Sensor (z. B. OpenWeatherMap)

## Blueprint importieren

1. Auf diesen Knopf klicken und die eigene Home-Assistant-Adresse bestätigen:

   [![Blueprint importieren](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2FCrazyJimPro%2Fha-heizung%2Fblob%2Fmain%2Fblueprints%2Fautomation%2Fcrazyjimpro%2Fraum_heizung.yaml)

   Ohne den Knopf: **Einstellungen → Automationen & Szenen → Blueprints → Blueprint importieren** und diese Adresse einfügen:

   ```
   https://github.com/CrazyJimPro/ha-heizung/blob/main/blueprints/automation/crazyjimpro/raum_heizung.yaml
   ```

2. **Vorschau** und danach **Blueprint importieren** klicken.

## Einen Raum anlegen

1. **Einstellungen → Automationen & Szenen → Blueprints**, dann bei **Raum-Heizung** auf **Automation erstellen**.
2. Bereich **Raum**: Regler und Fensterkontakte wählen. Bei Bedarf eine Verzögerung einstellen (z. B. 30 Sekunden, damit kurzes Lüften nicht sofort abschaltet).
3. Bereich **Hauptschalter**: den Heizmodus-Schalter wählen.
4. Bereich **Außentemperatur**: Sensor wählen und den Grenzwert prüfen (Standard 20 °C).
5. Bereiche **Zeitfenster 1–6**: Bei jedem gewünschten Zeitfenster **Aktiv** einschalten, dann Von, Bis, Wochentage und Temperatur eintragen. Nicht benutzte Zeitfenster bleiben aus.
6. Bereich **Außerhalb der Zeitfenster**: „Regler ausschalten“, „Absenktemperatur halten“ oder „Nicht eingreifen (Handbetrieb)“ wählen.
7. Optional, Bereich **Anzeige im Dashboard**: einen Text-Helfer wählen (siehe nächster Abschnitt).
8. **Speichern**. Einen Namen vergeben, z. B. „Heizung Schlafzimmer“.

Für jeden weiteren Raum die Schritte 1–8 wiederholen.

## Zeitplan im Dashboard anzeigen (optional)

Ein Dashboard kann die Einstellungen einer Automation nicht lesen. Damit eine Karte trotzdem zeigen kann, wie lange ein Raum noch heizt oder ab wann er heizt, schreibt die Automation ihre aktiven Zeitfenster in einen Text-Helfer.

1. **Einstellungen → Geräte & Dienste → Helfer → Helfer erstellen → Text**.
2. Name z. B. „Heizplan Schlafzimmer“, **Maximale Länge** auf **255** stellen, erstellen.
3. Die Automation des Raums öffnen, im Bereich **Anzeige im Dashboard** diesen Helfer wählen, speichern.

Der Helfer wird sofort nach dem Speichern gefüllt und danach bei jedem Lauf aktuell gehalten. Er enthält eine Liste im Format `[[von, bis, temperatur, [wochentage]], …]`: Von und Bis in Minuten ab Mitternacht, Wochentage als Zahlen (0 = Montag … 6 = Sonntag). Beispiel: `[[1035,1170,25,[0,1,2,3,4,5,6]]]` heißt täglich 17:15–19:30 mit 25 °C. Bei einem Zeitfenster über Mitternacht ist Bis kleiner als Von.

Eine Karte liest den Plan mit `states('input_text.heizplan_schlafzimmer') | from_json`.

### Hinweise zu den Zeitfenstern

- „Bis“ gehört nicht mehr zum Zeitfenster. Zwei Zeitfenster 17:15–19:30 und 19:30–21:15 schließen also nahtlos aneinander an.
- Liegt „Bis“ vor „Von“ (z. B. 22:00–06:00), läuft das Zeitfenster über Mitternacht. Als Wochentag zählt dabei der Tag, an dem es gerade ist.
- Überschneiden sich zwei Zeitfenster, gewinnt das mit der kleineren Nummer.

## Ändern oder entfernen

- **Einstellungen ändern**: die Automation öffnen, Werte anpassen, speichern.
- **Blueprint aktualisieren**: **Blueprints** öffnen, beim Blueprint über das Drei-Punkte-Menü **Blueprint erneut importieren** wählen. Alle Räume übernehmen die neue Version.
- **Entfernen**: zuerst alle Automationen löschen, die den Blueprint nutzen, danach den Blueprint selbst.

## Wenn etwas nicht klappt

- **Was hat die Automation gemacht?** Die Automation öffnen, oben rechts das Drei-Punkte-Menü → **Traces**. Dort steht, welcher Auslöser kam und welche Regel gegriffen hat (z. B. „Fenster offen“ oder „Zeitfenster 2“).
- **Heizung geht gar nicht an**: Ist der Hauptschalter an? Ist es draußen wärmer als der Grenzwert? Ist beim passenden Zeitfenster **Aktiv** eingeschaltet und stimmen die Wochentage?
- **Zwei Automationen steuern denselben Regler**: Alte Fenster-Automationen, Zeitpläne (z. B. Scheduler-Karten) oder Test-Automationen für denselben Raum ausschalten. Sonst überschreiben sie sich gegenseitig.
- **Plan-Helfer bleibt leer**: Ist beim Helfer die maximale Länge auf 255 gestellt? Bei der Standardlänge 100 passen nur zwei bis drei Zeitfenster hinein, dann schreibt die Automation nichts.
- **Fehlermeldungen**: **Einstellungen → System → Protokolle**, nach `automation` oder dem Namen der Automation filtern.

## Lizenz

MIT
