# tz

[![CI](https://github.com/alya-lang/tz/actions/workflows/ci.yml/badge.svg)](https://github.com/alya-lang/tz/actions/workflows/ci.yml)
[![License](https://img.shields.io/github/license/alya-lang/tz?color=blue&label=License)](LICENSE)
[![Alya](https://img.shields.io/badge/dynamic/toml?url=https%3A%2F%2Fraw.githubusercontent.com%2Falya-lang%2Ftz%2Fmain%2Falya.toml&query=%24.package.alya-version&label=Alya&color=orange&prefix=%3E%3D)](https://github.com/alya-lang/alya)
[![Package Version](https://img.shields.io/badge/dynamic/toml?url=https%3A%2F%2Fraw.githubusercontent.com%2Falya-lang%2Ftz%2Fmain%2Falya.toml&query=%24.package.version&label=Version&color=brightgreen)](alya.toml)

IANA timezone database and zone-aware datetime conversion for Alya

---

## 🌟 Features

- 🌍 **53 Bundled IANA Zones**: Fixed-offset zones (exact) plus recurring current-era DST rules (US, EU, Australia, New Zealand, Egypt)
- ⏰ **Zone-Aware Conversion**: `epoch_to_zoned` / `zoned_to_epoch` roundtrips over `std/time`, with gap/fold documented behavior
- 📏 **Offset & Abbreviation Lookup**: `tz_offset_min` / `tz_abbrev` evaluated at any UTC epoch (transition-second precision)
- 🧩 **Modular Architecture**: Clean public facade (`src/lib.alya`), rule model (`src/types.alya`), conversion engine (`src/engine.alya`), embedded data (`src/tzdata/zones.alya`)
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
│   └── tzdata/             # Embedded zone table (mirrors V's time.tzdata role)
│       └── zones.alya      # 53 bundled rules + alias canonicalization
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
| `tz_offset_min(name, epoch)` | `pub function` | UTC offset in minutes east of UTC active at the epoch. Throws `TzError` for unknown zones. |
| `tz_abbrev(name, epoch)` | `pub function` | Active zone abbreviation (`"EST"`/`"EDT"`). Throws `TzError` for unknown zones. |
| `epoch_to_zoned(epoch, name)` | `pub function` | UTC epoch to zoned calendar map (`year`..`weekday` plus `offset_min`, `abbr`, `iana`). Throws `TzError` for unknown zones. |
| `zoned_to_epoch(d, name)` | `pub function` | Zoned wall-time map to UTC epoch (gaps resolve forward, folds to standard-time occurrence). Throws `TzError` for unknown zones or invalid dates. |
| `tz_rule(name)` | `pub function` | Zone rule lookup, or null when the zone is not bundled. |
| `tz_is_valid(name)` | `pub function` | 1 for bundled zones/aliases, 0 otherwise. |
| `tz_names()` | `pub function` | Array of all 53 bundled IANA zone names. |
| `tz_canonical(name)` | `pub function` | Alias canonicalization (`"UTC"` → `"Etc/UTC"`). |
| `TzRule` | `pub struct` | Zone rule model (offsets, recurring DST schedule, UTC/local transition flags). |
| `TzRule.has_dst()` | `pub method` | 1 for DST zones, 0 for fixed zones. |
| `TzError` | `pub struct` | Thrown error (`message`). |
| `fixed_rule(name, abbr, offset_min)` | `pub function` | Factory for fixed-offset zones. |
| `dst_rule(name, std_abbr, dst_abbr, std, dst, sm, sw, sd, sh, su, em, ew, ed, eh, eu)` | `pub function` | Factory for DST zones (nth-weekday rules, UTC/local flags). |

> [!TIP]
> **Scope:** Rules are recurring current-era schedules (US post-2007, EU post-1996). Fixed zones are exact going forward; pre-changeover historical dates are out of scope for v0.1.0. Public symbols are documented with `##` docstrings for `alya doc`.

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
