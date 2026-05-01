# AC Model JSON feature map

Internal notes for LG ThinQ AC/HVAC features observed or expected in ThinQ2
Model JSON data. This document is intentionally limited to field names,
generic values, and implementation notes. Do not add raw diagnostics, account
identifiers, signed URLs, device IDs, SSIDs, tokens, or full snapshots here.

This file is documentation only. It does not change runtime behavior.

## Scope

- Target device family: LG ThinQ AC/HVAC, especially RAC/HVAC devices exposed
  through the existing `custom_components/smartthinq_sensors` integration.
- Primary implementation reference: `CODEX.md`.
- Current integration behavior reviewed in:
  - `custom_components/smartthinq_sensors/wideq/const.py`
  - `custom_components/smartthinq_sensors/wideq/devices/ac.py`
  - `custom_components/smartthinq_sensors/switch.py`
  - `custom_components/smartthinq_sensors/sensor.py`
  - `custom_components/smartthinq_sensors/climate.py`

## Existing AC entities

These fields are already represented by the integration and are listed here so
future work can avoid duplicate or renamed entities.

| LG field | Known values | Probable function | LG manual correspondence | Suggested Home Assistant entity | Confidence | Implementation status |
|---|---|---|---|---|---|---|
| `airState.wMode.airClean` | `@AC_MAIN_AIRCLEAN_OFF_W`, `@AC_MAIN_AIRCLEAN_ON_W` | Air cleaning / ionizer mode | Ionizer / air purification | Existing `switch.<device>_ionizer` via `AirConditionerFeatures.MODE_AIRCLEAN` | High | Implemented |
| `airState.wMode.jet` | `@OFF`, `@COOL_JET`, `@HEAT_JET`, `@DRY_JET_W`, `@HIMALAYAS_COOL`; additional values may exist by model | Jet operation variants | Jet cool, jet heat, jet dry, fast cooling variants | Existing `switch.<device>_jet_mode`; future optional `select.<device>_jet_mode` if multiple variants are confirmed | High for current switch; Medium for future select | Implemented as boolean switch |
| `airState.lightingState.displayControl` | Display on/off values vary by model, including normal and inverted LED mappings | Indoor unit display light | Display / light off | Existing `switch.<device>_display_light` via `AirConditionerFeatures.LIGHTING_DISPLAY` | High | Implemented |
| `airState.reservation.sleepTime` | Numeric minutes within model range | Sleep timer reservation | Sleep / timer | Existing `sensor.<device>_sleep_time` and `set_sleep_time` climate service | High | Implemented |
| `airState.energy.onCurrent` | Numeric watts when available | Instant power draw | Energy monitoring | Existing `sensor.<device>_energy_current` | High | Implemented |
| `airState.humidity.current` | Numeric relative humidity | Indoor humidity | Humidity display where supported | Existing humidity sensor and climate current humidity | High | Implemented |
| `airState.quality.PM1` | Numeric concentration | PM1 air quality | Air quality monitoring | Existing `sensor.<device>_pm1` | High | Implemented |
| `airState.quality.PM2` | Numeric concentration | PM2.5 air quality | Air quality monitoring | Existing `sensor.<device>_pm2_5` | High | Implemented |
| `airState.quality.PM10` | Numeric concentration | PM10 air quality | Air quality monitoring | Existing `sensor.<device>_pm10` | High | Implemented |
| `airState.filterMngStates.useTime` / `airState.filterMngStates.maxTime` | Numeric filter use/max time | Filter remaining life | Filter maintenance | Existing filter remaining life sensor with use/max attributes | High | Implemented |
| `airState.opMode` values `ENERGY_SAVING` / `ENERGY_SAVER` | Enum operation mode | Energy-saving operation mode | Energy saving | Existing climate `eco` preset when supported as operation mode | High | Implemented as climate preset, not independent switch |
| `airState.miscFuncState.silentAWHP` | `@ON` / `@OFF` for supported AWHP models | Outdoor/silent mode for air-to-water heat pump | Silent operation | Existing `switch.<device>_silent_mode` for AWHP | High for AWHP only | Implemented |

## High-priority candidates

These fields have a clear manual correspondence and appear suitable for small
future implementation phases, but they are not implemented by this documentation
change.

| LG field | Known values | Probable function | LG manual correspondence | Suggested Home Assistant entity | Confidence | Implementation status |
|---|---|---|---|---|---|---|
| `airState.miscFuncState.autoDry` plus `support.racMode` containing `@AUTODRY` | `@ON` / `@OFF`, `@STEP2`, `@STEP3`, `@AIAUTODRY` by model | Keeps indoor unit dry after operation to reduce moisture | Auto clean / automatic drying | `switch.<device>_auto_dry` | High when support marker exists | Implemented in Phase 2 with support-marker detection |
| `airState.miscFuncState.Uvnano` plus `support.pacModeExt` containing `@UV_NANO` | `@ON` / `@OFF` | UVnano sterilization / fan cleaning feature | UVnano | `switch.<device>_uvnano` | High when support marker exists; low if using only the state field | Implemented in Phase 2 with support-marker detection |
| `airState.miscFuncState.antiBugs` | Likely `@ON` / `@OFF` or `1` / `0`; support marker not confirmed | Mosquito-repellent mode | Anti mosquito | `switch.<device>_anti_bugs` | Medium/Low until support marker is found | Implemented in Phase 3 as disabled-by-default experimental switch |
| `airState.wMode.lowHeating` | Likely `@ON` / `@OFF` or `1` / `0`; support marker not confirmed | Low heating / minimum heat protection | Low heating | `switch.<device>_low_heating` | Medium/Low until support marker is found | Implemented in Phase 3 as disabled-by-default experimental switch |
| `airState.powerSave.basic` | Boolean or level; requires model validation | Power save / reduced consumption mode | Energy saving | `switch.<device>_power_save` if boolean | High function match; Medium value model | Implemented in Phase 4 as disabled-by-default switch when the model has an ON/OFF mapping |
| `airState.bellSound.control` | Boolean or volume enum; requires model validation | Buzzer / button sound control | Sound / beep setting | `switch.<device>_sound` if on/off; `select.<device>_buzzer_volume` if volume enum | Medium | Documented only |

## Medium-priority candidates

These fields probably map to real features, but they need app or physical
device validation before exposing enabled entities.

| LG field | Known values | Probable function | LG manual correspondence | Suggested Home Assistant entity | Confidence | Implementation status |
|---|---|---|---|---|---|---|
| `airState.wMode.smartCare` | Likely `@ON` / `@OFF` or `1` / `0`; confirm if control or status only | Smart Care / automatic comfort optimization | Smart care where present in app/manual | `switch.<device>_smart_care` if controllable | Medium | Documented only |
| `airState.deepSleep.onOff` | Unknown on/off enum | Deep Sleep / comfort sleep state | Sleep / comfort sleep | `switch.<device>_deep_sleep` | Medium/High | Documented only |
| `airState.deepSleep.adjustTemp` | Unknown enum or numeric range | Temperature adjustment while sleeping | Comfort sleep temperature adjustment | `number.<device>_deep_sleep_adjust_temp` if numeric; otherwise `select` | Medium | Documented only |
| `airState.deepSleep.adjustOperation` | Unknown enum | Operation behavior while sleeping | Comfort sleep operation adjustment | `select.<device>_deep_sleep_operation` | Medium | Documented only |
| `airState.wMode.humanCare` | Unknown enum | Airflow behavior relative to people | Human care / direct or indirect airflow if present | `select.<device>_human_care` | Medium | Documented only |
| `airState.wMode.indirectWind` | Likely `@ON` / `@OFF` or `1` / `0` | Indirect wind | Comfort airflow / indirect airflow | `switch.<device>_indirect_wind` | Medium | Documented only |
| `airState.wDir.leftRight` | Likely `@ON` / `@OFF`, `@100`, or model-specific enum | Horizontal swing fallback where step mode is absent | Left/right swing | Existing climate swing support may already cover related fields; add fallback only if needed | Medium | Documented only |
| `airState.wDir.upDown` | Likely `@ON` / `@OFF`, `@100`, or model-specific enum | Vertical swing fallback where step mode is absent | Up/down swing | Existing climate swing support may already cover related fields; add fallback only if needed | Medium | Documented only |

## Energy and target settings

These fields should be treated carefully because LG manuals can distinguish
simple energy saving from stepped energy control.

| LG field | Known values | Probable function | LG manual correspondence | Suggested Home Assistant entity | Confidence | Implementation status |
|---|---|---|---|---|---|---|
| `activeEnergyControl` | Unknown; may be stepped percentages such as 20%, 40%, 60% depending on model | Active energy control level | Energy control | `select.<device>_energy_control` | Medium | Implemented in Phase 4 as disabled-by-default select when model options are available |
| `powerSave` | Unknown container or state, model dependent | Power save feature group | Energy saving | Depends on subfield; avoid direct entity until schema is confirmed | Medium | Documented only |
| `energyDesiredCtrl` | Unknown command/control group | Energy target control | Energy monitoring / target usage | No direct entity until command semantics are confirmed | Low/Medium | Documented only |
| `airState.energy.desiredDay` | Numeric energy target if present | Daily energy usage target | Energy target | `number.<device>_energy_target_day` | Medium | Documented only |
| `airState.energy.desiredWeek` | Numeric energy target if present | Weekly energy usage target | Energy target | `number.<device>_energy_target_week` | Medium | Documented only |
| `airState.energy.desiredMonth` | Numeric energy target if present | Monthly energy usage target | Energy target | `number.<device>_energy_target_month` | Medium | Documented only |

## Low-priority or unclear fields

These names are too model-specific or unclear for immediate public entities.
They should remain documentation-only until a real device/app/manual validation
shows the exact behavior.

| LG field | Known values | Probable function | LG manual correspondence | Suggested Home Assistant entity | Confidence | Implementation status |
|---|---|---|---|---|---|---|
| `airState.wMode.iceValley` | Unknown enum/on-off | Special cooling or airflow mode | No confirmed match | Do not implement initially | Low | Documented only; blocked pending validation |
| `airState.wMode.flowShower` | Unknown enum/on-off | Special airflow pattern | No confirmed match | Do not implement initially | Low | Documented only; blocked pending validation |
| `airState.wMode.flowForest` | Unknown enum/on-off | Natural/forest airflow pattern | No confirmed match | Do not implement initially | Low | Documented only; blocked pending validation |
| `airState.wMode.flowLongPower` | Unknown enum/on-off | Long or powerful airflow | No confirmed match | Do not implement initially | Low/Medium | Documented only; blocked pending validation |
| `airState.wMode.flowHuman` | Unknown enum/on-off | Person-oriented airflow | Possible human care / comfort airflow | Do not implement initially | Low/Medium | Documented only; blocked pending validation |

## Jet mode enum notes

`airState.wMode.jet` is currently exposed as a boolean switch. The underlying
field is an enum on many models, so a future `select` could preserve the switch
for compatibility while exposing additional variants.

| Value | LG label | Interpretation | Current status |
|---|---|---|---|
| `0` or `@OFF` | `@OFF` | Jet disabled | Existing switch off |
| `1` or `@COOL_JET` | `@COOL_JET` | Jet cooling | Existing switch on when supported |
| `2` or `@HEAT_JET` | `@HEAT_JET` | Jet heating | Existing switch on when supported |
| `3` or `@DRY_JET_W` | `@DRY_JET_W` | Jet dry | Existing switch may treat as on if enum is recognized |
| `4` or `@HIMALAYAS_COOL` | `@HIMALAYAS_COOL` | Fast cooling variant | Existing switch may treat as on if enum is recognized |
| `5` or `@FAN_JET` | `@FAN_JET` | Strong fan mode | Not currently listed in `JetMode` enum |
| `6` or `@AUTO_MODE_JET` | `@AUTO_MODE_JET` | Jet in automatic mode | Not currently listed in `JetMode` enum |

## Phase 2 validation notes

Phase 2 added switch entities for `Auto dry` and `UVnano`. A later comparison of
four AC diagnostics showed that state fields under `airState.*` can be generic
schema entries rather than proof of physical support, so support detection must
prefer `support.*` markers.

- `Auto dry`: use `support.racMode` containing `@AUTODRY` as the support marker.
  In the reviewed diagnostics, all four ACs exposed this marker.
- `UVnano`: use `support.pacModeExt` containing `@UV_NANO` as the support marker.
  In the reviewed diagnostics, only the known UVnano-capable AC exposed this
  marker. The `airState.miscFuncState.Uvnano` field existed in all four schemas,
  so it must not be used by itself to create an enabled entity.

## Phase 3 validation notes

Phase 3 added `Anti bugs` and `Low heating` as experimental switches.

- `Anti bugs`: `airState.miscFuncState.antiBugs` appears as a generic field, but
  no reliable `support.*` marker has been identified yet.
- `Low heating`: `airState.wMode.lowHeating` appears as a generic field, but no
  reliable `support.*` marker has been identified yet.
- Until a marker is found or physical/app testing confirms support, these
  entities should remain disabled by default.

## Next steps

1. Continue observing `Auto dry` across different operation modes, especially
   while the AC is on, after turning it off, and after the next polling cycle.
2. Compare the Home Assistant `Auto dry` state with any LG app behavior if a
   future app version or device model exposes the control.
3. For future features, compare at least one supported and one unsupported
   device when possible, looking first at `support.*` capability fields.
4. Find reliable support markers for `AntiBugs` and `Low Heating` before making
   them enabled by default.

## Implementation guidance for future phases

- Detect support dynamically from `model_info`, `available_features`, and/or
  current device status.
- Do not treat `Value.airState.*` or `device_status` field existence as proof of
  support by itself. These fields may be generic across model JSONs.
- Prefer explicit capability markers under `support.*`; add small helper
  properties in `wideq/devices/ac.py` when a marker is confirmed.
- If a feature has only a generic state field and no confirmed marker, keep the
  entity disabled by default or leave the feature documented as pending.
- Add new AC feature keys to `AirConditionerFeatures` only when implementing
  behavior, not while documenting.
- For on/off features, prefer `ThinQSwitchEntityDescription`.
- Use `SelectEntityDescription` or `NumberEntityDescription` only when the real
  model field is confirmed to be enum-like or numeric.
- Keep existing entities and unique IDs unchanged.
- If control support is uncertain, consider disabling the new entity by default
  or leaving it documented as pending.
