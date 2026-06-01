---
name: slop-cop
description: Judges visual design assets and AI-generated images before they ship. Use when the user wants to compare design options, choose between asset variants for a hero/banner/icon slot, vet AI-generated images for hallucinated text or artifacts (extra fingers, gibberish, melted hands, watermarks), audit a landing page's visual choices, or get a second opinion on visual design before deploying. Triggers on phrases like "which is best as a hero", "is this image safe to ship", "does this fit the brand", "pick between these assets", "audit this banner", "check this for AI slop", or general "is this slop?" requests.
---

# slop-cop

A visual-design referee. Given one or more image assets plus a decision context, produce strict per-asset verdicts (`SHIP`, `FIX`, or `KILL`) and, when multiple candidates compete for one slot, a ranked recommendation with placement reasoning.

The goal: stop hallucinated text, melted hands, off-brand vibes, and obvious AI artifacts from reaching production.

## When to invoke

- User has 1–N images and a decision to make ("which works best for hero?", "is this safe to ship?", "does this fit my brand?").
- User wants a second opinion on a visual choice before deploy.
- User asks to audit a landing page or compare AI-generated variants.
- User explicitly says "slop check" / "is this AI slop?"

## Inputs the skill needs

Before analysis, confirm or infer:

1. **Image paths** — 1 or more local file paths or URLs.
2. **Decision context** — what slot/role is this for? Examples: `"hero banner at 1200x600"`, `"square avatar 1024x1024"`, `"mobile card at 4:5"`, `"is this safe to ship anywhere?"`.
3. **Target render size / aspect ratio** — if relevant (e.g. hero rendered at 600x600 with rounded corners and a 4px border).
4. **Brand palette / style** — hex colors and a one-line style descriptor when available (e.g. `"navy #0f3a66 / orange #f3812a, cartoon illustration"`).
5. **Mode** — single-asset audit (`SHIP/FIX/KILL`) or comparative pick (rank + recommend one).

If the user does not provide brand context, ask once. If they decline, proceed without brand-fit scoring and note it in the verdict.

## Workflow

### 1. Run the vision pass

For each image, call the OpenClaw `image` tool with the strict checklist prompt in [references/vision-prompt-template.md](references/vision-prompt-template.md). Pass one image per call when possible — keeps the model focused. Use `images` (multi) only for explicit side-by-side comparison once each has been individually vetted.

The prompt template forces the vision model to enumerate findings against a fixed checklist instead of writing vibes-based prose.

### 2. Score against the full checklist

The mandatory checklist lives in [references/checklist.md](references/checklist.md). Every asset must be scored on:

- **Hallucination scan** — gibberish text, extra/melted fingers, broken anatomy, duplicated objects, watermarks, AI signatures, lighting that contradicts itself.
- **Legibility at target size** — can any text on the asset be read at its actual render size?
- **Responsive safety** — will the focal subject survive cropping to 16:9, 4:5, 1:1, and 9:16? Identify the focal point in pixel/percent terms.
- **Cross-browser / format** — transparency needs (PNG/WEBP vs JPG), color profile concerns (sRGB vs P3), iOS Safari quirks.
- **Brand fit** — if palette/style provided, check coherence; flag major mismatches.
- **Format / size sanity** — actual dimensions, file size for web, aspect-ratio fit for the target slot.

### 3. Assign a verdict per asset

Use exactly one verdict word per asset, plus a one-sentence reason. No hedging, no "looks okay but...".

| Verdict | Meaning |
|---------|---------|
| `SHIP`  | Clean. Deploy as-is. |
| `FIX`   | Salvageable with a specific edit (crop, recolor, regenerate text region, swap to different aspect). State the fix. |
| `KILL`  | Do not use. Hallucination, off-brand, broken anatomy, or wrong-tool-for-the-job. |

**Hard kill triggers** (any one of these = automatic `KILL`):

- Visible hallucinated/gibberish text on a graphic shipping to prod.
- Extra/missing/melted fingers on a human or human-adjacent character.
- Visible watermark or AI-tool signature.
- Major brand-palette violation (when palette provided) that can't be fixed by recolor.

See [references/anti-patterns.md](references/anti-patterns.md) for the full kill list and CSS-level gotchas that come up on real sites.

### 4. Comparative mode (multiple candidates, one slot)

When the user is choosing between assets for a single slot:

1. Verdict each candidate individually first.
2. Drop all `KILL` verdicts from the running.
3. Rank remaining `SHIP` and `FIX` candidates by fit-to-context (brand match > focal-point survival > legibility > polish).
4. Recommend one. Name the file path, the slot, and one-sentence placement reasoning.
5. If every candidate is `KILL` or `FIX`, recommend regeneration with a brief brief.

### 5. Output format

Return a structured response:

```
## Verdicts
- <filename> — VERDICT — one-sentence reason
- <filename> — VERDICT — one-sentence reason
...

## Anti-patterns flagged
- (optional) bullet list of CSS/HTML/format gotchas detected from context

## Recommendation
<For comparative mode: which file goes in which slot, why, and any FIX steps needed before deploy.>

## Deploy notes
<Concrete file paths, target dimensions, format conversions, and any CSS/HTML lines that should change. Do NOT execute deploys — describe them.>
```

Keep it tight. No filler, no "great question."

## Failure modes

- **Vision tool unavailable / errors out** — Document the failure, then make a best-effort judgment from filename, file metadata (`identify` / `file` / `exiftool` if available), and decision context. Mark the verdict as `BEST-EFFORT` in parens and flag that a manual eyeball is required before ship.
- **No brand context provided** — Proceed; note "no brand check performed."
- **Asset is a wordmark/logo** — Skip hallucination scan for stylized typography (intentional design ≠ gibberish), but still check legibility, format, and brand-palette match.

## References

- [references/vision-prompt-template.md](references/vision-prompt-template.md) — exact prompt to send to the vision tool.
- [references/checklist.md](references/checklist.md) — the full per-asset checklist.
- [references/anti-patterns.md](references/anti-patterns.md) — concrete bugs and CSS/HTML pitfalls to flag.
- [references/promo-copy.md](references/promo-copy.md) — optional marketing copy for this skill (not loaded during normal use).
