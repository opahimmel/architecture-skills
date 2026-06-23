---
name: semantic-hooks-review
description: Audits existing semantic hooks (@invariant, @boundary, @xp) for consistency, completeness, cross-references, and staleness. Use when hooks are already in place and need a health check, after a refactor, or when running an architecture review. Builds on the semantic-hooks skill.
---

# Semantic Hooks Review

Builds on [semantic-hooks](../semantic-hooks/SKILL.md). Assumes hooks are already present.

## Quick start

```bash
grep -rn "@invariant:\|@boundary:\|@xp:" \
  --include="*.ts" --include="*.tsx" --include="*.js" --include="*.jsx" .
```

## Workflow

1. **Scan** — grep für alle aktiven Hook-Tags, File+Zeilen-Inventar aufbauen
2. **Analyse** — fünf Checks ausführen (siehe [CHECKS.md](CHECKS.md))
3. **Report** — strukturierte Findings, gruppiert nach Check
4. **Fix** — Edits direkt anwenden, jede Änderung im Report loggen

## Fix-Regeln

| Issue | Aktion |
|-------|--------|
| Kebab-case-Verletzung in Slug | Auto-fix |
| `@boundary:` ohne Symbol-Name | Auto-fix — Symbol von Folgezeile |
| `@xp:` ohne db.jsonl-Eintrag | Ask — Eintrag fehlt oder Slug falsch |
| `@invariant:` zu abstrakt | Ask — Architekturentscheidung |
| Veralteter Slug (nicht mehr im Code) | Ask — kann intentional sein |
| `@invariant:` Änderung jeglicher Art | Immer Diff zeigen, immer fragen |

## Report-Format

```
## Semantic Hooks Review — <datum>

### 1 Konsistenz       (N issues)
- src/input.ts:8  @invariant:NoIpcCalls → @invariant:no-ipc-calls

### 2 Vollständigkeit  (N missing)
- src/ipc.ts:44  direkter IPC-Aufruf ohne @boundary:

### 3 Kreuzreferenzen  (N conflicts)
- src/canvas.ts:3  @xp:canvas-grab-engine — kein Eintrag in db.jsonl

### 4 Veraltete Hooks  (N stale)
- src/auth.ts:8  @xp:old-auth-flow — Slug nicht mehr in db.jsonl

### 5 Symbol-Mirroring (N issues)
- src/api.ts:12  @boundary:fetch-user — kein Symbol-Name

### Actions taken
- Fixed N / M issues direkt
- N Items brauchen manuelle Prüfung (oben gelistet)
```
