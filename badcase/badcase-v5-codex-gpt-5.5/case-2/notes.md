# Weather Care Card Notes

## Layout Rationale

- Scene and mode: status glance plus one action, new card mode.
- Size: 2x4, because the card must show current Shenzhen weather, high-temperature care context, health advice, and a one-click parent call action without squeezing protected text.
- Semantic roles: identity is Shenzhen weather; primary answer is 31 degrees and current condition; status/context is high-temperature warning; health context is cold index; action is calling parents.
- Visual focus: large temperature on the left over a deep-blue to sky-blue gradient; two translucent blocks create the requested frosted-glass feel.
- Information rhythm: left pane for current weather, middle pane for warning and health reminder, right pane for the call button.
- Key horizontal relationship: weather status and family contact action sit side by side, matching "see important info at a glance, contact parents quickly".
- Protected text: `31°`, `晴热`, `高温预警`, `注意防暑补水`, `易发｜少吹空调`, `拨打`, and `父母` must display completely.

## Explicit Improvement

Initial internal layout placed the warning label and call button in one row, which risked crowding `高温预警` and `拨打`. The final layout separates the CTA into a 44vp right column and gives warning/health text a 132vp frosted context pane, improving protected-text safety.

## Width And Protected-Text Budget

- Root: 320vp width, 12vp horizontal padding each side, 8vp gaps between 3 panes, leaving 280vp for panes.
- Pane allocation: weather 104vp, care 132vp, call 44vp.
- Weather header row: 104vp total, 20vp glyph plus 4vp gap leaves about 80vp for `深圳`, safe at 13fp.
- Care rows: each frosted block is 132vp with 8vp horizontal padding, leaving about 116vp for `注意防暑补水` and `易发｜少吹空调`, safe at 14fp with `minFontSize: 12`.
- CTA column: 44vp button with two-character `拨打`, safe at 13fp with `minFontSize: 11`; contact label `父母` is centered at 44vp.

## Data And Host Function Assumptions

- Weather is dynamic through declared capability `weather.overview.get`, writing to `/data/weather`.
- Declared weather output does not include high-temperature warning or cold index. These are represented as static user-provided caring context under `/care`.
- One-click call uses host custom function assumption `makePhoneCall(phoneNumber, contactName)`. Both args are bound from DataModel at `/family/phoneNumber` and `/family/contactName`.
- `phoneNumber` uses a placeholder app-owned route value `tel:parent-home`, not a real private phone number.

## Validation Log

- Passed: `python .agents\skills\harmony-card-generation-v5\scripts\validate_genui_card.py badcase\badcase-v5-codex-gpt-5.5\case-2\card.genui.jsonl`
  - GenUI card validation passed; messages: 3; components: 17; surfaceId: `weather-care-shenzhen`.
- Passed: `python .agents\skills\harmony-card-generation-v5\scripts\validate_cardspec.py badcase\badcase-v5-codex-gpt-5.5\case-2\card.cardspec.json`
  - CardSpec validation passed.
