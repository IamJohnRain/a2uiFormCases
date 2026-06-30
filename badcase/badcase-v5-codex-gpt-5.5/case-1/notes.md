# Notes

## Layout Rationale

- Scenario/mode: time/reminder + action card, mode 1 new Form card.
- Size: `2x4` (`320 x 160vp`) because the title, next-meeting name, time, reminder text, and focus CTA are all protected content and need safer horizontal room than `2x2`.
- Semantic roles: identity is the top meeting icon plus `会议日程`; primary answer is the next meeting name and time from `data.calendar.items[0]`; context is `提前10分钟提醒`; action is the focus-mode button.
- Main areas: left pane (`174vp`) carries identity and meeting block; right pane (`112vp`) carries reminder and the full-width CTA. Root padding is `12vp`, inner gap is `10vp`, leaving about `296vp` usable width.
- Visual focus: the translucent meeting block and warm time text make the next meeting the anchor; the yellow button is secondary but clearly tappable.
- Information rhythm: header -> next meeting -> reminder -> action, with no scrolling or list expansion.
- Key horizontal relationship: schedule details on the left, interruption-prevention action on the right.
- Must show fully: `会议日程`, next meeting title, time range, reminder minutes, and `开启专注模式`.

## Explicit Improvement

The first internal version placed the CTA in a narrow right rail. I widened the right pane to `112vp` and made the button full-width so `开启专注模式` remains a single protected line with `minFontSize`, while moving secondary reminder text above it.

## Width / Protected-Text Budget

- Root usable width: `320 - 24 padding = 296vp`.
- Main row: `leftPane 174vp + gap 10vp + rightPane 112vp = 296vp`.
- Header row: `174vp` parent, `22vp` icon + `6vp` gap leaves about `146vp` for `会议日程`, enough at `15fp`.
- Meeting block: `174vp - 18vp internal padding = 156vp`; title uses `20fp` with `minFontSize: 14` and no ellipsis; time uses `14fp` with `minFontSize: 12`.
- Right pane: `112vp`; full-width button with `13fp` and `minFontSize: 11` protects `开启专注模式`.

## Data And Host Function Assumptions

- Calendar dynamic data uses only `calendar.events.search` with `range: "today"`, `limit: 1`, and `writeResultTo: "/data/calendar"`.
- Initial DataModel avoids real private calendar content and provides a neutral first item under `/data/calendar/items[0]` so UI bindings align with the `calendar.events.search` output schema before host refresh.
- Host custom function assumption: `enterFocusMode` is declared by the host catalog and accepts `meetingId`, `focusModeId`, and `durationMinutes` from DataModel.

## Validation Log

- `python .agents\skills\harmony-card-generation-v5\scripts\validate_genui_card.py badcase\badcase-v5-codex-gpt-5.5\case-1\card.genui.jsonl` -> passed (`GenUI 卡片校验通过`, 3 messages, 12 components, surfaceId `meeting-focus-card`).
- `python .agents\skills\harmony-card-generation-v5\scripts\validate_cardspec.py badcase\badcase-v5-codex-gpt-5.5\case-1\card.cardspec.json` -> passed (`CardSpec validation passed`).
