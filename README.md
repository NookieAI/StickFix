# StickFix

**Fix dead or drifting analog sticks — and a dead power light — on the XiFan R36 Max.**
One small Windows app, one click. It backs up first, so it's safe.

<p align="center">
  <img src="https://github.com/user-attachments/assets/e8019f00-9fac-49a7-b238-3201a82a73c5" width="380" alt="StickFix">
</p>

---

## What it fixes

- **Dead analog sticks** — you push a stick and nothing happens.
- **Dead power light** — the LED stays off even though the console works fine.
- **Stick drift** — the cursor creeps on its own (adjustable, see *Deadzone* below).

It works by correcting one small settings file on your SD card. It does **not** reflash your
OS and it does **not** touch your games — and it **backs up the original first**, so you can
undo it any time.

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
That's a different issue (stick *direction*), and this tool doesn't change it. Open an
[issue](../../issues) and say exactly which stick does what.

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

StickFix v3.0 — made by **Nookie**.
