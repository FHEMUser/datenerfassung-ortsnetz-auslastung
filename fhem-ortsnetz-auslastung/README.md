# FHEM

Ein einzelnes FHEM-`at` liest drei Spannungs-Readings und sendet sie alle fünf Minuten per HTTPS an die Ortsnetz-Auslastung-API. Es werden nur die Pflichtfelder übertragen: Zeitstempel, ungefährer Standort und die Spannungen L1, L2 und L3. Ein eigenes Modul und Einträge in `99_myUtils.pm` sind nicht nötig.

- Datei: [`at_ortsnetz.txt`](at_ortsnetz.txt)
- Voraussetzungen: FHEM, drei Geräte mit Spannungs-Reading (zum Beispiel Shelly-Geräte, je eines pro Phase), Perl-Modul `IO::Socket::SSL` für HTTPS und Internetzugang

## Einrichtung

1. In `at_ortsnetz.txt` die Werte oben im Block anpassen:
   - `$name`: exakter Name des `at`-Devices (hier `at_ortsnetz`)
   - `$lat` und `$lon`: ungefährer Standort, siehe Abschnitt Koordinaten im [Haupt-README](../README.md)
   - `@src`: pro Phase Device-Name und Reading mit der Spannung in Volt (je nach Gerät zum Beispiel `voltage` oder `voltage_0`)
   - `$einphasig`: auf `1` setzen, wenn nur eine Phase gemessen wird
2. Den Inhalt in die FHEM-Kommandozeile einfügen. In der `fhem.cfg` muss jede Zeile des Blocks außer der letzten mit `\` enden.

## Verhalten

- Gesendet wird sofort nach dem Anlegen und danach alle fünf Minuten.
- Ein Wert gilt als gültig, wenn er größer als 0 und höchstens 500 V ist und das Reading nicht älter als 10 Minuten ist.
- Im Dreiphasen-Betrieb (`$einphasig = 0`) wird nur gesendet, wenn alle drei Werte gültig sind. Sonst wird übersprungen, damit keine Phase falsch zugeordnet wird.
- Mit `$einphasig = 1` wird nur die erste Quelle aus `@src` als `l1_v` gesendet, `l2_v` und `l3_v` stehen auf `-1`.
- Im `at`-Device entstehen die Readings `l1_v`, `l2_v`, `l3_v` (Rohwerte, auch ungültige) und `api_status` (`HTTP 202` bei Annahme). Bei Fehlern steht mit `verbose 3` ein Eintrag im Log, bei `verbose 5` auch die Antwort des Servers.

## Hinweise

- Jedes Gerät sollte dauerhaft derselben Phase zugeordnet bleiben, damit die Messreihe konsistent bleibt.
- Die Spannung wird dort gemessen, wo das Gerät angeschlossen ist. Ein Gerät nahe am Hausanschluss liefert aussagekräftigere Werte als eines am Ende einer langen Leitung.
- Die Frequenz und weitere optionale Felder werden in dieser Integration nicht gesendet.
