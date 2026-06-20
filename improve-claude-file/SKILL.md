---
name: improve-claude-file
description: Analyzes the current project and rewrites CLAUDE.md from scratch — concise, session-efficient, delegating navigation to PROJECT_MAP.md and Semantic Hooks instead of duplicating content. Use when CLAUDE.md is too long, too vague, overloaded, or no longer representative of the project, or when invoking /improve-claude-file.
---

# improve-claude-file

Schreibt CLAUDE.md neu. Ein Agent der diese Datei liest versteht WAS das Projekt IST — nicht wo Dateien liegen.

## Systemschichten

CLAUDE.md trägt nur was die anderen nicht tragen:

| Datei | Zuständigkeit |
|-------|--------------|
| **CLAUDE.md** | Identität — Was, warum, was gilt immer |
| **PROJECT_MAP.md** | Navigation — Architektur, Einstiegspunkte |
| **Semantic Hooks** | Navigation im Code — @domain, @routing, @boundary als Breadcrumbs |

CLAUDE.md wiederholt NIE was in Map oder Hooks steht.

## Prozess

### 1. Analyse

```bash
cat CLAUDE.md 2>/dev/null; echo "---"; cat PROJECT_MAP.md 2>/dev/null; echo "---"; ls -la; git log --oneline -5 2>/dev/null
```

Lese außerdem: package.json / pyproject.toml / go.mod / Cargo.toml — was vorhanden ist.

Bewerte: Was ist klar? Was fehlt? Was dupliziert Map oder Hooks? Was würde ein Agent ohne diesen Kontext falsch machen?

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

Wenn die Antwort bereits aus dem Code oder der Map folgt → Frage überspringen.

### 3. CLAUDE.md schreiben

**Regeln:**
- Max 40 Zeilen gesamt
- Kein Pfad, keine Dateiliste (→ PROJECT_MAP.md)
- Keine Code-Navigation (→ Semantic Hooks)
- Jeder Constraint konkret, nicht abstrakt

**Pflichtstruktur:**

```markdown
# [Projektname]

[1-2 Sätze: Was ist das. Für wen. Was es tut.]

## Kontext

[Wie wird es genutzt — 2-3 Sätze. Dev-Workflow, Produktion, Agent-Tool.]

## Hard Constraints

- [Was ein Agent NIE tun darf — präzise formuliert]

## Systemschichten

- Navigation & Architektur → PROJECT_MAP.md
- Code-Navigation → Semantic Hooks (@domain, @routing, @boundary)
```

**Optional wenn nötig:**
```markdown
## Nicht-offensichtlich

[Das eine das einen Agent ohne diesen Hinweis falsch laufen lässt]
```

### 4. Partner aktivieren

Führe diese Skills aus wenn sie noch nicht existieren — sie tragen was CLAUDE.md nicht tragen soll:

- **`/map-create`** — Erstellt PROJECT_MAP.md mit Cross-File-Einsichten und Architektur-Startpunkten, damit CLAUDE.md keine Dateistruktur erklären muss.
- **`/semantic-hooks`** — Setzt Breadcrumbs (@domain, @routing, @boundary) direkt im Code, damit Agents Architektur schnell erfassen und durchsuchen können ohne CLAUDE.md dafür zu brauchen.

### 5. Validierung vor dem Schreiben

- [ ] Liest sich in 30 Sekunden komplett
- [ ] Kein Inhalt der in PROJECT_MAP.md oder Hooks gehört
- [ ] Jeder Constraint konkret, nicht abstrakt
- [ ] Wenn PROJECT_MAP.md fehlt → Hinweis am Ende: "Nächster Schritt: /map-create"
- [ ] Wenn Semantic Hooks fehlen → Hinweis: "Nächster Schritt: /semantic-hooks"
