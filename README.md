# Architecture Skills for Claude Code

> Skills die AI-Agenten ermöglichen eine Codebase **wirklich** zu verstehen —
> nicht nur Dateien zu lesen, sondern Architektur zu navigieren.

---

## Das Problem

Ein AI-Agent der in deine Codebase taucht, sieht Code — aber nicht **Bedeutung**.

Er sieht `handlePointerDown()` — aber nicht dass das der einzige erlaubte Einstieg für Mouse-Events ist.  
Er sieht `source.write()` — aber nicht dass das ein IPC-Call nach Rust ist, der Batching braucht.  
Er sieht `CLAUDE.md` — aber nicht was vor drei Wochen falsch lief und warum.

Das Ergebnis: der Agent bricht Invarianten. Er dupliziert Logik. Er stellt Fragen die du schon 10x beantwortet hast. Er braucht dich als Korrektiv statt als Richtungsgeber.

Diese Skills lösen das — durch drei Schichten die sich nicht überlappen:

```
Schicht 1: CLAUDE.md          → Identität: Was ist das Projekt? Was gilt immer?
Schicht 2: .xp/db.jsonl       → Erfahrung: Was hat die Arbeit an diesem Repo gelehrt?
Schicht 3: Semantic Hooks      → Seams: Wo sind die Nahtstellen im Code?
```

Jede Schicht beantwortet andere Fragen. Zusammen geben sie einem Agenten ein vollständiges Bild — ohne Redundanz, ohne Token-Verschwendung.

---

## Warum das besser ist als alles in CLAUDE.md zu schreiben

Das klassische Muster: alles in CLAUDE.md stopfen. Architektur, Konventionen, Fallstricke, History.

Das Problem damit:

- **CLAUDE.md wird in jede Session geladen** — jedes Detail kostet Context-Budget
- **Details veralten schnell** — nach dem Refactoring stimmt die Hälfte nicht mehr
- **Der Agent findet nicht was er sucht** — 300-Zeilen-CLAUDE.md ist ein Blob, kein Navigationssystem

Die Alternative: Information dort platzieren wo sie entsteht und gebraucht wird.

```
Architectural boundary in code?     → @boundary: direkt an der Stelle
Non-obvious lesson from a session?  → .xp/db.jsonl, greppbar nach Topic
What this project IS?               → CLAUDE.md, kurz und stabil
```

Der Agent kann gezielt grepen statt blind zu lesen. `grep "@boundary:" src/` zeigt alle Grenzen in einer Zeile.

---

## Die Skills

### `semantic-hooks` — Code annotieren

Maschinenlesbare Tags direkt im Code. Der Agent sieht die Architektur dort wo sie passiert.

```typescript
/**
 * @domain:      canvas-input
 * @owns:        dom-events,hit-testing
 * @depends:     input-actions,state
 * @invariant:   no-ipc-calls
 */

// @routing:pointer-down pointerDown — Main entry for all mouse interactions
// @boundary:rust-ipc-write source.write — Batched position updates to DB
// @extend:shape-types SHAPE_TYPES — Add new shapes here
// @invariant:world-to-screen-math worldToScreen — Do not touch without tests
```

`grep "pointerDown"` liefert die Routing-Zeile automatisch mit — kein separater Lookup nötig.

**Wann:** Neues Modul ohne Kontext. Agent hat eine Grenze gebrochen. Codebase soll agent-ready werden.  
**Aufruf:** `/semantic-hooks`

---

### `semantic-hooks-review` — Hooks aktuell halten

Prüft bestehende Hooks auf Konsistenz, Vollständigkeit, Cross-References und Veralterung. Fixt automatisch fehlende Symbolnamen (Symbol-Mirroring).

**Wann:** Nach Refactoring. Bei Architecture Reviews. Wenn mehrere Personen Hooks geschrieben haben.  
**Aufruf:** `/semantic-hooks-review`

---

### `xp-setup` — XP-Datenbank anlegen

Legt `.xp/db.jsonl` an und erweitert CLAUDE.md mit dem Nutzungsabschnitt. Einmalig pro Repo.

```json
{"id": "auth:token-flow", "note": "JWT wird in Redis gecacht, nicht in der DB. Consumer erwarten immer ein gültiges Token — null wirft einen 500.", "files": ["src/auth/token.ts"], "updated": "2026-06-20"}
```

**Wann:** Neues Repo. Erster AI-Agent-Einsatz.  
**Aufruf:** `/xp-setup`

---

### `xp-update` — Insights nach Session eintragen

Reflektiert die Session, filtert nicht-offensichtliche Insights, schreibt direkt in `.xp/db.jsonl`.

Was hineingehört: **semantische Rollen** (nicht Syntax), **implizite Verträge**, **Fallen** die ein Agent ohne Hinweis wiederholen würde.

Was nicht hineingehört: Implementierungsdetails, Session-Tagebuch, was im Code selbst steht.

**Wann:** Am Ende jeder Session.  
**Aufruf:** `/xp-update`

---

### `improve-claude-file` — CLAUDE.md schärfen

Schreibt CLAUDE.md neu: präzise, kurz, ohne Redundanz zu XP-DB oder Hooks. CLAUDE.md beantwortet nur noch: *Was ist dieses Projekt? Warum existiert es? Was gilt immer?*

**Wann:** CLAUDE.md ist aufgebläht. Nach größerem Refactoring.  
**Aufruf:** `/improve-claude-file`

---

## Empfohlene Reihenfolge

```
1. /xp-setup              → XP-Datenbank anlegen
2. /semantic-hooks         → kritische Module annotieren
3. /improve-claude-file    → CLAUDE.md schärfen
4. /xp-update              → nach jeder Session
5. /semantic-hooks-review  → nach jedem Refactoring
```

---

## Installation

```bash
cp -r semantic-hooks ~/.claude/skills/
cp -r semantic-hooks-review ~/.claude/skills/
cp -r improve-claude-file ~/.claude/skills/
cp -r xp-system/xp-setup ~/.claude/skills/
cp -r xp-system/xp-update ~/.claude/skills/
```

**XP-System:** Die globale `~/.claude/CLAUDE.md` muss um den XP-Abschnitt erweitert werden —
ohne ihn wissen Agents nicht dass die Datenbank existiert.
Details in [`xp-system/README.md`](xp-system/README.md).

---

## Migration bestehender Codebases

| Datei | Was sie tut |
|-------|------------|
| [`MIGRATE_semantic-hooks_symbol-mirroring.md`](MIGRATE_semantic-hooks_symbol-mirroring.md) | Findet alle grep-blinden Inline-Hooks und fügt den Symbolnamen ein |
| [`MIGRATE_xp_session-log-analyse.md`](MIGRATE_xp_session-log-analyse.md) | Analysiert vergangene Sessions auf Irrwege und Muster → generiert XP-Einträge |

Datei öffnen → Inhalt kopieren → in neuem Gespräch im Ziel-Projekt einfügen.

---

## Update Juni 2026

- **semantic-hooks:** Inline-Hooks tragen jetzt den Symbolnamen direkt im Marker (Symbol-Mirroring). `grep "pointerDown"` liefert damit automatisch den Hook.
- **semantic-hooks-review:** Check 5 prüft Symbol-Mirroring und fixt fehlende Symbolnamen automatisch.
- **xp-setup / xp-update:** Neues Eintrag-Format: `domain:slug`-IDs, `note` statt `insight`, `files` + `file_hash` für Staleness-Erkennung.
