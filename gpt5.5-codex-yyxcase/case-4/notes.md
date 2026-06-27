# Case 4 Notes

## Layout Reasoning

- Scene: `time/reminder` plus `action card`; the card answers "what starts next, when should I be reminded, and can I switch focus mode?"
- Size: `2x4` / `320 x 160vp`, because the reminder and focus action need horizontal separation from the event time/title.
- Semantic roles: identity is "即将开始"; primary answer is the next event time/title; context is reminder time and location; action is focus mode toggle.
- Main regions: left focal event pane, middle reminder detail pane, right focus mode action pane. This keeps the card at three primary regions, under the `2x4` limit of four.
- Visual focus: the left translucent event block and large time text. The focus button is visible but smaller than the event.
- Information rhythm: label -> time/title -> reminder/location/progress -> focus state/action.
- Key horizontal relationship: schedule content flows left to right from upcoming event to reminder to focus action.
- Protected information that must be fully visible: "即将开始", event time, event title, "提前10分钟", "专注", "未开启", and CTA "开启". Optional location can compress with ellipsis.
- Row width budget: root width 320 minus horizontal padding 24 leaves 296vp. The three direct panes are 126vp + 100vp + 54vp with two 8vp gaps, total 296vp. The right CTA width is 54vp, enough for the two-character protected label "开启"; the reminder value is in a 100vp column and not sharing a crowded row.
- Interaction/DataModel: `focusButton` uses `onClick` with host function `toggleFocusMode`, with `modeId` and `source` from DataModel. Calendar UI reads `/data/calendar/items` through a `List` template.
- CardSpec: `suggestSize` is `2x4`. The card is dynamic for calendar data via `calendar.events.search`, range `next24Hours`, limit `1`, writing to `/data/calendar`. Refresh runs on install, visible, and every 900 seconds.

## Explicit Improvement

Initial internal draft placed the reminder text and focus button together in the right pane. I moved reminder details into a dedicated middle pane and kept the focus control alone on the right, which protects both the reminder time and CTA from narrow-row compression.

## Validation Result

- `python .agents/skills/harmony-card-generation-v5/scripts/validate_genui_card.py gpt5.5-codex-yyxcase/case-4/card.genui.jsonl`: passed. Messages: 3. Components: 16. Surface: `case-4-schedule-focus`.
- `python .agents/skills/harmony-card-generation-v5/scripts/validate_cardspec.py gpt5.5-codex-yyxcase/case-4/card.cardspec.json`: passed.

## Host Function Assumptions

- `toggleFocusMode` is a host catalog custom function. It accepts `modeId` and `source`, toggles the user's focus mode, and may update `/focus/stateText` and `/focus/buttonLabel` at runtime.
