# Semantic Hooks: Symbol-Mirroring nachrüsten

Für Codebasen in denen `/semantic-hooks` bereits installiert ist.
Bringt alle bestehenden Inline-Hooks auf das neue Symbol-Mirroring-Format.

Kopiere den folgenden Prompt in ein neues Gespräch im Ziel-Projekt:

---

```
Migriere alle Inline Semantic Hooks in dieser Codebase auf Symbol-Mirroring.

**Problem:** Hooks ohne Symbolnamen sind grep-blind — der Agent findet sie nicht bei normaler
Symbol-Suche.

**Altes Format:**
// @routing:pointer-down — Main entry for all mouse interactions
function pointerDown(event) { ... }

**Neues Format:**
// @routing:pointer-down pointerDown — Main entry for all mouse interactions
function pointerDown(event) { ... }

**Schritt 1 — Alle grep-blinden Hooks finden:**
```bash
grep -rn "@routing:\|@boundary:\|@extend:\|@invariant:" --include="*.ts" --include="*.tsx" \
  --include="*.js" --include="*.jsx" . \
  | grep -v "@[a-z]*:[a-z-]* [a-zA-Z]"
```

Jede Zeile in diesem Output ist ein Hook ohne Symbolnamen.

**Schritt 2 — Für jeden Treffer:**
1. Zeile darunter lesen → das ist das Symbol (Funktionsname, Variable, Methodenname)
2. Symbol nach dem Slug einfügen: @tag:slug symbolName — description
3. Wenn die Folgezeile kein klares Symbol enthält (z.B. Block-Anfang, Leerzeile) → zeige mir die
   Stelle, fix nicht automatisch

**Schritt 3 — Report:**
Liste alle geänderten Stellen im Format:
- datei.ts:141  @routing:pointer-down → @routing:pointer-down pointerDown

Nur Inline-Tags (@routing, @boundary, @extend, @invariant) sind betroffen.
File-level Headers (@domain, @owns, @depends, @side-effects) — nicht anfassen.
```
