# CS2 No-Recoil Logitech Script Macro

> **Use at your own risk.** This project is intended for single-player / offline
> and local-server use, and for studying recoil patterns. Do not use it on
> third-party competitive platforms.

---

## Files

| File | Description |
| --- | --- |
| `CS2-Ethan-v3.lua` | **Recommended.** Rewritten, fully bilingual (CN/EN) comments. Keyboard G-keys + mouse buttons, single/double click on both. 17 weapon tables. |
| `CS2-(2.40-2.60)Sens-rebound.lua` | Legacy. Auto-rebounds to the initial position when the macro finishes or the left button is released. |
| `CS2-(2.40-2.60)Sens-no-rebound.lua` | Legacy. Same as above but does **not** rebound. |

The only difference between the two legacy files is `round = 40` (rebound)
versus `round = 0` (no-rebound).

---

## CS2-Ethan-v3.lua

### Key bindings

| Key | Single click | Double click |
| --- | --- | --- |
| `G1` | AK47 | — |
| `G2` | M4A1-S | M4A4 |
| `G3` | GALIL AR | FAMAS |
| `G4` | SG 553 | AUG |
| `G5` | Negev | M249 |
| `Mouse 4` | Close all | (configurable) |
| `Mouse 5` | Universal SMG | (configurable) |

- **Single click** takes effect immediately.
- **Double click** = a second press within `DOUBLE_WINDOW` (default 280 ms).
- Hold `Left Alt` to temporarily suspend recoil control (`SUSPEND_KEY`).

### Supported weapons (17 tables)

All weapon tables ship with the script, so any of them can be bound without
touching the data:

```
Rifles      ak47  m4a1  m4a4  galil  famas  sg553  aug
SMGs        p90  bizon  mp9  mac10  ump45  mp7  mp5sd
LMG / MG    m249  negev
Pistol      cz75
```

`negev` covers a full 150-round magazine: fire rate 75 ms/round (800 RPM) for a
total of 11250 ms, cumulative pull 170, and no horizontal drift. The pattern
climbs for the first 15 rounds and then holds flat.

The 7 raw SMG tables are kept as standby data. The `"smg"` action on mouse
button 5 does **not** reference them directly — it uses a composite curve
(a per-round median across all 7 SMGs) so one button covers every SMG.

> A single curve cannot be exact for all 7 SMGs at once. The composite curve was
> fitted to keep the average error low; individual SMGs will still deviate,
> mostly toward the end of the magazine where magazine capacities differ.

### Configuration

Everything you normally need to change sits at the top of the file:

| Variable | Meaning |
| --- | --- |
| `DOUBLE_WINDOW` | Double-click window in ms, shared by keyboard and mouse |
| `SUSPEND_KEY` | Hold to pause recoil control; `nil` disables |
| `SCALE_Y` | Vertical strength — lower if over-compensating |
| `SCALE_X` | Horizontal strength |
| `LAG_TOLERANCE` | Allowed timing drift in ms before resync |
| `HOLD_AT_END` | `true` = hold position when the table ends (prevents snap-back) |
| `round` | Recoil recovery subdivision steps |
| `speed` | Recovery speed |
| `wireless` | `1` for wireless mouse, `0` for wired |
| `OUTPUT_BANNER` | Print the banner on weapon switch |
| `SMG_EXTEND` | Keep compensating after the SMG curve ends |
| `SMG_PLATEAU` | Loop start point as a fraction of the curve |
| `SMG_GAIN` | Overall SMG strength |

### Adding more mouse buttons

`MOUSEMAP` accepts any button number — there is no upper bound:

```lua
MOUSEMAP = {
    [4] = { single = "off",  double = nil      },
    [5] = { single = "smg",  double = nil      },
    [6] = { single = "p90",  double = "cz75"   },   -- add your own
    [8] = { single = "mp9",  double = nil      },
}
```

Allowed values are a weapon name, `"smg"`, `"off"`, or `nil`.
Unmapped button numbers are silently ignored.

**Button numbering:** `4` and `5` are the two side buttons on virtually every
Logitech mouse, and that is what the default layout uses. Numbers `6` and above
**vary by mouse model** — tilt wheel, DPI keys and extra buttons all get their
own numbers, and the mapping differs between devices. Do not guess them.

Note that the numbering used for event `arg` is not guaranteed to match the
numbering accepted by `PressMouseButton` / `IsMouseButtonPressed` (some
Logitech APIs count 1 = left, 2 = middle, 3 = right, others differ). If you mix
event binding with simulated clicks, verify both directions.

To discover the numbers on your own hardware, load this snippet once in G HUB
and press each button:

```lua
function OnEvent(event, arg)
    if (event == "MOUSE_BUTTON_PRESSED") then
        OutputLogMessage("mouse button = %d\n", arg)
    end
end
```

---

## Installation

1. Install Logitech G HUB and connect a programmable Logitech mouse.
2. Run G HUB as administrator.
3. Create a new script for your device profile.
4. Paste the contents of `CS2-Ethan-v3.lua` into the script editor, or import
   the file directly.
5. Save and run.
6. Edit `KEYMAP` / `MOUSEMAP` at the top of the script to match your hardware.

### Loading from a file outside the C: drive

Some users have reported bans. If you prefer to keep the macro outside G HUB's
script folder:

1. Place the `.lua` file somewhere other than the `C:` drive.
2. Create a new script in G HUB.
3. Set the script body to:

   ```lua
   data = dofile[[D:\yourmacrolocation.lua]]
   ```

4. Save and run.

> **This approach does not work on current G HUB builds.** The G HUB Lua VM is a
> restricted sandbox. In practice `dofile` resolves to `nil` at runtime, so the
> script fails immediately with:
>
> ```
> [string "LuaVM"]:1: attempt to call a nil value (global 'dofile')
> ```
>
> The same applies to `loadfile` and `io`. Paste the script contents directly
> into the G HUB script editor instead — G HUB accepts exactly one script file
> per profile, so a `.lua` kept on another drive cannot be loaded this way.

---

## Q&A

### How does this work?

1. You need a programmable Logitech mouse (or a Logitech keyboard with G-keys).
2. Make sure G HUB is running as administrator.
3. Download the script and import it into G HUB.
4. Bind the weapons you use to keys in `KEYMAP` / `MOUSEMAP`.
5. Enter the game and press the key for your weapon.

> The **legacy** scripts (`CS2-(2.40-2.60)Sens-*.lua`) indicate their state via
> the Scroll Lock LED: it lights up while the macro is active and goes out when
> it is turned off. If your keyboard has no Scroll Lock LED, you get no feedback.
>
> `CS2-Ethan-v3.lua` does **not** use any keyboard LED. It needs no state
> indicator because every weapon has its own key — press the key for the weapon
> you are holding. State is also printed to the G HUB log on every switch.

### What is the difference between rebound and no-rebound?

The rebound version automatically returns to the initial position when the
macro finishes or the left mouse button is released. The no-rebound version
does not.

### What do "single click", "double click" and "Ctrl + side button" mean?

- **Single click** — click the mapped button once to enable that weapon.
- **Double click** — click the mapped button twice within the window.
- **Ctrl + side button** — hold `Ctrl` and click the mapped side button once.
  (Legacy scripts only; `CS2-Ethan-v3.lua` uses the single/double scheme.)

### How do I switch macros between different weapons?

Press the key bound to the weapon you picked up. If you are unsure which macro
is currently active, press the close key (`Mouse 4` in the default layout) and
then press the key for your new weapon.

---

## Verification status

The scripts in this repository have been **syntax-checked with `luaparse`
(Lua 5.1)**. The recoil data was verified numerically against the source
tables.

They have **not** been validated in-game across every weapon and sensitivity
setting. Recoil data is sensitive to in-game sensitivity, DPI, and mouse
polling rate — expect to tune `SCALE_Y` / `SCALE_X` for your own setup.

Specific things worth knowing:

- **Sensitivity is not bounded by this script.** The legacy scripts were tuned
  for in-game sensitivity 2.40–2.60; `CS2-Ethan-v3.lua` carries the same
  sensitivity assumption in its data. Different settings will need different
  scaling.
- **No runtime test was performed** in G HUB for the v3 script. Checks were
  static: Lua 5.1 syntax parsing, row-by-row comparison of every weapon table
  against the source files, and numeric verification of totals and durations.

---

## Credits

- **Author:** Ethan
- **README:** rewritten in English, documenting the v3 script.
- **Assisted by DeepSeek V4.1 Flash** — code restructuring, bilingual inline
  comments, weapon table assembly, numerical verification of recoil data, and
  this README.
