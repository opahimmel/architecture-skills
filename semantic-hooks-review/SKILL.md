---
name: semantic-hooks-review
description: Audits and updates existing semantic hooks (@domain, @owns, @routing, @boundary, @extend, @invariant) for consistency, completeness, cross-references, and staleness. Use when hooks are already in place and need a health check, after a refactor, or when running an architecture review. Builds on the semantic-hooks skill.
---

# Semantic Hooks Review

Builds on [semantic-hooks](../semantic-hooks/SKILL.md). Assumes hooks are already present.

## Quick start

```bash
grep -rn "@domain\|@owns\|@depends\|@routing\|@boundary\|@extend\|@invariant\|@side-effects\|@critical-path\|@context" \
  --include="*.ts" --include="*.tsx" --include="*.js" --include="*.jsx" .
```

## Workflow

1. **Scan** — grep for all hook tags, build file+line inventory
2. **Analyse** — run the four checks (see [CHECKS.md](CHECKS.md))
3. **Report** — print structured findings grouped by check
4. **Fix** — apply edits directly, log every change made

## Fix rules

| Issue type | Action |
|------------|--------|
| kebab-case violations | Auto-fix |
| Duplicate tags in same header | Auto-fix (keep first) |
| Inline hook missing symbol name | Auto-fix — insert symbol name from the line below |
| Missing file-header | Ask — needs role judgment |
| Orphaned / dangling cross-refs | Ask — may be intentional |
| Ownership conflict | Ask — architecture decision |
| Stale domain names | Ask — may be mid-refactor |
| `@invariant` changes | Always show diff, always ask |

## Report format

```
## Semantic Hooks Review — <date>

### 1 Konsistenz     (N issues)
- src/foo.ts:12  @domain:CamelCase → @domain:camel-case
- src/input.ts:141  @routing:pointer-down missing symbol name → @routing:pointer-down pointerDown

### 2 Vollständigkeit (N missing)
- src/bar.ts  no file-header — role: central event dispatcher (>5 switch cases)

### 3 Kreuzreferenzen (N conflicts)
- src/payments.ts  @owns:billing — no file has @depends:billing

### 4 Veraltete Hooks (N stale)
- src/auth.ts:8  @owns:legacy-auth — label not found anywhere in codebase

### Actions taken
- Fixed N / M issues directly
- N items need manual review (listed above)
```
