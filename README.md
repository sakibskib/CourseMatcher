# UCO Course Articulation Matcher

A single-file static web app. No backend, no API keys, no database. Everything — the full 3,337-course UCO catalog, keyword index, and semantic embedding model — runs in the visitor's browser.

## How it works

1. **First visit:** the app downloads a small quantized embedding model (`bge-small-en-v1.5`, ~34 MB, via Transformers.js) and embeds every catalog course. This takes 1–5 minutes depending on the machine (much faster with WebGPU, e.g. Chrome/Edge). The result is cached in the browser's IndexedDB, so it only happens once per device.
2. **Searching:** the transfer course's title + description is embedded and compared to all 3,337 courses by cosine similarity (semantic channel), and simultaneously scored with BM25 keyword matching where titles are double-weighted (keyword channel). Final rank = 65% semantic + 35% keyword.
3. **Checks:** if you enter credit hours or a course number, each result is flagged for hours mismatch and course-level mismatch (1xxx vs 3xxx etc.). These are advisory flags, not score penalties.
4. **Confidence badge** (High / Moderate / Low) is driven mainly by the raw cosine similarity, which is more comparable across searches than the normalized rank score.

## Updating the catalog

Re-run your extractor to produce a new JSONL, convert it to a minified JSON array with fields `code, title, description, credit_hours, url`, and replace the contents of the `<script id="course-data">` block in `index.html` (escape `</` as `<\/`). The cache key includes the course count, so the browser re-indexes automatically when the catalog changes size. If you want to force a rebuild after an edit that keeps the same count, bump `DB_KEY` in the script (`uco-emb-v1-` → `v2`).

## Tuning knobs (in the script)

- `W_SEM` / `W_KW` — hybrid blend weights (default 0.65 / 0.35).
- Confidence thresholds — `conf > 0.72` (High) and `> 0.62` (Moderate); adjust after you've eyeballed real transfer courses.
- `MODEL` — swap to `Xenova/gte-small` or back to `Xenova/all-MiniLM-L6-v2` if you prefer; same dimension (384), no other changes needed. Bump `DB_KEY` when you change models.
