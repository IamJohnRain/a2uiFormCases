# Case 3 Notes

## Layout Rationale

- Scene: device/product + status glance + action card.
- Size: `2x4` (`320 x 160vp`). The card needs to show the device identity, two protected battery percentages, connection status, and two protected shortcut labels. `2x2` would force the two actions or two battery values into unsafe narrow rows.
- Semantic roles: identity = `Free Clip 2`; status = `已连接`; metric = left/right battery `47%` and `95%`; action = `每日30首` and `心动歌单`; media = no reliable local image, so it is downgraded to abstract frosted-glass orbit and watermark layers.
- Visual focus: a clean light-gray frosted shell with soft white glass aura, subtle product watermark, and one translucent battery panel.
- Information rhythm: top identity/status, middle battery metrics, bottom shortcuts.
- Key horizontal relationship: left and right earbuds are shown as equal-width metric columns in one glass panel.
- Must fully display: `Free Clip 2`, `已连接`, `左耳`, `47%`, `右耳`, `95%`, `每日30首`, `心动歌单`.
- Crowded row budget: root inner width is `296vp`. Battery panel horizontal padding is `16vp`, row gap is `8vp`, leaving `272vp`; each battery column is `135vp`, total `270vp`. Inside each battery header, label and percent share `135vp` with `8vp` gap; `左耳/右耳` at 11fp and `47%/95%` at 14fp fit without ellipsis. Action row has `296vp` with `8vp` gap; two buttons are `142vp` each, total `292vp`, so both CTA labels fit.
- Interaction and DataModel: two buttons use `onClick` EventHandler arrays. Args are bound from `actions.daily` and `actions.heart`.
- CardSpec: static card, `suggestSize` is `2x4`; no declared device battery capability exists in the skill CardSpec capability list, so no `dataBindings` or `refreshPlan` are invented.

## Explicit Improvement

- First internal version risk: using separate narrow right-side action rail would compress `每日30首` and `心动歌单`.
- Improvement: moved actions into a full-width bottom row with two 142vp buttons, and used the center glass panel only for battery metrics to keep protected values safe.

## Host Assumption

- `openHeadphoneRecommendation` is assumed to be a host catalog custom function registered by the app. It receives `playlistId` and `source` to open the requested recommendation entry.

## Media Downgrade

- No local headphone image path was provided. Per skill rules, the card does not invent a remote URL and uses abstract frosted-glass visual elements instead of an `Image` background.

## Validation Log

- `python .opencode\skills\harmony-card-generation-v5\scripts\validate_genui_card.py gpt5.5-xhigh\case-3\genui.jsonl`: passed.
- `python .opencode\skills\harmony-card-generation-v5\scripts\validate_cardspec.py gpt5.5-xhigh\case-3\cardspec.json`: passed.
- Final review: protocol, protected text, static CardSpec alignment, media downgrade, and host action assumption checked.
