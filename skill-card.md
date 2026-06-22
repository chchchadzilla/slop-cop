# Slop Cop

Strict visual QA for shipping creative. Slop Cop audits images, ads, banners, email previews, dashboard mockups, and generated visuals before they go live.

## Catches

- Hallucinated/gibberish text
- Broken hands/anatomy/artifacts/watermarks
- Text overflow outside buttons/cards/squircles
- Overlapping cards, clipped content, broken margins
- Chart/copy contradictions like “Cost per lead trending down” paired with a flat or upward line
- Weak responsive crops and wrong aspect-ratio usage

## Output

Every asset gets a hard verdict:

- `SHIP` — clean enough to deploy
- `FIX` — salvageable with a specific edit
- `KILL` — do not ship

Slop Cop prioritizes semantic truth and layout containment before visual polish.
