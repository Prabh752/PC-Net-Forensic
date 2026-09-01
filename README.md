# 📡 WiFi Forensics Extractor

A simple Windows script that collects WiFi-related evidence from a computer — no installs, no third-party tools, just built-in Windows commands.

If you're doing digital forensics, incident response, or just need to document a machine's WiFi history, this script does the boring collection work for you in one click and saves everything neatly into a folder.

---

## What does it actually do?

In plain English, it opens Command Prompt and runs a bunch of Windows commands that already exist on every Windows PC — things like `netsh` and `ipconfig` — and saves their output as text files. It doesn't download anything, doesn't connect to the internet, and doesn't install anything on the computer.

Here's what you'll find in the folder it creates:

- 📋 **Every WiFi network the computer remembers** — including the saved passwords (if you run it as admin)
- 📶 **The WiFi networks it can currently see** around it
- 🖥️ **Info about the WiFi adapter/driver** — make, model, driver version
- 🌐 **Full network configuration** (IP address, DNS, MAC address, etc.)
- 📜 **Windows' own connection history logs** — when it connected/disconnected from networks and when
- 🗂️ **Registry entries** that record which networks the PC has joined and when it first/last saw them
- 🔗 **A snapshot of current network connections** at the time you ran it
- 🔒 **A hash file** — basically a fingerprint of every file collected, so you can prove later that nothing was tampered with

---

## How to use it

1. Download `WiFi_Forensics_Extractor.bat`
2. Right-click it and choose **"Run as administrator"**
   *(You can run it without admin rights too, but you'll miss the saved passwords and some logs.)*
3. A black window will pop up and show its progress — just let it finish
4. When it's done, look next to the script for a new folder named something like:
   ```
   WiFi_Forensics_YOURPCNAME_2026-09-01_14-30-00
   ```
5. Open that folder — everything is saved as plain `.txt` files you can open with Notepad

That's it. No setup, no menus, no configuration needed.

---

## Why run it as admin?

Some information — like saved WiFi passwords and detailed connection logs — is protected by Windows and only visible to an administrator. If you run it as a normal user, the script still works, it just won't be able to grab those specific pieces.

---

## A few honest notes before you use it

- **This only works on Windows.** It won't run on Mac or Linux.
- **Running it on a computer changes it slightly** — for example, it "touches" the event logs when reading them. If you're doing serious forensic work where the evidence needs to stay 100% untouched, it's better to run this on a **copy/image of the drive**, not the original live machine.
- **Only use this on computers you own or have permission to check.** Pulling saved WiFi passwords off someone else's computer without permission is not okay, and in many places it's illegal.
- **The hash file is there to protect you.** If anyone ever questions whether the collected files were altered afterward, you can re-check the hashes to prove they weren't.

---

## Requirements

- Windows 10 or 11
- That's it — no extra software needed

---

## License

Free to use and modify (MIT License). Just use it responsibly.
