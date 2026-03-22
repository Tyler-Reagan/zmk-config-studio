# ZMK Config — TOTEM Split Keyboard

Personal ZMK firmware for the [TOTEM](https://github.com/GEIGEIGEIST/TOTEM) — a 38-key column-staggered wireless split keyboard running on two [Seeed XIAO nRF52840](https://wiki.seeedstudio.com/XIAO-nRF52840/) microcontrollers.

- **Hardware files & build guide:** [GEIGEIGEIST/TOTEM](https://github.com/GEIGEIGEIST/TOTEM)
- **QMK/Vial config (wired):** see `qmk-config-totem/` — same layout, wired TRRS variant
- **Firmware source:** [zmkfirmware/zmk](https://github.com/zmkfirmware/zmk) (`main` branch)

---

<details>
<summary><strong>Building firmware</strong></summary>

Firmware is built automatically by GitHub Actions on every push to `main`. The workflow produces two UF2 artifacts:

| Artifact | Half | Notes |
|---|---|---|
| `xiao_ble-totem_left` | Left (central) | Built with ZMK Studio + USB UART snippet |
| `xiao_ble-totem_right` | Right (peripheral) | Standard build |
| `xiao_ble-settings_reset` | Either | Clears EEPROM / BLE bond data |

The build matrix is defined in [`build.yaml`](build.yaml).

### Building locally

```sh
# Install west
pip3 install west

# Bootstrap ZMK workspace
west init -l config
west update

# Build left half
west build -s zmk/app -b xiao_ble \
  -- -DSHIELD=totem_left \
     -DZMK_CONFIG=$(pwd)/config \
     -DSNIPPET=studio-rpc-usb-uart \
     -DCONFIG_ZMK_STUDIO=y \
     -DCONFIG_ZMK_STUDIO_LOCKING=n

# Build right half
west build -s zmk/app -b xiao_ble \
  -- -DSHIELD=totem_right \
     -DZMK_CONFIG=$(pwd)/config
```

</details>

---

<details>
<summary><strong>Flashing</strong></summary>

The XIAO nRF52840 appears as a USB mass storage device named **`XIAO-SENSE`** (or **`XIAO`**) in bootloader mode.

**Enter bootloader mode**
1. Double-tap the reset button on the XIAO — the drive mounts automatically.
2. Or: hold the boot button while plugging in USB.

**Flash**
Drag-drop the UF2 onto the mounted drive. The board reboots automatically when the copy completes.

> Flash **one half at a time** with the other disconnected.

**First-time pairing**
1. Flash both halves.
2. Power on both. The right half advertises to the left; they pair automatically.
3. The left half then advertises to the host. Open Bluetooth settings and pair **TOTEM**.

**Reset bonds** (if halves stop communicating or host won't reconnect)
1. Flash `xiao_ble-settings_reset.uf2` to both halves.
2. Re-flash the normal firmware to both halves.
3. Re-pair from scratch.

</details>

---

<details>
<summary><strong>ZMK Studio</strong></summary>

[ZMK Studio](https://zmk.studio/) enables live keymap editing over USB without reflashing.

- Connect the **left half** via USB.
- Open [zmk.studio](https://zmk.studio/) in a Chromium-based browser.
- The TOTEM should appear automatically.
- Changes are saved to the keyboard's flash — no compile step needed.

> ZMK Studio only connects to the left (central) half. The right half does not expose the Studio interface.

Studio locking is **disabled** (`CONFIG_ZMK_STUDIO_LOCKING=n`) so no unlock sequence is required.

</details>

---

<details>
<summary><strong>Keymap</strong></summary>

Six layers. Layer-tap on all three thumb keys per side.

| # | Name | Primary hand | Activation |
|---|---|---|---|
| 0 | **BASE** | Both | Default |
| 1 | **DEV** | Right | Hold right-thumb inner (`DEV/SPC`) |
| 2 | **SYS** | Right | Hold right-thumb middle (`SYS/TAB`) |
| 3 | **NUM** | Left | Hold left-thumb middle (`NUM/ENT`) |
| 4 | **FUN** | Left | Hold left-thumb inner (`FUN/DEL`) |
| 5 | **BOOT** | Both | Assign via ZMK Studio or add a combo |

**BASE layer highlights**
- Home-row mods: `GUI/S` `CTRL/D` `SHIFT/F` (left), `SHIFT/J` `CTRL/K` `GUI/L` (right)
- Left outer pinky: `HYPER` (Ctrl+Shift+Alt+GUI)
- Right outer pinky: `'`
- Thumbs (left→right): `ESC` · `SYS/TAB` · `DEV/SPC` ‖ `BSPC` · `NUM/ENT` · `FUN/DEL`

**DEV** — developer symbols on the right hand (`-{}` `` ` `` `=_[]'$&|*`), modifiers on the left, `@()` on right thumbs, `Shift+Tab` on right outer pinky.

**SYS** — navigation and media on the right hand (arrows, vol, prev/next, screenshot, new tab, undo/cut/copy/paste), refresh/play/mute on right thumbs, lock screen on right outer pinky.

**NUM** — number pad layout on the left hand (`-789` / `=456` / `_123`, `.0` on thumbs), right-hand modifiers on the right.

**FUN** — function keys on the left hand (`F12 F7-F9` / `F11 F4-F6` / `F10 F1-F3`), `SPC/TAB` on left thumbs, right-hand modifiers on the right.

**BOOT** — `QK_RBT` (soft reset) on top-row outer keys, `QK_BOOT` (bootloader) on bottom-row outer keys. Access this layer by assigning a key or combo via ZMK Studio.

</details>

---

<details>
<summary><strong>BLE profiles</strong></summary>

ZMK supports up to 5 Bluetooth profiles, allowing the keyboard to pair with multiple hosts and switch between them.

Default bindings in the keymap (assign via ZMK Studio if needed):

| Binding | Action |
|---|---|
| `&bt BT_SEL 0` | Switch to profile 0 |
| `&bt BT_SEL 1` | Switch to profile 1 |
| `&bt BT_SEL 2` | Switch to profile 2 |
| `&bt BT_CLR` | Clear bond on active profile |
| `&bt BT_CLR_ALL` | Clear all bonds |

After switching to an unconnected profile, the left half begins advertising. Open Bluetooth on the new host and pair **TOTEM**.

</details>

---

<details>
<summary><strong>Onboard LED indicators</strong></summary>

The XIAO nRF52840 has a 3-colour onboard LED (active-low GPIO). Current configuration:

| LED | Colour | GPIO | Role |
|---|---|---|---|
| `led0` | Red | P0.26 | Unused |
| `led1` | Green | P0.30 | Battery level status (see below) |
| `led2` | Blue | P0.06 | Unused (see note) |

**Battery level (`led1` — green)**
When `CONFIG_ZMK_BATTERY_REPORTING=y` is enabled, the keyboard reports battery percentage to the host over BLE — visible in macOS menu bar / Windows system tray without any LED activity.

The `zmk,battery-level-status-led = &led1` chosen property is set in both overlays, ready to activate visual battery feedback. To enable LED blink indication, find the correct ZMK Kconfig symbol (search ZMK's `app/Kconfig` for `BATTERY_LEVEL`) and add it to the relevant `.conf` file. `CONFIG_ZMK_BATTERY_LEVEL_STATUS=y` is the likely name but has not been verified against the current ZMK main branch and has been left out to avoid breaking the build.

**BLE status (`led2` — blue)**
`zmk,ble-status-led` is intentionally omitted. The "solid on while connected" state would drain the battery continuously. It can be re-added to `totem_left.overlay` if ZMK's implementation is confirmed to be pulse-on-event rather than continuous:

```devicetree
/ {
    chosen {
        zmk,ble-status-led = &led2;
    };
};
```

Expected behaviour if re-enabled:
- **Fast blink** — advertising (searching for host)
- **Solid** — connected *(only safe if event-triggered, not continuous)*
- **Off** — disconnected, not advertising

</details>
