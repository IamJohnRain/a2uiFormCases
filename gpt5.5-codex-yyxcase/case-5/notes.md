# Case 5 Notes

## Layout Rationale

- Scene and mode: new compact Form card, classified as `device/product` plus `action card`.
- Size: `2x4` / `320 x 160vp`, because the card must show left and right ear battery values and keep two music entrances visible. A `2x2` card would make the protected percentage values and CTA labels compete for one narrow column.
- Semantic roles:
  - `identity`: "耳机快捷", connected status.
  - `primaryAnswer` / `metric`: left and right ear battery percentages.
  - `context`: charging case battery and daily recommendation summary.
  - `action`: open earbud controls, open daily recommendation, open heartbeat playlist.
  - `media`: no real local device image was provided, so the card uses gradient, text glyph, translucent blocks, and Progress rings.
- Main areas: left battery pane, daily recommendation block, heart playlist button. This stays within the `2x4` limit of no more than four main areas.
- Visual focus: the two battery rings are the primary glance target; the right side supports quick music continuation without becoming a page.
- Information rhythm: identity/status first, paired L/R battery values second, charging case context third, then recommendation and playlist entrances.
- Key horizontal relationship: left pane carries device state; right pane carries music actions. Inside the left pane, the L/R batteries are intentionally parallel.
- Must show completely: "耳机快捷", "已连接", "左耳", "右耳", "86%", "78%", "充电盒 64%", "每日推荐", "今日 30 首", and "心动歌单". No protected text uses ellipsis; only the secondary recommendation summary may ellipsize.

## Width Budget

- Root: 320vp width, 12vp horizontal padding each side, 10vp gap between panes, leaving 286vp. `batteryPane` is 132vp, so `musicPane` receives about 144vp.
- `titleRow`: 132vp pane minus 20vp internal padding gives 112vp. The glyph plus gap uses about 34vp, leaving about 78vp for title/status. Both are short protected Chinese labels with `minFontSize`.
- `earRow`: 112vp inner width, two 50vp ear columns plus 8vp gap equals 108vp. Percent strings are short and centered under 38vp rings.
- `recommendationHeader`: right pane about 144vp wide, block padding removes 20vp, leaving about 124vp. "每日推荐" and "通勤" are short protected labels with a 6vp gap and no truncation.
- `playlistButton`: full right pane width, 42vp high. "心动歌单" is protected and fits as a single CTA label.

## Explicit Improvement

First internal version put the two music entrances in one bottom Row. That made "每日推荐" and "心动歌单" compete with protected battery values and reduced CTA safety. The improved version keeps daily recommendation as a full-width tappable block and moves "心动歌单" into a separate full-width Button, improving hierarchy and preserving complete CTA text.

## CardSpec And Data

- `suggestSize`: `2x4`, aligned with the DSL root dimensions.
- Shape: static CardSpec. The skill's declared capabilities only include weather and calendar; no declared earbud battery or music recommendation capability exists, so this case does not invent `dataBindings` or a refresh plan.
- Initial DataModel groups data by semantics: `device`, `recommendation`, `playlist`, and `actions`.

## Host Function Assumptions

The following `onClick` calls are host catalog custom functions assumed to be registered by the card host:

- `openEarbudsControl(deviceId)`: opens the earbud quick operation panel.
- `openDailyRecommendation(stationId)`: opens the daily recommendation entrance.
- `openHeartbeatPlaylist(playlistId)`: opens the heart playlist entrance.

All business identifiers are bound from DataModel rather than hard-coded in the event handlers.

## Validation Result

- Passed: `validate_genui_card.py` reported "GenUI 卡片校验通过" with 3 messages, 25 components, surfaceId `earbuds-quick-card`.
- Passed: `validate_cardspec.py` reported "CardSpec validation passed".
- Final review-validation checklist: passed. The card uses only Form-supported components, one `updateComponents`, matching `2x4` sizing, real `onClick` handler arrays, no network media, no invented CardSpec capability, and no ellipsis on protected content.
