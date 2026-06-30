# Case 2: Weather Care Card

## User Request

> Make a weather care card with deep blue to sky blue gradient, frosted glass feel. Show Shenzhen current temperature (31 deg) and weather condition. If there's a high-temp alert, show it prominently. Below show a health tip like cold index "easy to catch". Add a one-tap call button to easily contact parents. Keep information clear and glanceable.

## Layout Rationale

- **Size**: `2x4` (320x160vp) - needs weather metric + alert + health tip + call action = 3 logical panes benefit from horizontal space.
- **Rhythm**: `metric-context-action`
  - Metric pane (~120vp): location + large temperature + condition
  - Context pane (flex): alert tag + frosted health block
  - Action pane (~56vp): circular call button
- **Semantic roles**:
  - `identity`: location label (Shenzhen)
  - `primaryAnswer`: temperature at 36fp bold - dominant visual anchor
  - `status`: high-temp alert tag with yellow emphasis color
  - `context`: frosted glass block with cold index info
  - `action`: round call button
- **Visual anchor**: the 36fp temperature is the single dominant focal point.
- **Background**: left-to-right gradient from deep navy (#0D2B5E) through medium blue (#1B4B8A) to sky blue (#4A9BD9) - scene-appropriate weather atmosphere.

## Key Decisions

- Frosted glass effect achieved with semi-transparent white background (`#24FFFFFF`) + border (`#3DFFFFFF`) + borderRadius 14 on the health block.
- Alert tag uses yellow foreground on yellow-tinted semi-transparent background for urgency without overpowering.
- Call button is a 48x48 circle (borderRadius:24) for easy thumb-tap target.
- `makePhoneCall` is declared as a host-assumed function; contactId bound from DataModel.
- Weather data fetched via `weather.overview.get` capability with districtName="Shenzhen".
- Health data (cold index) stored in `state` since there's no declared health-data capability - static until host provides updates.

## Width Budget

- Root padding: 12vp each side = 24vp horizontal
- Available: 320 - 24 = 296vp
- Row itemMargin: 10vp x 2 gaps = 20vp
- Metric pane: 120vp fixed
- Action pane: 56vp fixed
- Context pane: 296 - 20 - 120 - 56 = 100vp (layoutWeight:1)
- Alert tag "High Temp Warning" (~4 CJK chars) at 11fp + padding 8 ~ 52vp - fits 100vp
- Health block inner: 100 - 20(padding) - 2(border) = 78vp - fits "Cold Index" + "Easy to catch" labels
- Temperature "31C" at 36fp ~ 54vp - fits 120vp pane

## Improvement Log

- **V1 draft**: Initial design had all elements in a Column (portrait layout) - switched to Row-based 2x4 for better information density and clearer separation of weather data vs. care actions.
- **V2 refinement**: Added frosted glass styling to health block, increased temperature font weight to 700 for stronger visual hierarchy, moved alert tag outside the glass block for independent prominence.

## Validation

- `validate_genui_card.py`: PASSED (3 messages, 12 components)
- `validate_cardspec.py`: PASSED
