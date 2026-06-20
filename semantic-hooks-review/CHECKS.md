# Semantic Hooks — Check Details

## Check 1: Konsistenz

Tag-Werte müssen kebab-case sein — alles andere ist ein Fehler:
- `@domain:CamelCase` → flag
- `@domain:snake_case` → flag
- `@owns:some module` (Leerzeichen) → flag
- Prosa in Tag-Werten (Verben, Sätze) → flag
- Doppelte Tags im selben File-Header → flag (keep first)

**Grep-Basis:**
```bash
grep -rn "@domain:\|@owns:\|@depends:\|@side-effects:\|@context:" --include="*.ts" .
```

## Check 2: Vollständigkeit

**Fehlendes File-Header** — Heuristik für "architekturkritische" Datei:
- Mehr als 3 Imports UND wird von mehr als 2 anderen Dateien importiert
- Enthält eine zentrale Switch/Match-Struktur (>5 Cases)
- Macht direkte IPC-, fetch- oder DB-Aufrufe

**Fehlende Inline-Hooks:**
- Switch/if-else-Kette >5 Branches ohne `@routing:` → fehlt
- Fetch/IPC/DB-Aufruf ohne `@boundary:` → fehlt
- Union Type / Enum der bei jedem neuen Feature erweitert wird ohne `@extend:` → fehlt
- Kritische Math/Koordinaten-Logik ohne `@invariant:` → fehlt

## Check 3: Kreuzreferenzen

**Domain-Graph aufbauen:**
1. Alle `@owns:X` sammeln → Ownership-Map: `{X → [file]}`
2. Alle `@depends:X` sammeln → Dependency-Map: `{X → [file]}`

**Drei Fehlertypen:**

| Problem | Beschreibung |
|---------|--------------|
| Orphaned ownership | A `@owns:X`, aber kein File hat `@depends:X` |
| Dangling dependency | B `@depends:X`, aber kein File hat `@owns:X` |
| Ownership conflict | A und B beide `@owns:X` |

**Hinweis:** Orphaned ownership kann intentional sein (extern konsumiert). Immer fragen.

## Check 4: Veraltete Hooks

**Vorgehen:**
1. Alle Domain-Labels aus `@domain:`, `@owns:`, `@depends:` extrahieren
2. Für jeden Label prüfen: existiert er noch irgendwo als `@domain:X` in der Codebase?
3. Labels die in `@owns:` / `@depends:` auftauchen aber nirgends als `@domain:X` deklariert sind → potenziell veraltet
4. Zusätzlich: Prüfe ob referenzierte Dateinamen / Modul-Pfade (z.B. in `@context:`) noch existieren

**Ausnahmen nicht flagen:**
- `@side-effects:none` → kein Label
- `@critical-path:true/false` → kein Domain-Label

---

## Check 5: Symbol-Mirroring

**Ziel:** Inline hooks müssen den Symbol-Namen im Marker wiederholen, damit `grep "symbolName"` den Hook automatisch mitliefert.

**Altes Format (grep-blind):**
```typescript
// @routing:pointer-down — Main entry for all mouse interactions
function pointerDown(event) { ... }
```

**Neues Format (grep-sichtbar):**
```typescript
// @routing:pointer-down pointerDown — Main entry for all mouse interactions
function pointerDown(event) { ... }
```

**Grep für alte Hooks ohne Symbol-Namen:**
```bash
grep -rn "@routing:\|@boundary:\|@extend:\|@invariant:" --include="*.ts" --include="*.tsx" . \
  | grep -v "@[a-z]*:[a-z-]* [a-zA-Z]"
```

Trifft jede Zeile mit einem Inline-Hook-Tag, die nach dem Slug **keinen** Bezeichner (Wort-Zeichen) enthält.

**Fix-Regel:** Auto-fix — Symbol-Namen von der Zeile direkt darunter auslesen und nach dem Slug einfügen.

```
// @boundary:rust-ipc-write — Batched updates
source.write(snapshot)
→
// @boundary:rust-ipc-write source.write — Batched updates
source.write(snapshot)
```

**Ausnahme:** Hooks ohne nachfolgende Symbol-Zeile (z.B. Block-Kommentare über einem ganzen Abschnitt) → Ask, nicht auto-fix.
