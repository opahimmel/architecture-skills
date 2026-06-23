---
name: semantic-hooks
description: Annotate codebases with minimal machine-readable hooks (@invariant, @boundary, @xp) so AI agents respect architectural constraints and navigate via XP-DB. Use when user mentions agent hooks, semantic annotations, wants to mark write-forbidden areas, or connect code to XP entries.
---

# Semantic Hooks

Drei Hooks — nicht mehr. Alles andere ist Rauschen das nie gegrepped wird.

## Die drei Hooks

### `@xp:slug` — XP-Verknüpfung (primäre Navigation)

Verbindet Code-Symbole mit XP-DB-Einträgen. Der einzige Hook der nachweislich wirkt:
Agent findet `@xp:input-layer-separation` in einer Datei → grepped sofort `input-layer-separation` in db.jsonl.

```typescript
// @xp:input-layer-separation
// @invariant:no-ipc-calls
function handlePointerDown(event: PointerEvent) { ... }
```

### `@invariant:slug` — Schreibverbot

Nur für echte Schreibverbote. Kein generisches "handle with care".

```typescript
// @invariant:no-ipc-calls — IPC nur über AppBridge, nie direkt hier
// @invariant:single-write-path — alle DB-Writes gehen durch source.write()
```

**Erlaubt:**
- `@invariant:no-ipc-calls`
- `@invariant:single-write-path`
- `@invariant:world-to-screen-math` (Koordinaten-Math nicht anfassen)

**Nicht erlaubt:**
- `@invariant:single-source-of-truth` — zu abstrakt, kein Schreibverbot
- `@invariant:keep-clean` — nichts sagend

### `@boundary:slug symbolName` — Systemgrenze

Inline, direkt über dem Aufruf. Symbol-Name wiederholen damit grep die Stelle findet.

```typescript
// @boundary:rust-ipc-write source.write — Batched position updates to DB
source.write(shapesSnapshot)
```

Nur an echten Systemgrenzen: IPC, fetch, DB, Filesystem. Nicht an internen Funktionsaufrufen.

## File-Header

**Kein Header mehr.** @domain, @owns, @depends, @routing, @extend, @side-effects, @context, @critical-path — alle entfernt. Navigation läuft über XP-DB.

Einzige Ausnahme: Wenn eine ganze Datei unter einem Invariant steht, kann `@invariant:` im Header stehen (eine Zeile):

```typescript
/**
 * @invariant:no-ipc-calls  @xp:input-layer-separation
 */
```

## Workflow: Datei annotieren

1. Lies die Datei — existiert ein XP-Eintrag? `grep "domain:slug" .xp/db.jsonl`
2. Wenn ja: `@xp:slug` an die zentrale Funktion/den zentralen Export
3. Schreibverbote für die ganze Datei → eine `@invariant:`-Zeile im Header
4. IPC/fetch/DB-Aufrufe → `@boundary:` direkt über dem Aufruf
5. Alles andere: weglassen

## Vor dem Schreiben in eine Datei

```bash
grep "@boundary\|@invariant" <zieldatei>
```

Zeigt alle aktiven Schranken. Treffer lesen — dann entscheiden.

## Regel

Symbol-Mirroring bei @boundary: Tag-Slug + Symbolname + Beschreibung.
`@boundary:slug symbolName — warum`
