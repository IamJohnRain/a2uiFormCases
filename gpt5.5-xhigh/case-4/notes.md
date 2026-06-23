# Case 4 Notes

## Layout Reasoning

- Scene/mode: device/product status glance, mode 1 new desktop card.
- Size: `2x4` (`320 x 160vp`) because the card must show the device name, two protected battery percentages, and two shortcut entries without crowding.
- Semantic roles: `identity` is `Free Clip 2`; `primaryAnswer` is the connected listening state; `metric` is left/right battery (`47%`, `95%`); `context` is the short connection hint; `action` is `每日30首` and `心动歌单`.
- Visual focus: frosted-glass dark gradient shell, translucent left battery pane, two circular L/R earbud markers, and subtle cyan glow text glyphs instead of unsupported remote/product images.
- Information rhythm: left pane gives device identity and battery at a glance; right pane gives short state copy and two bottom shortcuts.
- Key horizontal relation: left and right earbuds are shown side by side with equal width, matching the left/right battery relation.
- Protected information: `Free Clip 2`, `47%`, `95%`, `每日30首`, and `心动歌单` are all one-line texts with no ellipsis/clip/marquee.

## Width Budgets

- Root content width: `320 - 24 padding = 296vp`.
- Main row: `focalPane 126 + itemMargin 12 + detailPane flexible ~= 158vp`, fitting within `296vp`.
- `identityRow`: parent inner width about `126 - 20 = 106vp`; glyph `18` + margin `6` leaves `82vp` for `Free Clip 2`, sufficient at 15fp with min 13fp.
- `earVisual`: parent inner width `106vp`; `leftBud 44 + margin 10 + rightBud 44 = 98vp`, leaving 8vp slack. `47%` and `95%` each get 44vp, sufficient at 16fp.
- `shortcutRow`: detail width about `158vp`; two flexible shortcuts share `158 - 8 = 150vp`, each about 75vp. After horizontal padding `20`, each label gets about 55vp; `每日30首` and `心动歌单` fit at 14fp with min 12fp.

## Interaction And DataModel

- Both shortcut blocks are clickable `Column` components with `onClick` EventHandler arrays.
- Host assumption: `openMusicShortcut` is a host catalog custom function declared by the music/device host. It accepts `shortcutId` from DataModel and opens the corresponding music shortcut.
- The DataModel is static and grouped semantically under `device` and `actions`.

## CardSpec

- `suggestSize` is `2x4`, aligned with root `width: 320` and `height: 160`.
- Static card: no `dataBindings` or `refreshPlan` are declared because no supported capability manifest exists for live earbud battery or music shortcut data in the skill references.

## Explicit Improvement

- First internal version used a narrower right-side action column, which risked squeezing the two shortcut labels.
- Improved by making actions a bottom row inside the flexible detail pane, reducing decorative spacing, and using translucent blocks to preserve the requested frosted-glass feel while keeping protected labels complete.

## Validation Log

- `python .opencode\skills\harmony-card-generation-v5\scripts\validate_genui_card.py gpt5.5-xhigh\case-4\genui.jsonl`: passed. Output: `GenUI 卡片校验通过`, `消息数：3`, `组件数：29`, `surfaceId：free-clip-2-card`.
- `python .opencode\skills\harmony-card-generation-v5\scripts\validate_cardspec.py gpt5.5-xhigh\case-4\cardspec.json`: passed. Output: `CardSpec validation passed`.
