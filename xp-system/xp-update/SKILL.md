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
| Bereits in db.jsonl? (`grep "slug" .xp/db.jsonl`) | Upsert — alte Zeile wird ersetzt |
| Eintrag veraltet / irrelevant? | Zeile aktiv löschen (Upsert ohne Anhängen) oder weglassen |
| Note würde nichts über den lesbaren Code hinausgehen? | trotzdem aufnehmen — semantische Rolle + Connections zählen |

### 3. Eintrag schreiben (Upsert)

**Format:**
```json
{"id": "domain:slug", "keywords": ["natürlichsprachige", "suchbegriffe", "deutsch+englisch"], "note": "Semantische Rolle. Connections + warum. Fallen oder implizite Verträge.", "files": ["src/relevant.ts"], "file_hash": "abc123", "updated": "YYYY-MM-DD"}
```

`keywords` — deutschsprachige Begriffe aus der Aufgaben-Beschreibung, nicht Code-Symbole. Ziel: `grep -i "reihenfolge"` trifft den Eintrag `sequencer:resolve-chain`. Mindestens 3, maximal 8 Keywords pro Eintrag.

**File-Hash ermitteln:**
```bash
git hash-object src/relevant.ts
```

**Upsert — bestehende ID ersetzen, neue anhängen:**
```bash
python3 -c "
import json, sys
new = json.loads(sys.argv[1])
lines = [l for l in open('.xp/db.jsonl') if json.loads(l)['id'] != new['id']]
lines.append(json.dumps(new, ensure_ascii=False) + '\n')
open('.xp/db.jsonl', 'w').writelines(lines)
" '<JSON>'
```

Kein Append ohne Prüfung. Kein `status: obsolet`. Keine Duplikate.

**Die Note — Dichte über Vollständigkeit:**

Alles in einem bis zwei Sätzen. Drei Aspekte, kein Fülltext.

❌ `"Shape ist ein Interface in types.ts"` — Syntax, keine Semantik  
❌ `"Heute Shape refactored"` — Session-Tagebuch  
✅ `"Zentrale Canvas-Datenstruktur — in DB als canvas_object gespeichert (Namens-Diskrepanz). ShapeNormalizer muss vor jedem render-Call laufen, sonst crash. Consumer: RenderEngine, ExportService."` — Rolle + Connections + Falle  
✅ `"Einziger Einstiegspunkt für alle API-Fehler — Handling woanders wird nicht gefangen. Verbindet sich mit Logger und ResponseBuilder. Namenskonvention: E_USER_ prefix für user-facing errors."` — Semantik + Topologie + Vertrag  

### 4. Anzeigen

Alle neuen oder geänderten Einträge zeigen.
