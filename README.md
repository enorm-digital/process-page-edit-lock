# PageEditLock

Temporärer Bearbeitungs-Lock für ProcessWire 3.x (getestete Zielversion: 3.0.255).

## Zweck

Verhindert, dass zwei Benutzer dieselbe Seite gleichzeitig bearbeiten und dass ein sehr lange offen gebliebener Editor später die zwischenzeitlichen Änderungen eines anderen Benutzers überschreibt.

### Verhalten

- Beim Öffnen des Page Editors wird ein Lock angelegt.
- Der Browser sendet regelmäßig einen Heartbeat.
- Wird der Tab lange im Hintergrund gehalten, darf der Lock ablaufen.
- Beim Zurückkehren in den Tab wird der Lock sofort erneut geprüft.
- Ist der Lock verloren, wird der Editor gesperrt und ein Reload verlangt.
- Auch serverseitige Speichervorgänge werden geprüft.
- Ein gemeinsam genutzter ProcessWire-Account wird über eine zusätzliche Browser-Client-ID auf unterschiedlichen Browsern/Rechnern unterschieden.
- Beim Schließen des Tabs wird der Lock best-effort freigegeben; der Timeout bleibt die Sicherheitsinstanz.
- Superuser können optional einen aktiven Lock übernehmen.
- Templates können ausgenommen werden.

## Installation

1. Verzeichnis `PageEditLock` nach `/site/modules/` kopieren.
2. Im ProcessWire-Admin zu **Modules** gehen.
3. **Check for New Modules** ausführen.
4. **Page Edit Lock** installieren.
5. Modul konfigurieren.

## Empfohlene Werte

- Lock timeout: 120 Sekunden
- Heartbeat: 20 Sekunden

Der Timeout sollte deutlich größer als der Heartbeat sein.

## Wichtiger Hinweis

Die Browser-Client-ID liegt in `localStorage`. Sie ist keine Sicherheits- oder Authentifizierungs-ID. Die eigentliche Berechtigung bleibt die ProcessWire-Session.

Das Modul ist bewusst als Schutz gegen versehentliches paralleles Bearbeiten konzipiert. Vor dem Produktiveinsatz sollte es auf der konkreten ProcessWire-Installation mit normalen Saves, AJAX-Saves, Repeatern und ggf. weiteren Admin-Modulen getestet werden.

## Technische Hinweise

Das Modul verwendet die hookbaren Methoden von `ProcessPageEdit` und eine eigene InnoDB-Tabelle. Die Tabelle hat einen Unique-Key auf `page_id`, damit zwei gleichzeitig eintreffende Lock-Anforderungen nicht beide erfolgreich sein können.

Beim Speichern wird der Lock serverseitig erneut geprüft. Das ist wichtig, weil ein Browser-Tab für Stunden oder Tage offen bleiben kann und JavaScript in Hintergrund-Tabs gedrosselt werden darf.

## Schutz vor Überschreiben

Beim Lock wird der aktuelle `pages.modified`-Zeitstempel als Ausgangswert gespeichert. Vor jedem normalen oder AJAX-Speichern wird der aktuelle Datenbankwert verglichen. Wenn die Seite inzwischen von einer anderen Sitzung geändert wurde, wird das Speichern abgebrochen und ein Reload verlangt. Nach einem erfolgreichen eigenen Speichern wird der Ausgangswert aktualisiert.
