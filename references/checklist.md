# Per-Asset Checklist

Every asset must be scored against every item. Missing data is not a pass — note "unverified" and downgrade confidence.

## 1. Hallucination scan

- [ ] **Text integrity** — All visible text is real words, correctly spelled, in a real script. No gibberish ("EDIT AGGREE", "BANANN OF", random Latin-ish strings). No half-formed letterforms.
- [ ] **Hands/fingers** — Each visible hand has exactly 5 fingers, sane joint count, no fusion or melting.
- [ ] **Faces/eyes** — Symmetric where intended, no extra pupils, no melted features, no uncanny double-mouth.
- [ ] **Limbs** — Correct count; no extra arms/legs; joints bend the right way.
- [ ] **Objects** — No duplicated/floating/half-clipped objects from generation noise.
- [ ] **Physics & lighting** — Shadows consistent with light sources; reflections plausible.
- [ ] **Watermarks / signatures** — None visible.
- [ ] **AI tool tells** — No Midjourney corner mark, no DALL·E rainbow strip, no SDXL noise patterns.

## 2. Legibility at target size

- [ ] Smallest text element readable at actual render dimensions (not the source 4K version — the size it will *ship* at).
- [ ] Sufficient contrast against background.
- [ ] No text crammed against safe-area edges where it gets cropped on mobile.

## 3. Responsive safety

- [ ] Focal subject identifiable.
- [ ] Survives crop to **16:9** (desktop hero, video card).
- [ ] Survives crop to **4:5** (Instagram-style mobile card).
- [ ] Survives crop to **1:1** (avatar, square card).
- [ ] Survives crop to **9:16** (mobile full-bleed / story).
- [ ] If composition is rigid (e.g. text-overlaid hero), document the safe rect.

## 4. Cross-browser / format

- [ ] Format matches need: PNG/WEBP for transparency, JPG for photos with solid bg.
- [ ] Color profile sRGB (P3 only if you know the pipeline supports it).
- [ ] File size under reasonable budget for web (hero <300KB ideal, <600KB acceptable, >1MB needs justification).
- [ ] Dimensions ≥ 2× the largest render size for retina.
- [ ] No EXIF rotation surprises (sideways photos that look right in Finder, wrong in browser).

## 5. Brand fit (when palette/style provided)

- [ ] Color palette overlaps brand colors or is intentionally complementary.
- [ ] Style matches site's other assets (don't mix photoreal + cartoon unless intentional).
- [ ] Tone/mood matches brand voice (serious / playful / aggressive / chill).

## 6. Format / size sanity for slot

- [ ] Aspect ratio matches slot aspect — flag if it'll be letterboxed or stretched.
- [ ] Resolution adequate (no upscaling blur).
- [ ] If used as full-bleed banner, image was actually composed for that aspect — not a square shoved into a wide frame.

## Scoring shortcut

- Any **hard kill trigger** (gibberish text, extra fingers, watermark, major brand clash without recolor path) → `KILL`.
- Any **fixable** failure (wrong format, wrong aspect, minor recolor, crop adjustment) → `FIX` with named remedy.
- All checks pass → `SHIP`.
