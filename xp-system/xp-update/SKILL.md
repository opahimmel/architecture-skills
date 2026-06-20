---
name: xp-update
description: Schreibt Wissen-Infusionen aus der aktuellen Session in .xp/db.jsonl. Nutze es mit /xp-update am Session-Ende oder wenn der Nutzer explizit darum bittet.
---

# xp-update

Direkt schreiben, dann Marker-Snippet zeigen. Kein Explore. Nur Session-Wissen.

## Voraussetzung

`.xp/db.jsonl` muss existieren. Wenn nicht → zuerst `/xp-setup` aufrufen.

## Zweck

Das XP-System ist ein semantischer Grundriss der Codebasis. Jeder Eintrag beschreibt ein gelabeltes Symbol so, dass ein neuer Agent nach einem grep in 30 Sekunden versteht was es ist, womit es verbunden ist und was nicht im Code steht.

Nicht Ausnahme-Wissen. Basis-Wissen.

## Prozess

### 1. Was wird eingetragen?

Jedes Symbol das in dieser Session mit einem `@xp`-Marker versehen wurde oder bereits einen trägt und dessen Note veraltet oder unvollständig ist.

Ein Eintrag beantwortet drei Fragen:

| Frage | Was reingehört |
|-------|---------------|
| **Was ist es?** | Semantische Rolle, nicht Syntax — was das Symbol im System bedeutet |
| **Womit connected es?** | Direkte Abhängigkeiten, Consumer, verwandte Konzepte im System |
| **Was steht nicht im Code?** | Implizite Verträge, Namens-Diskrepanzen, edge cases, Fallen |

### 2. Filter

| Prüfung | Ergebnis |
|---------|---------|
| Bereits in db.jsonl? (`grep "slug" .xp/db.jsonl`) | überspringen oder updaten |
| Note würde nichts über den lesbaren Code hinausgehen? | trotzdem aufnehmen — semantische Rolle + Connections zählen |

### 3. Eintrag schreiben

**Format:**
```json
{"id": "domain:slug", "note": "Semantische Rolle. Connections + warum. Fallen oder implizite Verträge.", "files": ["src/relevant.ts"], "file_hash": "abc123", "updated": "YYYY-MM-DD"}
```

**File-Hash ermitteln:**
```bash
git hash-object src/relevant.ts
```

**Die Note — Dichte über Vollständigkeit:**

Alles in einem bis zwei Sätzen. Drei Aspekte, kein Fülltext.

❌ `"Shape ist ein Interface in types.ts"` — Syntax, keine Semantik  
❌ `"Heute Shape refactored"` — Session-Tagebuch  
✅ `"Zentrale Canvas-Datenstruktur — in DB als canvas_object gespeichert (Namens-Diskrepanz). ShapeNormalizer muss vor jedem render-Call laufen, sonst crash. Consumer: RenderEngine, ExportService."` — Rolle + Connections + Falle  
✅ `"Einziger Einstiegspunkt für alle API-Fehler — Handling woanders wird nicht gefangen. Verbindet sich mit Logger und ResponseBuilder. Namenskonvention: E_USER_ prefix für user-facing errors."` — Semantik + Topologie + Vertrag  

### 4. Marker-Snippet generieren

Nach dem Schreiben: zeige den Marker den der Nutzer direkt vor das Symbol im Code einfügt.

```
Eintrag: ui:shape

Marker für den Code:
// @xp:ui:shape — Shape
```

Format: `// @xp:<id> — <SymbolName>`

Der Symbol-Name im Kommentar sorgt dafür dass der Marker automatisch im grep-Output erscheint wenn der Agent nach dem Symbol sucht — ohne explizite Instruktion.

### 5. Anzeigen

Alle neuen oder geänderten Einträge zeigen. Marker-Snippets pro Eintrag.
