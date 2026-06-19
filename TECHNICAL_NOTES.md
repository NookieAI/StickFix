# R36 Max — device-tree technical notes

Everything learned fixing the **XiFan R36 Max** analog sticks + LED on AURKNIX,
written down so it never has to be reverse-engineered again. If an OS update wipes
the dtb, or the tool isn't to hand, this is enough to redo the whole thing by hand.

Everything below was **verified on real hardware** unless explicitly marked otherwise.

---

## 1. The hardware

| | |
|---|---|
| Device | XiFan R36 Max |
| Board | RF3536K4KA |
| SoC | Rockchip **RK3326** (PX30 family) |
| Screen | 720×720 MIPI-DSI |
| Audio | RK817 codec |
| Sticks | dual analog, read through a **4-channel analog multiplexer** on **gpio2** |
| OS as shipped | **EmuELEC** (this unit was upgraded to AURKNIX, a ROCKNIX-based clone) |

Both sticks share one SARADC channel through a mux. Two GPIOs select which of the
four axes is on the ADC at any moment (`amux-a` / `amux-b`), and a third GPIO
**enables** the mux (`amux-en`). No enable = the mux never conducts = sticks dead.

---

## 2. Which device-tree actually boots

The boot/FAT partition's `extlinux/extlinux.conf` selects **one** dtb:

```
FDT /rk3326-aislpc-r36tmax.dtb          # model: "AISLPC R36TMax"
FDTOVERLAYS /overlays/mipi-panel.dtbo
```

So the file to edit is **`rk3326-aislpc-r36tmax.dtb`** (model string *AISLPC
R36TMax*). Ignore every other dtb on the card — this hardware boots only this one.

**The overlay matters.** `mipi-panel.dtbo` is layered on top at boot and it:

- overrides the panel timings,
- enables `adc-keys`,
- force-sets `invert-absx` + `invert-absy` on the **left** stick (via a
  `__fixups__: joypad` entry),
- enables the RK817 sound + headphone-detect.

The overlay does **not** touch the right stick, and it binds to the joypad node by
the **`joypad` symbol** in `__symbols__`. That's why the patched dtb must keep that
symbol intact (it does — only properties change, the node path doesn't).

> Note: ROCKNIX's hardware detection identifies this unit as **`eeclone`**. The
> `eeclone` reference dtb is where the correct mux-enable pin and the `gpio-leds`
> node came from — it's the same silicon.

---

## 3. The fix — exactly what changes, and why

Three changes to the joypad node / root. Nothing else.

### 3a. Driver: `rocknix-singleadc-joypad` → `odroidgo3-joypad`

```
compatible = "odroidgo3-joypad";
```

The stock node uses the **singleadc** driver. On AURKNIX that driver binds, but it
only reads the **positive half** of each axis — down and left come back dead. The
**odroidgo3** driver reads the full ±range. This is the difference between "half the
stick works" and "all of it works". [verified: swapping only the compatible string
changed half-dead axes into full-range ones.]

### 3b. Add the mux-enable GPIO (THE dead-stick fix)

```
amux-en-gpios = <&gpio2 0x0c 0x01>;     # gpio2 PB4, active-high
```

The stock AISLPC node is **missing this property entirely**. Without it the analog
mux is never switched on, so both sticks are stone dead regardless of driver. Adding
it is the single most important change. [verified: adding only this revived the
sticks from fully dead to reading.] The pin value (`0x0c` = PB4) came from the
`eeclone` reference.

### 3c. Graft a `gpio-leds` node (LED fix)

The AISLPC profile defines the `led-pins` pinctrl (it muxes the LED pins) but never
adds the **device** that actually drives them, so the power LED stays off. Grafting
the `gpio-leds` node from `eeclone` fixes it:

```
gpio-leds {
    compatible = "gpio-leds";
    pinctrl-names = "default";
    pinctrl-0 = <&led_pins>;
    led-0 {                              # charge LED
        color = <1>; function = "charging";
        default-state = "off"; linux,default-trigger = "none";
        gpios = <&gpio0 0x11 0x00>;      # gpio0 PC1
    };
    led-1 {                              # power LED — lit while running
        color = <3>; function = "power";
        default-state = "on"; linux,default-trigger = "default-on";
        gpios = <&gpio0 0x00 0x00>;      # gpio0 PA0
    };
};
```

[verified: "led is working!" after this graft.] The two LED pins are fixed by the
board: **charge = gpio0 PC1 (0x11)**, **power = gpio0 PA0 (0x00)**.

#### Turning a LED on or off (and triggers)

Whether a grafted LED lights is governed by two string properties on its child node:

| Want | `default-state` | `linux,default-trigger` |
|---|---|---|
| **on** (lit while running) | `on` | `default-on` |
| **off** (dark) | `off` | `none` |

The kernel forces the LED on if *either* `default-state="on"` *or*
`linux,default-trigger="default-on"`, so turning it truly **off** needs both `off` *and*
`none` — setting only `default-state="off"` while the trigger stays `default-on` leaves
it lit. StickFix's `led on`/`led off` set both together. The change is a property edit on
the existing child (the node is never removed), so it's reversible and the panel overlay's
`joypad` symbol is untouched.

`linux,default-trigger` can also be any kernel LED trigger — `heartbeat` (CPU-load
double-pulse), `mmc0`/`disk-activity` (SD access), `timer` (steady blink), `panic`. These
turn the indicator LED into a live activity light and are pure device-tree changes; an
unknown trigger just leaves the LED governed by `default-state`. StickFix whitelists the
set it writes (`led trigger <name>`), so it can never write a garbage trigger string.

A LED child is matched by its **`function`** value (`"power"` / `"charging"`) and then its
`gpios` pin, not by node name, so an existing (e.g. eeclone) `gpio-leds` node with
differently-named children is still driven correctly.

---

## 4. Stick direction — fixable in the dtb (corrected)

**Direction — a crossed stick, a reversed stick, and the left/right sticks being
swapped — is all fixable in the device tree.** This corrects an earlier conclusion in
this project (that "the mapping is sign-only and the inverts are ignored, so direction
can only be fixed in-device via *Configure a Controller*"). That came from one muddy
test and was **disproven by reading the driver source**.

**`amux-channel-mapping` is a free channel→axis assignment.** Order is
`<RY RX leftY leftX>` (position 0→ABS_RY, 1→ABS_RX, 2→ABS_Y, 3→ABS_X); each value is the
mux channel feeding that axis. So this one array sets **both** which physical stick is
left vs right **and** which pot is X vs Y on each stick. The four `invert-*` flags are
**honoured** per axis (sign). [verified from `rocknix-singleadc-joypad.c`.]

**The left/right sticks were swapped** on this unit, and the original direction reports
had them mislabeled (the right-hand stick was called "left"). Swapping the two channel
pairs corrects it: `<2 3 0 1>` → **`<0 1 2 3>`** (the default routing). The patched dtb
now carries **`<0 1 2 3>`** with inverts off; the corrected overlay drops the forced
left inverts so sign is controllable from the base. **Pending a hardware flash-test.**

StickFix can apply this: `swap` (swap L/R, reversible) and `mapping <a b c d>` (set the
routing directly). The **default repair leaves direction alone** — it only revives the
sticks (full range) and the LED — because the exact direction values are per-unit and
must be confirmed on the hardware; direction is opt-in until then.

> Driver note: `rocknix-singleadc-joypad` honours the mapping but on AURKNIX reads only
> the positive half of each axis (down/left dead), which is why **odroidgo3** is the
> driver used here. odroidgo3 is a sibling driver assumed to share the same mapping/
> invert semantics — the one inference still pending the flash-test.

---

## 5. Deadzone & feel

The relevant joypad properties (raw ADC counts):

| Property | Stock | Meaning |
|---|---|---|
| `button-adc-deadzone` | `0x40` (64) | dead patch at stick centre |
| `button-adc-fuzz` | `0x20` (32) | noise hysteresis |
| `button-adc-flat` | `0x20` (32) | reported flat/centre region |
| `abs_{x,y,rx,ry}-{p,n}-tuning` | `0xc8` (200) | per-axis ± scaling |

StickFix exposes `button-adc-deadzone` directly (`deadzone <dtb> <n>`; lower = snappier,
higher = tames a worn, drifting stick). It also bundles `deadzone`, `fuzz` and `flat` into
named **feel presets** (`preset <dtb> <name>`), each a `(deadzone, fuzz, flat)` triple:

| Preset | deadzone | fuzz | flat | For |
|---|---|---|---|---|
| `snappy` | 24 | 16 | 16 | fresh sticks, tight centre |
| `stock` | 64 | 32 | 32 | the vendor values |
| `drift-tamer` | 128 | 40 | 48 | a little drift |
| `worn` | 192 | 48 | 64 | a tired, drifting stick |

`*-tuning` (per-axis range) is still left at stock; edit it by hand if you want to rescale.

---

## 6. Re-applying by hand (no tool)

If you ever need to redo this from a stock dtb with just `dtc`:

```bash
# 1. decompile
dtc -I dtb -O dts rk3326-aislpc-r36tmax.dtb -o r36.dts

# 2. in r36.dts, in the joypad node:
#    - set:  compatible = "odroidgo3-joypad";
#    - add:  amux-en-gpios = <&gpio2 0x0c 0x01>;   (and amux-count = <4>; if absent)
#    - set amux-channel-mapping = <0 1 2 3>; (swap-corrected) and remove invert-* lines
#      (or just run: StickFix swap <dtb>)
# 3. add the gpio-leds node from section 3c at root level
#    (referencing &gpio0 and &led_pins, or their numeric phandles)

# 4. recompile (-@ keeps __symbols__ so the panel overlay still binds)
dtc -@ -I dts -O dtb r36.dts -o rk3326-aislpc-r36tmax.dtb
```

Then back up the original on the FAT partition and copy the new one over it.

`StickFix.py` does exactly this surgery via a vendored copy of python-fdt (so it
needs no `dtc` on the target), resolving the `&gpio0` and `&led_pins` phandles from
whatever dtb you feed it rather than hard-coding them — so it stays correct across
R36 Max profile variants.

---

## 7. Phandle / pin reference (this dtb)

For convenience when editing by hand. Phandle values are specific to the AISLPC dtb;
re-read them if you start from a different one.

| Node | Role | Phandle (AISLPC) |
|---|---|---|
| `gpio@ff040000` | gpio0 — status-LED bank | `0x57` |
| `gpio@ff260000` | gpio2 — mux select + enable | `0xad` |
| `gpio@ff270000` | gpio3 — rumble / d-pad | `0x97` |
| `led-pins` | LED pinctrl | `0x139` |

Pin map:

| Function | GPIO | Pin |
|---|---|---|
| mux select A | gpio2 | PC0 (`0x10`) — *the R36 signature* |
| mux select B | gpio2 | PB7 (`0x0f`) |
| **mux enable** | gpio2 | PB4 (`0x0c`) — *missing on stock = dead sticks* |
| charge LED | gpio0 | PC1 (`0x11`) |
| power LED | gpio0 | PA0 (`0x00`) |

StickFix recognises an "R36-class" target by exactly one cheap check: the joypad's
`amux-a-gpios` points at **gpio2, pin PC0**. Everything else (gpio3-mux boards,
direct-ADC boards, unrelated handhelds) fails that check and is skipped untouched.

---

## 8. The short version

- Edit `rk3326-aislpc-r36tmax.dtb` (model *AISLPC R36TMax*); a panel overlay binds to
  its `joypad` symbol, so don't break the node path.
- Driver → `odroidgo3-joypad`, add `amux-en-gpios = <&gpio2 0x0c 0x01>`, graft
  `gpio-leds`. That's the whole fix.
- Direction IS dtb-fixable: the mapping is a free channel→axis assignment and the
  inverts are honoured. L/R was swapped here → mapping `<0 1 2 3>`; apply via StickFix
  `swap` / `mapping` (default repair leaves direction alone until confirmed).
- Deadzone is `button-adc-deadzone` (stock 64): lower = snappier, higher = tames drift.

*Made by Nookie*
