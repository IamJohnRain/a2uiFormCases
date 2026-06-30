# Case 1: Meeting Schedule Card

## User Request

> Make a meeting schedule card. Top: meeting icon + "Meeting Schedule" title. Middle: show the next meeting's name and time. Below: small text showing how many minutes in advance to remind. Bottom: button that opens phone focus mode so meetings won't be interrupted.

## Layout Rationale

- **Size**: `2x2` (160x160vp) - single next meeting + action fits naturally in a compact square.
- **Rhythm**: `header-primary-action` - header identity row (18vp), primary meeting block (58-72vp), action button (30-36vp).
- **Semantic roles**:
  - `identity`: calendar icon + "Meeting Schedule" header
  - `primaryAnswer`: meeting title + time (largest visual weight)
  - `context`: reminder text (secondary)
  - `action`: focus mode button
- **Visual anchor**: the time text at 18fp bold is the one visual focal point.
- **Background**: dark indigo gradient to convey focus/work atmosphere.

## Key Decisions

- Used template loop (`children.componentId/path`) on `/data/calendar/items` with `limit:1` to display the next meeting dynamically, avoiding array index expressions.
- Time text uses `maxLines:1` to protect the critical time value from wrapping.
- Meeting title uses `minFontSize:11` to gracefully shrink if the title is long, while staying readable.
- Button uses full width for easy tap target.
- `enterFocusMode` is declared as a host-assumed function that activates device DND/focus.

## Width Budget

- Root padding: 12vp each side = 24vp horizontal
- Available inner width: 160 - 24 = 136vp
- Header row: icon ~14vp + gap 4vp + text fills rest (~118vp) - safe
- Meeting title at 14fp, maxLines:1 with minFontSize:11 - fits ~12 CJK chars at min
- Time "14:00-15:00" at 18fp ~ 90vp - well within 136vp
- Button label (5 CJK chars at default size) ~ 70vp - fits 136vp width

## Improvement Log

- **V1 draft**: Used `$__dataModel.data.calendar.items[0].title` expression with array indexing - invalid in Form expressions.
- **V2 fix**: Replaced with template loop `children: {componentId, path}` pattern. Action targetId moved to a separate DataModel field bound by the host.

## Validation

- `validate_genui_card.py`: PASSED (3 messages, 10 components)
- `validate_cardspec.py`: PASSED
