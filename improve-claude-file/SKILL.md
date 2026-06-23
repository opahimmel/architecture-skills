---
name: improve-claude-file
description: Analysiert das aktuelle Projekt und schreibt CLAUDE.md radikal neu — präzise, session-schonend, verweist auf PROJECT_MAP.md und XP-Datenbank statt Inhalte zu duplizieren. Nutze es mit /improve-claude-file oder wenn CLAUDE.md zu lang, zu vage, überladen oder nicht mehr repräsentativ für das Projekt ist.
---

# improve-claude-file

Schreibt CLAUDE.md neu. Ein Agent der diese Datei liest versteht WAS das Projekt IST — nicht wo Dateien liegen.

## Systemschichten

CLAUDE.md trägt nur was die anderen nicht tragen:

| Datei | Zuständigkeit |
|-------|--------------|
| **CLAUDE.md** | Identität — Was, warum, was gilt immer |
| **PROJECT_MAP.md** | Navigation — Architektur, Einstiegspunkte |
| **.xp/db.jsonl** | Wissen — nicht-offensichtliche Konzepte, Entscheidungen, Muster |
| **@invariant / @boundary / @xp** | Schreibschutz & Wissensverweise direkt im Code |

CLAUDE.md wiederholt NIE was in Map oder XP-DB steht.

## Prozess

### 1. Analyse

```bash
cat CLAUDE.md 2>/dev/null; echo "---"; cat PROJECT_MAP.md 2>/dev/null; echo "---"; ls -la; git log --oneline -5 2>/dev/null
```

Lese außerdem: package.json / pyproject.toml / go.mod / Cargo.toml — was vorhanden ist.

Prüfe ob `.xp/db.jsonl` existiert. Falls ja: Inhalt überfliegen — was ist bereits als Wissen erfasst?

Bewerte: Was ist klar? Was fehlt? Was dupliziert Map oder XP-DB? Was würde ein Agent ohne diesen Kontext falsch machen?

### 2. Grill-Session (max 5 Fragen, einzeln stellen)

**Situationsabhängig:**
- Projekt klar → nur Lücken füllen (was fehlt, blinde Flecken)
- CLAUDE.md vage oder überladen → hinterfragen (ist das wirklich der Kern?)

Themen wenn unklar — jede Frage mit eigener Empfehlung:
1. **Identität** — Was ist der Kern in einem Satz?
2. **Nutzungskontext** — Wer nutzt es wie — Dev, CI, Agent?
3. **Hard Constraints** — Was darf ein Agent hier NIE tun?
4. **Nicht-offensichtlich** — Was überrascht neue Agents systematisch?
5. **Abgrenzung** — Was ist dieses Projekt explizit NICHT?

Wenn die Antwort bereits aus dem Code, der Map oder der XP-DB folgt → Frage überspringen.

### 3. CLAUDE.md schreiben

**Regeln:**
- Max 40 Zeilen gesamt
- Kein Pfad, keine Dateiliste (→ PROJECT_MAP.md)
- Keine Konzepterklärungen die in XP-DB gehören
- Jeder Constraint konkret, nicht abstrakt

**Pflichtstruktur:**

```markdown
# [Projektname]

[1-2 Sätze: Was ist das. Für wen. Was es tut.]

## Kontext

[Wie wird es genutzt — 2-3 Sätze. Dev-Workflow, Produktion, Agent-Tool.]

## Hard Constraints

- [Was ein Agent NIE tun darf — präzise formuliert]

## Navigation

Wenn `.xp/db.jsonl` im Repo-Root existiert:

**Schritt 1 — XP-Datenbank**
`grep -i "<aufgaben-keyword>" .xp/db.jsonl`
Bei Treffer der die Frage direkt beantwortet: sofort handeln, kein weiterer Lookup.

**Schritt 2 — nur wenn Schritt 1 keinen Treffer liefert**
Wenn `@xp:slug` im Quellcode auftaucht: `grep "slug" .xp/db.jsonl` — Wissens-Infusion für dieses Konzept.

**Schritt 3 — nur wenn Schritt 2 keinen Treffer liefert**
Dateinamen direkt aus dem Kontext lesen und File öffnen.

**Bevor du Code in eine Datei schreibst**: `grep "@boundary\|@invariant" <zieldatei>` — welche Schranken gelten.

Prinzipien gelten beim Schreiben — nicht als Checkliste beim Lesen. Wenn die Task lautet "prüfe ob X existiert", reicht ein Treffer. Kein Durchverifizieren der Kette.

## Systemschichten

- Navigation & Architektur → PROJECT_MAP.md
- Wissen & Konzepte → .xp/db.jsonl
```

**Optional wenn nötig:**
```markdown
## Nicht-offensichtlich

[Das eine das einen Agent ohne diesen Hinweis falsch laufen lässt]
```

### 4. Semantic Hooks bereinigen (wenn vorhanden)

Wenn Hooks im Code existieren, gilt: **weniger ist mehr.**

**Raus** — nie als Navigation-Target genutzt, XP-DB macht das besser:
- `@domain:` / `@owns:` / `@depends:` / `@routing:` / `@extend:` / `@side-effects:` / `@context:` / `@critical-path:`

**Bleibt:**
- `@invariant:` — nur wenn er ein echtes Schreibverbot ausdrückt (`@invariant:no-ipc-calls`). Abstrakte Formulierungen wie `@invariant:single-source-of-truth` raus.
- `@boundary:` — direkt über der Funktion/Naht, nicht im Datei-Header.
- `@xp:slug` — der einzige Hook der nachweislich wirkt: Agent findet `@xp:dual-write-paths`, grepped sofort in db.jsonl.

Ziel: Header von 8–10 Zeilen auf 1–2 Zeilen.

```js
// Vorher
/**
 * @domain:canvas-input  @xp:domain-label-uniqueness
 * @owns:dom-events,hit-testing
 * @depends:canvas-input-actions,app-state
 * @side-effects:none
 * @critical-path:true
 * @invariant:no-ipc-calls  @xp:input-layer-separation
 */

// Nachher
/**
 * @invariant:no-ipc-calls  @xp:input-layer-separation
 */
```

### 5. Partner aktivieren

- **`/map-create`** — wenn PROJECT_MAP.md fehlt
- **`/xp-setup`** — wenn `.xp/db.jsonl` fehlt und nicht-offensichtliches Wissen im Projekt steckt

### 6. Validierung vor dem Schreiben

- [ ] Liest sich in 30 Sekunden komplett
- [ ] Navigation-Sektion mit XP-Lookup-Kette vorhanden (wenn .xp existiert)
- [ ] Kein Inhalt der in PROJECT_MAP.md oder .xp/db.jsonl gehört
- [ ] Jeder Constraint konkret, nicht abstrakt
- [ ] Wenn PROJECT_MAP.md fehlt → Hinweis: "Nächster Schritt: /map-create"
- [ ] Wenn .xp fehlt und Wissen vorhanden → Hinweis: "Nächster Schritt: /xp-setup"
