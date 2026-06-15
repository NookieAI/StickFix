<img width="622" height="832" alt="image" src="https://github.com/user-attachments/assets/e8019f00-9fac-49a7-b238-3201a82a73c5" />

# StickFix — R36 Max analog-stick + LED fix (AURKNIX / ROCKNIX)

If your **XiFan R36 Max** has dead analog sticks (or a dead power LED) on
**AURKNIX** or **ROCKNIX**, this fixes it. (The R36 Max ships with EmuELEC; this is
for units that have been upgraded to AURKNIX/ROCKNIX.) The sticks read
the full range again and the power LED lights while the unit is on.

It ships two ways. Use whichever you like:

1. **The patched device-tree** (`rk3326-aislpc-r36tmax.dtb`) — drop it in and go.
2. **StickFix** (`StickFix.py`) — a small tool that patches *your* dtb for you,
   keeps a backup, and can also retune the deadzone.

---

## What it actually fixes

The R36 Max ships a device-tree that's broken three ways under AURKNIX/ROCKNIX:

| Symptom | Cause | Fix |
|---|---|---|
| Sticks completely dead | the analog multiplexer is never enabled | adds the missing mux-enable GPIO |
| Sticks only read half each axis (down/left dead) | wrong analog driver | switches to the `odroidgo3-joypad` driver |
| Power LED stays off | the LED pins are wired but nothing drives them | grafts a `gpio-leds` node |

Everything else — screen, sound, rumble, battery — is left exactly as it was.

> **One thing the dtb can't fix: stick *direction*.** If a stick reads but points
> the wrong way (crossed or reversed), that's hardware wiring, and no device-tree
> edit can swap a stick's axes on this driver. You fix it **once**, on the device
> — see [Fixing stick direction](#fixing-stick-direction) below. It takes 20 seconds.

---

## Requirements

- XiFan **R36 Max** (board RF3536K4KA, RK3326 / PX30, 720×720 screen)
- **AURKNIX** or **ROCKNIX** installed
- Access to the SD card's **boot / FAT partition** (the small one Windows can see —
  it shows up as a drive letter like `H:`)

The device-tree the R36 Max boots is named `rk3326-aislpc-r36tmax.dtb` and lives on
that FAT partition.

---

## Option 1 — just use the patched dtb

1. Power off, pull the SD card, plug it into your PC.
2. On the **boot/FAT partition**, find `rk3326-aislpc-r36tmax.dtb`.
3. **Back up the original** (copy it somewhere safe) — always.
4. Copy the `rk3326-aislpc-r36tmax.dtb` from this repo over the top.
5. Eject, boot the handheld. Sticks alive, LED on.
6. If a stick points the wrong way, do [the direction step](#fixing-stick-direction).

That's it. If you'd rather the tool do it (and keep its own backup), use Option 2.

---

## Option 2 — StickFix (the tool)

`StickFix.py` is a single self-contained Python file. No pip installs, no internet.
Python 3.8+ (Windows users: install Python from python.org, tick *Add to PATH*).

### Graphical version

Run it with no arguments and a window opens:

```
python StickFix.py
```

Insert your SD card, press **Scan for card**, then **Backup & Repair card**. It
backs up every original to `.stickfix/backups/<timestamp>/` on the card first, so
you can always undo. Eject and boot.

### Command line

```bash
# See the current state of a dtb (driver, mux-enable, LED, deadzone)
python StickFix.py inspect H:\rk3326-aislpc-r36tmax.dtb

# Patch it in place (keeps a .stickfix-bak next to it)
python StickFix.py fix H:\rk3326-aislpc-r36tmax.dtb --in-place

# Patch and also set a custom deadzone in one go
python StickFix.py fix H:\rk3326-aislpc-r36tmax.dtb --in-place --deadzone 48

# Undo (restore from the .stickfix-bak)
python StickFix.py restore H:\rk3326-aislpc-r36tmax.dtb
```

Without `--in-place` it writes a new `…-repaired.dtb` beside the original and leaves
yours untouched, so you can try it safely first.

StickFix only ever touches an **R36-class** dtb. Point it at anything else and it
says so and does nothing — it can't corrupt the wrong file.

---

## Fixing stick direction

After the sticks are alive, if one reads but points the wrong way:

1. On the handheld, open **EmulationStation**.
2. **Main Menu → Controller Settings → Configure a Controller**
   (on some builds: **Main Menu → Configure Input**).
3. Walk through the prompts and, for each stick direction, push the stick **the way
   it physically asks** — if it says "UP", push up; if the axis is crossed, push the
   direction that actually moves the on-screen highlight the way it wants.
4. Finish the wizard. Done — it sticks across reboots.

This is a normal EmulationStation remap. It's the correct place to fix wiring-level
axis quirks, and it's why the dtb deliberately doesn't try to.

---

## Deadzone tuning (optional)

The deadzone is the dead patch in the centre of each stick. Stock is `64`.

- **Stick feels twitchy / too sensitive at rest** → lower it (try `32`).
- **Stick drifts or the cursor creeps when you let go** (common on worn sticks) →
  raise it (try `96`–`128`).

```bash
python StickFix.py deadzone H:\rk3326-aislpc-r36tmax.dtb 96 --in-place
```

Change it, boot, see how it feels, adjust. There's no wrong answer — it's taste.

---

## Troubleshooting

**Sticks still dead after flashing.**
Make sure you actually replaced the file the device boots
(`rk3326-aislpc-r36tmax.dtb` on the FAT partition) and that it saved. Run
`python StickFix.py inspect <file>` — it should say *Mux enable: present* and
*Driver: odroidgo3-joypad*.

**A stick reads but points wrong.**
That's the direction step above — it's not a dtb problem.

**LED still off.**
`inspect` should say *Status LED: present*. If it does and the LED is still dark,
your unit may wire the LED differently; open an issue with your board photo.

**An OS/AURKNIX update killed it again.**
Updates can overwrite the dtb on the FAT partition. Just re-apply (re-flash the dtb
or re-run StickFix). Keep a copy of your patched dtb somewhere off the card.

**Restoring the original.**
Tool GUI: *Restore card from backup*. CLI: `restore`. Or copy back the original you
saved before you started.

---

## Credits

Made by **Nookie**

Built and verified on a real R36 Max running AURKNIX. The deep dive on *why* each
change is needed (and why direction can only be fixed on-device) is in
[`TECHNICAL_NOTES.md`](TECHNICAL_NOTES.md).

Use at your own risk — but it backs up before it touches anything, and it won't
touch a dtb that isn't an R36 Max.
