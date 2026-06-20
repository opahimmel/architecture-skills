---
name: xp-setup
description: Richtet das XP-System im aktuellen Repo ein — .xp/db.jsonl und Marker-Konvention. Einmalig pro Projekt. Nutze es mit /xp-setup oder wenn ein Repo noch keine .xp/db.jsonl hat.
---

# xp-setup

Einmalige Infrastruktur. Kein Inhalt — Inhalt kommt über `/xp-update`.

## Prozess

### 1. Prüfen ob bereits vorhanden

```bash
ls .xp/db.jsonl 2>/dev/null
```

Existiert bereits → abbrechen, Nutzer informieren.

### 2. Datei anlegen

```bash
mkdir -p .xp
```

Leere `db.jsonl` anlegen. Kein Beispiel-Eintrag.

### 3. CLAUDE.md des Projekts erweitern

Wenn `CLAUDE.md` im Repo-Root existiert, folgenden Abschnitt anhängen. Wenn nicht, neu anlegen.

```markdown
## XP-Datenbank

Wenn `.xp/db.jsonl` existiert:

- Vor dem ersten File-Lookup: `grep -i "<aufgaben-keyword>" .xp/db.jsonl`
- Wenn `@xp:domain:slug` im grep-Output erscheint: sofort `grep "domain:slug" .xp/db.jsonl` ausführen
- Eintrag lesen und als gesichertes Wissen behandeln — nicht neu herleiten
```

### 4. Bestätigung

Zeige dem Nutzer Pfad der angelegten Datei und den eingefügten CLAUDE.md-Abschnitt.

---

## Format

Ein Eintrag pro Konzept, eine Zeile:

```json
{"id": "domain:slug", "note": "Wo es lebt + wie es connected ist + warum + Fallen. Ein bis zwei Sätze.", "files": ["relevant/file.ts"], "file_hash": "abc123", "updated": "YYYY-MM-DD"}
```

- `id` — namespaced slug: `domain:konzept` (z.B. `auth:token-flow`, `ui:shape`)
- `note` — alles in einem Satz: Ort, Verbindungen, Semantik, Fallen
- `files` — die eine Quelldatei wo das Konzept definiert ist
- `file_hash` — SHA256 der Quelldatei: `git hash-object <file>`
- `updated` — Datum des Eintrags

## Marker-Konvention

Marker direkt vor dem Symbol, Symbol-Name im Kommentar:

```ts
// @xp:ui:shape — Shape
export interface Shape { ... }
```

```python
# @xp:auth:token-flow — validate_token
def validate_token():
```

**Warum der Symbol-Name im Kommentar:** Wenn der Agent nach `Shape` grepped, erscheint der Marker automatisch in der Zeile davor. Der Agent sieht `@xp:ui:shape` im grep-Output bevor er die Datei öffnet — ohne explizite Instruktion.
