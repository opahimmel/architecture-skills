# Semantic Hooks Review — Check Details

## Check 1: Konsistenz

Slugs müssen kebab-case sein:
- `@invariant:NoIpcCalls` → flag
- `@boundary:fetch_user` → flag
- `@xp:canvas:grabEngine` → flag (Doppelpunkt nur als Trennzeichen für domain:slug)

```bash
grep -rn "@invariant:\|@boundary:\|@xp:" --include="*.ts" --include="*.tsx" .
```

## Check 2: Vollständigkeit

**Fehlende @boundary-Hooks** — Heuristik:
- Direkter IPC-Aufruf (`invoke(`, `ipcRenderer.send(`) ohne `@boundary:` darüber
- `fetch(`, `axios.`, `.query(`, `fs.readFile(` ohne `@boundary:`

**Fehlende @invariant-Hooks:**
- Datei macht ausschließlich Koordinaten-Mathematik → `@invariant:` fehlt wahrscheinlich
- Datei ist einziger Schreibpunkt in DB/IPC → Single-write-path prüfen

**Fehlende @xp-Links:**
- Wenn db.jsonl-Einträge mit `"files": ["src/foo.ts"]` existieren, aber foo.ts kein `@xp:` hat

## Check 3: Kreuzreferenzen

**@xp-Slugs gegen db.jsonl abgleichen:**
```bash
# Alle @xp-Slugs im Code
grep -rn "@xp:" --include="*.ts" --include="*.tsx" --include="*.js" . \
  | grep -oP "@xp:\K[a-z0-9:_-]+" | sort -u > /tmp/xp_code.txt

# Alle IDs in db.jsonl
jq -r '.id' .xp/db.jsonl | sort -u > /tmp/xp_db.txt 2>/dev/null \
  || grep -o '"id":"[^"]*"' .xp/db.jsonl | sed 's/"id":"//;s/"//' | sort -u > /tmp/xp_db.txt

# Differenz
comm -23 /tmp/xp_code.txt /tmp/xp_db.txt
```

Treffer = @xp-Slug im Code, aber kein Eintrag in db.jsonl → fragen.

## Check 4: Veraltete Hooks

**@xp-Slugs die in db.jsonl stehen aber nicht mehr im Code:**
```bash
comm -13 /tmp/xp_code.txt /tmp/xp_db.txt
```

Treffer → fragen. Kann intentional sein (Eintrag für gelöschtes Konzept als Histor).

**@invariant-Slugs:** Prüfen ob das beschriebene Verbot noch sinnvoll ist.
`@invariant:no-ipc-calls` in einer Datei die IPC-Calls gemacht hat → Widerspruch.

## Check 5: Symbol-Mirroring bei @boundary

Format muss sein: `@boundary:slug symbolName — beschreibung`

```bash
grep -rn "@boundary:" --include="*.ts" --include="*.tsx" . \
  | grep -v "@boundary:[a-z-]* [a-zA-Z]"
```

Trifft alle @boundary-Zeilen ohne Symbolname nach dem Slug.

**Auto-fix:** Symbol-Namen von der Zeile direkt darunter auslesen:
```
// @boundary:rust-ipc-write — Batched updates
source.write(snapshot)
→
// @boundary:rust-ipc-write source.write — Batched updates
source.write(snapshot)
```

**Ausnahme:** Hook über einem Block-Kommentar ohne direkte Symbol-Zeile → Ask.
