# ha-heizung – Raum-Heizung als Home-Assistant-Blueprint

Ein Blueprint, aus dem du **pro Raum eine Heizungs-Automation** anlegst. Alles wird in der Oberfläche eingestellt, ohne YAML:

- Heizungsregler des Raums
- ein oder mehrere Fensterkontakte (mit Verzögerung)
- Hauptschalter (z. B. „Heizmodus“)
- Außentemperatur mit Grenzwert
- bis zu **6 Zeitfenster**, jedes mit eigener Uhrzeit, eigenen Wochentagen und eigener Temperatur
- Verhalten außerhalb der Zeitfenster: Regler aus oder Absenktemperatur

## So entscheidet die Automation

Bei jedem Auslöser (Fenster auf/zu, Beginn oder Ende eines Zeitfensters, Außentemperatur über/unter dem Grenzwert, Hauptschalter, HA-Start, Regler wieder erreichbar) wird neu bewertet. Die erste Regel, die zutrifft, gewinnt:

| Reihenfolge | Wenn … | dann … |
|---|---|---|
| 1 | Hauptschalter ist aus | Regler aus (oder „nichts tun“, einstellbar) |
| 2 | ein Fenster ist offen | Regler aus |
| 3 | draußen ist es so warm wie der Grenzwert oder wärmer | Regler aus |
| 4 | ein aktives Zeitfenster passt | heizen mit dessen Temperatur |
| 5 | sonst | Regler aus oder Absenktemperatur |

Geht das Fenster wieder zu, gilt sofort wieder, was der Zeitplan gerade vorsieht.
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
6. Bereich **Außerhalb der Zeitfenster**: „Regler ausschalten“ oder „Absenktemperatur halten“ wählen.
7. **Speichern**. Einen Namen vergeben, z. B. „Heizung Schlafzimmer“.

Für jeden weiteren Raum die Schritte 1–7 wiederholen.

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
- **Fehlermeldungen**: **Einstellungen → System → Protokolle**, nach `automation` oder dem Namen der Automation filtern.

## Lizenz

MIT
