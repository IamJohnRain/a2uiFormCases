# Case 2 Notes

- Size: `2x4`, `300x140`; root padding is `12`, leaving a `276x116` content area.
- Layout budget: columns are `122 + 90 + 48` with two `8` gaps, matching the `276` content width.
- DataModel-first bindings use `{{ $__dataModel... }}` for visible weather, alert, health, and call labels.
- Weather CardSpec uses `ViewWeather` with `writeResultTo: "/data/weather"`; UI paths align with the weather output schema.
- Call action uses declared `clickToCallPhone` with the required `phoneNumber` argument. The initial phone number is empty because no real parent number was provided.
- Colors: root gradient uses official blue family colors; glass panels use component translucent white tokens.
