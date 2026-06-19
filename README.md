# StickFix

**Fix dead or drifting analog sticks — and a dead power light — on the XiFan R36 Max.**
One small Windows app, one click. It backs up first, so it's safe.

<p align="center">
<img width="662" height="920" alt="image" src="https://github.com/user-attachments/assets/05ad2f06-b2d0-4944-ae7b-2400bca221b9" />
</p>

---

## What it fixes

- **Dead analog sticks** — you push a stick and nothing happens.
- **Dead power light** — the LED stays off even though the console works fine.
- **Stick drift** — the cursor creeps on its own (adjustable, see *Deadzone* below).

It works by correcting one small settings file on your SD card. It does **not** reflash your
OS and it does **not** touch your games — and it **backs up the original first**, so you can
undo it any time.

## New in 4.0

- **Turn the power light on or off.** Like it lit while you play, or prefer it dark to save a
  little battery? Your call — the **Power LED** switch (or leave it on, same as before).
- **One-tap stick feel.** Presets for a fresh stick, a slightly drifty one, or a worn, drifty
  one — pick from the **Feel** menu instead of guessing at numbers.
- **See it before you commit.** **Preview changes** shows exactly what it'll do without
  writing anything, and **Verify card** later confirms a fix is still in place.
- **Fix a stick that points the wrong way.** Crossed, reversed, or left/right-swapped sticks
  can now be corrected too (see the FAQ).

Same one-click repair as always underneath — these are just extra controls when you want them.

## Is this for me?

**Yes** — if all of these are true:
- Your console is a **XiFan R36 Max** (board **RF3536K4KA**, RK3326 chip, two analog sticks).
- It's running **ROCKNIX** or **AURKNIX**.
- The sticks and/or power light stopped working *after* you moved to ROCKNIX/AURKNIX.

**No** — if:
- You're on the **stock EmuELEC** the console ships with — the sticks already work there.
- Your sticks already work fine.

## Download

Grab the latest **`StickFix.exe`** from the [**Releases**](../../releases) page.

It's a **single file** — no installer, nothing to set up, no Python needed. Download and run.

## How to use it

1. Put your console's **SD card** into your PC (a card reader works fine).
2. Run **`StickFix.exe`**.
3. It finds the card on its own and shows **"Compatible"**. Click **Backup & Repair card**.
4. Put the SD card back in the console and power on. Done — sticks and light work.

If anything looks off, click **Restore card from backup** and you're exactly back to before.

## Deadzone (optional)

Only matters if a stick **drifts** (menu moves when you're not touching it) or feels **sluggish**:

- **Drifts on its own** → put a bigger number in (try `96`), then Repair again.
- **Feels sluggish / slow to respond** → put a smaller number in.
- **Leave it blank** to not change it.

## FAQ

**Windows says "unknown publisher" / Defender flags it.**
That's the normal warning for a small app that isn't code-signed. Click *More info → Run
anyway*, or allow it in Defender. It only edits the one file on your SD card.

**It says my card isn't compatible.**
StickFix only works on the R36 Max's settings file. If the card is for a different console — or
isn't a ROCKNIX/AURKNIX card — it safely refuses rather than touch the wrong thing.

**My sticks work but point the wrong way (push up, cursor goes right).**
That's stick *direction* — and 4.0 **can** fix it now (swap the left/right sticks, or flip an
axis). It's a quick command rather than a button, so open an [issue](../../issues) with which
stick does what and I'll give you the exact line for your unit.

**Will it mess up my games or my card?**
No. It edits one settings file on the boot partition and saves a backup of the original first.
Your games partition isn't touched.

**Mac / Linux?**
The download is Windows-only for now.

## Found a bug or have a fix that didn't work?

Open an [issue](../../issues) — include your console model, which OS (ROCKNIX or AURKNIX), and
exactly what the sticks/light do before and after.

## Disclaimer

Homebrew tool — **use at your own risk.** Not affiliated with XiFan, ROCKNIX, AURKNIX, or
EmuELEC. Keep the backup it makes. It's reversible, but you are modifying your own device.

---

StickFix v4.0 — made by **Nookie**.
