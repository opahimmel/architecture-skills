---
name: xp-scan
description: Scannt die gesamte Codebasis autonom und schreibt Einträge für alle bedeutsamen Symbole in .xp/db.jsonl. Nutze es mit /xp-scan oder wenn eine frische Codebasis vollständig ins XP-System aufgenommen werden soll.
---

# xp-scan

Einmaliger autonomer Durchlauf. Keine Pausen. Alles wird gelabelt was semantisches Gewicht hat.

## Voraussetzung

`.xp/db.jsonl` muss existieren. Wenn nicht → zuerst `/xp-setup` aufrufen.

## Prozess

### 1. Codebasis kartieren

```bash
find . -type f \( -name "*.ts" -o -name "*.tsx" -o -name "*.js" -o -name "*.py" -o -name "*.go" -o -name "*.rs" \) \
  ! -path "*/node_modules/*" ! -path "*/.xp/*" ! -path "*/dist/*" ! -path "*/build/*" ! -path "*/__pycache__/*" \
  | sort
```

Bereits vorhandene Einträge in db.jsonl laden:
```bash
cat .xp/db.jsonl 2>/dev/null
```

### 2. Domains ableiten

Gruppiere Dateien nach Verzeichnis-Struktur. Jede logische Gruppe bekommt einen Domain-Prefix:

| Verzeichnis | Domain |
|-------------|--------|
| `src/auth/` | `auth` |
| `src/ui/components/` | `ui` |
| `src/api/` | `api` |
| `src/store/` | `store` |
| `lib/utils/` | `util` |
| Flache Struktur | Dateiname als Domain |

Passe Domains an die tatsächliche Projektstruktur an — kein Schema erzwingen.

### 3. Was wird gelabelt

**Aufnehmen:**
- Exportierte Interfaces, Types, Enums — besonders wenn sie als Datenstruktur durch das System fließen
- Service-Klassen, Store-Klassen, Manager-Klassen
- Funktionen die an mehreren Stellen konsumiert werden
- Haupt-Router, Controller, Handler
- Konfigurationsobjekte mit systemweiter Wirkung
- Error-Typen und -Hierarchien
- Utility-Funktionen mit nicht-offensichtlichem Verhalten

**Überspringen:**
- Einfache getter/setter ohne Logik
- Reine Typ-Aliases die nichts semantisch hinzufügen
- Testdateien
- Generierter Code
- Interne Hilfsvariablen

### 4. Pro Symbol: Note schreiben

Jede Note beantwortet drei Fragen in ein bis zwei Sätzen:

1. **Was ist es semantisch** — Rolle im System, nicht Syntax
2. **Womit connected es** — direkte Abhängigkeiten, Consumer, verwandte Konzepte
3. **Was steht nicht im Code** — implizite Verträge, Namens-Diskrepanzen, edge cases, Fallen

```json
{"id": "domain:slug", "keywords": ["natürlichsprachige", "suchbegriffe", "deutsch+englisch"], "note": "Semantische Rolle. Connections + warum. Fallen oder implizite Verträge.", "files": ["src/relevant.ts"], "file_hash": "abc123", "updated": "YYYY-MM-DD"}
```

`keywords` — deutschsprachige Begriffe aus der Aufgaben-Beschreibung, nicht Code-Symbole. Ziel: `grep -i "reihenfolge"` trifft den Eintrag `sequencer:resolve-chain`. Mindestens 3, maximal 8 Keywords pro Eintrag.

**File-Hash:**
```bash
git hash-object src/relevant.ts
```

### 5. Reihenfolge

Arbeite domain-weise durch. Innerhalb einer Domain: zuerst Datenstrukturen, dann Services, dann Utilities.

Für jede Domain:
1. Relevante Dateien lesen
2. Symbole identifizieren
3. db.jsonl-Einträge schreiben (Edit-Tool, append)

### 6. Abschluss-Report

Nach dem Durchlauf:

```
XP-Scan abgeschlossen
─────────────────────
Einträge neu:      23
Einträge updated:   4
Übersprungen:      11

Domains: auth (5), ui (8), api (6), store (4), util (4)
```

Alle neuen Einträge kompakt auflisten — id + erste Hälfte der Note.
