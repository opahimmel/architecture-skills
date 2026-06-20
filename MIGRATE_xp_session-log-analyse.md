# XP: Session-Logs nach Kandidaten durchsuchen

Für Projekte in denen `/xp-setup` bereits ausgeführt wurde.
Analysiert vergangene Sessions um hochwertige XP-Einträge zu identifizieren.

Kopiere den folgenden Prompt in ein neues Gespräch im Ziel-Projekt:

---

```
Analysiere alle Session-Logs des aktuellen Projekts als XP-Kandidaten-Suche.

## Wo die Logs liegen

Das aktuelle Arbeitsverzeichnis bestimmt den Log-Pfad.
Leite ihn so ab:
  1. Nimm den absoluten Pfad des Projekts (z.B. /Users/fritzheydt/Desktop/mein-projekt)
  2. Ersetze alle / durch - (führenden Slash weglassen)
  3. Hänge es an: ~/.claude/projects/-Users-fritzheydt-Desktop-mein-projekt/

Die JSONL-Dateien darin (UUID.jsonl) sind die Sessions dieses Projekts.
Format: eine JSON-Zeile pro Nachricht, type-Felder: "user", "assistant", "tool_use", "tool_result"

---

## WICHTIG: Führe die drei Schritte strikt nacheinander aus.
Schließe jeden Schritt vollständig ab und präsentiere das Ergebnis,
bevor du mit dem nächsten beginnst. Warte nach jedem Schritt auf Bestätigung.

---

## Schritt 1 — Irrwege finden

Lies alle Sessions. Zähle für jede Session die Tool-Calls zwischen
erstem EXPLORE und erstem Edit/Write.
Sessions mit >8 Tool-Calls vor dem ersten Edit = "langer Anlauf".

Extrahiere pro Session:
- Was wurde als erstes gesucht?
- Welcher Pfad war falsch?
- Was war die tatsächliche Lösung?

Präsentiere die vollständige Liste aller langen Anläufe.
Warte dann auf Bestätigung bevor du mit Schritt 2 beginnst.

---

## Schritt 2 — Wiederholungsmuster finden

Erst wenn Schritt 1 bestätigt wurde.

Gehe durch alle Sessions und zähle:
- Welche Bash-Befehle / grep-Patterns tauchen mehrfach auf?
- Welche Dateipfade werden immer wieder zuerst gelesen?
- Welche Symbole, Config-Keys, Strukturen werden repeated gesucht?

Präsentiere die vollständige Häufigkeitstabelle.
Warte dann auf Bestätigung bevor du mit Schritt 3 beginnst.

---

## Schritt 3 — XP-Einträge formulieren

Erst wenn Schritt 2 bestätigt wurde.

Kombiniere die Irrwege aus Schritt 1 mit den Mustern aus Schritt 2.
Was taucht oft auf UND hat einen langen Anlauf? → höchste Priorität.

Output pro Kandidat:
KANDIDAT: <kurze Beschreibung>
BEWEIS: <wie oft / in welchen Sessions>
IRRWEG: <was wurde falsch versucht bevor die Lösung gefunden wurde>
XP-EINTRAG: {"id": "domain:slug", "note": "...", "files": ["..."], "file_hash": "...", "updated": "YYYY-MM-DD"}
```
