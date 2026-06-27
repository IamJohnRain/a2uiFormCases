# Case 6 Notes

## Layout rationale

- Scene and mode: new `status glance` plus `device/product` card, mode 1.
- Size: `2x4` / `320 x 160vp`, because the card must show three battery percentages, memory usage, and one action. A `2x2` card would crowd protected percentages and CTA text.
- Semantic roles: identity is `设备状态`; primary metrics are phone, earbuds, and case battery rings; context is memory used/free/advice; action is `一键清理`.
- Main areas: left device battery pane, right memory pane, memory progress block, bottom clean button. This stays within the `2x4` limit of four main areas.
- Visual focus: three compact ring progress indicators on the left; the memory percentage is secondary but prominent in the right pane.
- Information rhythm: title/status first, battery rings next, memory percentage and linear progress next, action last.
- Key horizontal relationship: left = connected device power; right = system memory pressure and cleanup action.
- Must display fully: `设备状态`, `已连接`, `82%`, `64%`, `48%`, `内存使用`, `68%`, `已用 5.4GB`, `可用 2.6GB`, and `一键清理`.

## Width budget

- Root internal width: `320 - 24 padding = 296vp`; root row gap is `10vp`; panes are `136vp + 150vp = 286vp`, total `296vp`.
- Device header row: `136vp`; title about `70vp`, badge about `50vp`, gap `4vp`, leaving about `12vp` safety.
- Battery row: `136vp`; three ring cells are `42vp * 3 = 126vp`, gaps are `5vp * 2 = 10vp`, total `136vp`.
- Memory pane internal width: `150 - 20 padding = 130vp`.
- Memory top row: `130vp`; label about `62vp`, percent about `46vp`, gap `4vp`, leaving about `18vp` safety.
- Memory meta row: `130vp`; two short labels about `54vp` each plus `6vp` gap, leaving about `16vp` safety.
- CTA: full internal width `130vp`, enough for protected label `一键清理`.

## Explicit improvement

The first internal version tried to place three battery rings, memory percentage, and the CTA in one broad row. I changed it to a left battery pane plus right memory/action pane so protected percentages and the CTA have fixed readable space, and the card keeps a single clear visual hierarchy.

## Validation result

- GenUI JSONL: passed with `python .agents/skills/harmony-card-generation-v5/scripts/validate_genui_card.py gpt5.5-codex-yyxcase/case-6/card.genui.jsonl`.
- CardSpec JSON: passed with `python .agents/skills/harmony-card-generation-v5/scripts/validate_cardspec.py gpt5.5-codex-yyxcase/case-6/card.cardspec.json`.
- Final review: passed. The card uses only Form-supported components, keeps `suggestSize` aligned to `2x4`, avoids network media, keeps the CTA clickable with a real `onClick` handler, and does not truncate protected percentages or action text.

## Host function assumptions

- `clearDeviceMemory` is assumed to be a host catalog custom function for the one-tap cleanup action.
- It receives `scope` and `requestId` from DataModel.
- No device-status CardSpec data capability is declared by the skill references. This card is therefore static demo data until the host adds a device status capability manifest and maps its output under `/data`.
