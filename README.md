# tz

[![CI](https://github.com/alya-lang/tz/actions/workflows/ci.yml/badge.svg)](https://github.com/alya-lang/tz/actions/workflows/ci.yml)
[![License](https://img.shields.io/github/license/alya-lang/tz?color=blue&label=License)](LICENSE)
[![Alya](https://img.shields.io/badge/dynamic/toml?url=https%3A%2F%2Fraw.githubusercontent.com%2Falya-lang%2Ftz%2Fmain%2Falya.toml&query=%24.package.alya-version&label=Alya&color=orange&prefix=%3E%3D)](https://github.com/alya-lang/alya)
[![Package Version](https://img.shields.io/badge/dynamic/toml?url=https%3A%2F%2Fraw.githubusercontent.com%2Falya-lang%2Ftz%2Fmain%2Falya.toml&query=%24.package.version&label=Version&color=brightgreen)](alya.toml)

IANA timezone database and zone-aware datetime conversion for Alya

---

## 🌟 Features

- 🌍 **97 Bundled IANA Zones**: Fixed-offset zones (exact, including half/quarter-hour offsets) plus recurring current-era DST rules (US, EU, Australia, New Zealand, Egypt, Chile, Fiji, Paraguay, Lebanon)
- 🪟 **Windows Key Resolution**: Built-in mirrored Windows→IANA table, so OS-native names like `"Turkey Standard Time"` work directly — no `sysinfo` dependency
- ⏰ **Zone-Aware Conversion**: `epoch_to_zoned` / `zoned_to_epoch` / `zoned_now` roundtrips over `std/time`, with gap/fold documented behavior
- 📏 **Offset & Abbreviation Lookup**: `tz_offset_min` / `tz_offset_str` / `tz_abbrev` / `tz_is_dst` evaluated at any UTC epoch (transition-second precision)
- 🖨️ **Wall-Time Formatting**: `format_zoned` renders `YYYY-MM-DD HH:MM:SS+HH:MM`; `format_zoned_custom` takes token patterns
- 🔍 **Transitions & Discovery**: `tz_dst_range` / `next_transition` expose DST boundaries; `tz_by_region` / `tz_by_offset` / `tzdata_version` help explore the bundle
- ⌨️ **Wall-Time Parsing**: `parse_zoned` converts zone-local strings to epochs (complement of UTC `parse_iso8601`)
- 🧩 **Modular Architecture**: Clean public facade (`src/lib.alya`), rule model (`src/types.alya`), conversion engine (`src/engine.alya`), embedded data (`src/tzdata/zones.alya`, `src/tzdata/windows.alya`)
- 🔒 **Public/Private Visibility (`pub`)**: Engine internals (`engine_*`, `tzdata_*`) stay out of the facade; consumers use the documented API
- 🛡️ **Structured Errors**: Unknown zones and invalid dates throw `TzError`, caught with standard `try`/`catch`
- 🧪 **Test & Benchmark Suite**: 66 assertions (`std/test`) covering transitions, roundtrips, and error paths, plus micro-benchmarks

---

## 📁 Project Architecture

```
tz/
├── .alyalint               # Linter configuration (rules, exclusions, severity overrides)
├── .editorconfig           # Uniform formatting rules across IDEs and editors
├── .gitignore              # Ecosystem standard ignore filters
├── .vscode/                # VS Code workspace settings, DAP launch configurations & tasks
├── alya.toml               # Package manifest with dependencies and optional [build]
├── src/
│   ├── lib.alya            # Public API facade (tz_offset_min, epoch_to_zoned, zoned_to_epoch, ...)
│   ├── types.alya          # TzRule / TzError models, fixed_rule / dst_rule factories
│   ├── engine.alya         # Offset engine, transition math, conversions (engine_* internals)
│   └── tzdata/             # Embedded zone tables (mirrors V's time.tzdata role)
│       ├── zones.alya      # 97 bundled rules + alias canonicalization
│       └── windows.alya    # Mirrored Windows→IANA table (canonical source: sysinfo)
├── examples/
│   └── demo.alya           # Runnable walkthrough of all package capabilities
├── tests/
│   └── test_basic.alya     # Automated test suite (66 assertions)
└── benches/
    └── bench_basic.alya    # Micro-benchmarks measuring conversion throughput
```

> [!NOTE]
> **Visibility & Modularity:** Symbols annotated with `pub` (`pub function`, `pub struct`) are exported to external consumers and re-exporting modules. Symbols without `pub` remain strictly internal to their declaring module, preventing symbol collisions and implementation leakage.

---

## 📦 Installation

Add `tz` to the `[dependencies]` section in your `alya.toml`:

```toml
[dependencies]
tz = { git = "https://github.com/alya-lang/tz", branch = "main" }
```

Or install it directly using the Alya package CLI:

```bash
alya add tz --git https://github.com/alya-lang/tz --branch main
alya install
```

---

## 🚀 Quick Start

```alya
import "tz" as pkg

function main()
    # Wall time in Shanghai for a known UTC instant (V README example)
    let z = pkg::epoch_to_zoned(1704067200, "Asia/Shanghai")
    say z["hour"] # 8

    # DST-aware offset: EST in January, EDT in July
    say pkg::tz_offset_min("America/New_York", 1704067200) # -300
    say pkg::tz_offset_min("America/New_York", 1721068245) # -240

    # Back to epoch
    say pkg::zoned_to_epoch(z, "Asia/Shanghai") # 1704067200
end

main()
```

---

## 📖 API Reference

| Symbol | Visibility | Description |
|---|---|---|
| `tz_offset_min(name, epoch)` | `pub function` | UTC offset in minutes east of UTC active at the epoch. Accepts IANA names, aliases, and Windows keys. Throws `TzError` for unknown zones. |
| `tz_offset_str(name, epoch)` | `pub function` | Offset as `+HH:MM` (`"+03:00"`, `"-05:00"`, `"+00:00"`). Throws `TzError` for unknown zones. |
| `tz_abbrev(name, epoch)` | `pub function` | Active zone abbreviation (`"EST"`/`"EDT"`). Throws `TzError` for unknown zones. |
| `tz_is_dst(name, epoch)` | `pub function` | 1 when daylight saving applies, 0 otherwise. Throws `TzError` for unknown zones. |
| `epoch_to_zoned(epoch, name)` | `pub function` | UTC epoch to zoned calendar map (`year`..`weekday` plus `offset_min`, `abbr`, `iana`). Throws `TzError` for unknown zones. |
| `zoned_now(name)` | `pub function` | Zoned calendar map for the current time. Throws `TzError` for unknown zones. |
| `format_zoned(d)` | `pub function` | Zoned map to `YYYY-MM-DD HH:MM:SS+HH:MM`. |
| `tz_rule(name)` | `pub function` | Zone rule lookup (IANA, alias, or Windows key), or null when the zone is not bundled. |
| `tz_windows_iana(name)` | `pub function` | Windows key to IANA via the mirrored table, or "" when unmappable. |
| `tz_names()` | `pub function` | Array of all 97 bundled IANA zone names. |
| `tzdata_version()` | `pub function` | Opaque revision string for the bundled rule set. |
| `tz_by_region(prefix)` | `pub function` | Zone names under a region prefix (`"Europe/"`). |
| `tz_by_offset(offset_min)` | `pub function` | Zone names using an offset (standard or daylight). |
| `zoned_to_epoch(d, name)` | `pub function` | Zoned wall-time map to UTC epoch (gaps resolve forward, folds to standard-time occurrence). Throws `TzError` for unknown zones or invalid dates. |
| `parse_zoned(s, name)` | `pub function` | Wall-time string (`YYYY-MM-DD[THH:MM:SS]`) in the zone to UTC epoch. Throws `TzError` when malformed or unknown zone. |
| `tz_dst_range(name, year)` | `pub function` | Map with `has_dst` and UTC `start_utc`/`end_utc` for a zone year. Throws `TzError` for unknown zones. |
| `next_transition(name, epoch)` | `pub function` | Next DST transition strictly after the epoch, or null for fixed zones. Throws `TzError` for unknown zones. |
| `format_zoned_custom(d, fmt)` | `pub function` | Zoned map with token pattern (`YYYY MM DD HH mm ss Z A`). |
| `format_rfc3339_zoned(d)` | `pub function` | Zoned map to `YYYY-MM-DDTHH:MM:SS+HH:MM`. |
| `zoned_add_hours(epoch, name, h)` | `pub function` | Absolute hour shift, then re-zoned. |
| `zoned_add_days(epoch, name, n)` | `pub function` | Calendar day shift keeping the wall clock (DST-safe). |
| `tz_is_valid(name)` | `pub function` | 1 for bundled zones, aliases, and Windows keys, 0 otherwise. |
| `tz_canonical(name)` | `pub function` | Alias canonicalization (`"UTC"` → `"Etc/UTC"`). |
| `TzRule` | `pub struct` | Zone rule model (offsets, recurring DST schedule, UTC/local transition flags). |
| `TzRule.has_dst()` | `pub method` | 1 for DST zones, 0 for fixed zones. |
| `TzError` | `pub struct` | Thrown error (`message`). |
| `fixed_rule(name, abbr, offset_min)` | `pub function` | Factory for fixed-offset zones. |
| `dst_rule(name, std_abbr, dst_abbr, std, dst, sm, sw, sd, sh, su, em, ew, ed, eh, eu)` | `pub function` | Factory for DST zones (nth-weekday rules, UTC/local flags). |

> [!TIP]
> **Scope:** Rules are recurring current-era schedules (US post-2007, EU post-1996). Fixed zones are exact going forward; pre-changeover historical dates are out of scope for v0.1.0. Public symbols are documented with `##` docstrings for `alya doc`.

---

## 🗺️ Zone Coverage (97 bundled)

| Zone | Std (min) | Rule |
|---|---|---|
| `Etc/UTC` | 0 | fixed |
| `Europe/Istanbul` | 180 | fixed |
| `Europe/Moscow` | 180 | fixed |
| `Asia/Dubai` | 240 | fixed |
| `Asia/Karachi` | 300 | fixed |
| `Asia/Kolkata` | 330 | fixed |
| `Asia/Dhaka` | 360 | fixed |
| `Asia/Amman` | 180 | fixed |
| `Asia/Singapore` | 480 | fixed |
| `Asia/Shanghai` | 480 | fixed |
| `Asia/Hong_Kong` | 480 | fixed |
| `Asia/Taipei` | 480 | fixed |
| `Asia/Seoul` | 540 | fixed |
| `Asia/Tokyo` | 540 | fixed |
| `Australia/Perth` | 480 | fixed |
| `Australia/Darwin` | 570 | fixed |
| `Pacific/Honolulu` | -600 | fixed |
| `America/Phoenix` | -420 | fixed |
| `America/Mexico_City` | -360 | fixed |
| `America/Bogota` | -300 | fixed |
| `America/Lima` | -300 | fixed |
| `America/Sao_Paulo` | -180 | fixed |
| `America/Argentina/Buenos_Aires` | -180 | fixed |
| `Africa/Lagos` | 60 | fixed |
| `Africa/Johannesburg` | 120 | fixed |
| `Africa/Nairobi` | 180 | fixed |
| `America/New_York` | -300 | DST |
| `America/Chicago` | -360 | DST |
| `America/Denver` | -420 | DST |
| `America/Los_Angeles` | -480 | DST |
| `America/Anchorage` | -540 | DST |
| `America/Toronto` | -300 | DST |
| `America/Vancouver` | -480 | DST |
| `America/Halifax` | -240 | DST |
| `Europe/London` | 0 | DST |
| `Europe/Lisbon` | 0 | DST |
| `Atlantic/Azores` | -60 | DST |
| `Europe/Berlin` | 60 | DST |
| `Europe/Paris` | 60 | DST |
| `Europe/Rome` | 60 | DST |
| `Europe/Madrid` | 60 | DST |
| `Europe/Amsterdam` | 60 | DST |
| `Europe/Zurich` | 60 | DST |
| `Europe/Vienna` | 60 | DST |
| `Europe/Prague` | 60 | DST |
| `Europe/Warsaw` | 60 | DST |
| `Europe/Athens` | 120 | DST |
| `Europe/Helsinki` | 120 | DST |
| `Europe/Chisinau` | 120 | DST |
| `Australia/Sydney` | 600 | DST |
| `Australia/Melbourne` | 600 | DST |
| `Pacific/Auckland` | 720 | DST |
| `Africa/Cairo` | 120 | DST |
| `America/Havana` | -300 | DST |
| `Atlantic/Bermuda` | -240 | DST |
| `America/St_Johns` | -210 | DST |
| `Australia/Hobart` | 600 | DST |
| `Australia/Adelaide` | 570 | DST |
| `Australia/Lord_Howe` | 630 | DST |
| `Pacific/Chatham` | 765 | DST |
| `Pacific/Fiji` | 720 | DST |
| `America/Santiago` | -240 | DST |
| `Pacific/Easter` | -360 | DST |
| `America/Asuncion` | -240 | DST |
| `Asia/Beirut` | 120 | DST |
| `Europe/Kyiv` | 120 | DST |
| `Asia/Manila` | 480 | fixed |
| `Asia/Jakarta` | 420 | fixed |
| `Asia/Bangkok` | 420 | fixed |
| `Pacific/Guam` | 600 | fixed |
| `Pacific/Port_Moresby` | 600 | fixed |
| `Australia/Brisbane` | 600 | fixed |
| `Asia/Kathmandu` | 345 | fixed |
| `Asia/Baghdad` | 180 | fixed |
| `Asia/Tbilisi` | 240 | fixed |
| `Asia/Yerevan` | 240 | fixed |
| `America/Noronha` | -120 | fixed |
| `Pacific/Marquesas` | -570 | fixed |
| `Australia/Eucla` | 525 | fixed |
| `America/Regina` | -360 | fixed |
| `America/Cancun` | -300 | fixed |
| `America/Cuiaba` | -240 | fixed |
| `America/La_Paz` | -240 | fixed |
| `America/Cayenne` | -180 | fixed |
| `America/Araguaina` | -180 | fixed |
| `Atlantic/Reykjavik` | 0 | fixed |
| `Atlantic/Cape_Verde` | -60 | fixed |
| `Pacific/Apia` | 780 | fixed |
| `Pacific/Tongatapu` | 780 | fixed |
| `Pacific/Kiritimati` | 840 | fixed |
| `Pacific/Bougainville` | 660 | fixed |
| `America/Tijuana` | -480 | DST |
| `America/Grand_Turk` | -300 | DST |
| `America/Port-au-Prince` | -300 | DST |
| `America/Miquelon` | -180 | DST |
| `America/Nuuk` | -180 | DST |
| `Pacific/Norfolk` | 660 | DST |

Not covered (irregular rules): `Asia/Jerusalem`, `Asia/Tehran`, `Africa/Casablanca`, `Asia/Gaza`, `Asia/Damascus`. `tz_rule` returns null for these; every conversion throws `TzError`.

---

## 🧪 Running Tests & Benchmarks

Run the automated test suite using `alya test`:

```bash
alya test
```

Generate static API documentation:

```bash
alya doc . -o docs --markdown
```

Run the benchmark suite:

```bash
alya run benches/bench_basic.alya
```

Run the example demo:

```bash
alya run examples/demo.alya
```

Check code formatting:

```bash
alya fmt . --check
```

Run static code linter:

```bash
alya lint . --check
```

---

### 💻 Developer Tooling & VS Code Integration

This package comes preconfigured with recommended workspace settings and tasks for **Visual Studio Code**:
- **LSP & Formatting**: Auto-formatting on save and real-time Language Server diagnostics via `alya-lang.vscode-alya`.
- **DAP Debugging**: Launch configurations in `.vscode/launch.json` ready for interactive step-debugging via `F5`.
- **Predefined Tasks**: Press `Ctrl+Shift+B` or run tasks (`Test`, `Lint`, `Format`, `Build Docs`) directly from the Command Palette.

---

## 🤝 Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository and clone it locally
2. Install dependencies:
   ```bash
   alya install
   ```
3. Create your feature branch (`git checkout -b feature/my-feature`)
4. Verify tests and formatting before opening a PR:
   ```bash
   alya test
   ```
5. Commit your changes (`git commit -m "feat: add feature"`) and open a Pull Request

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
