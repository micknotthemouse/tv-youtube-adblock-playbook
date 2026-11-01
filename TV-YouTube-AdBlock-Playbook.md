# Playbook: ad-free YouTube on your Smart TV (hand this file to your Claude)

Written 2026-10-31 from a real run on an LG OLED65C5AUA (webOS 25). Every step in Route A was done and returned Success. **What was NOT verified:** that ads are actually blocked in playback (nobody could watch the TV screen during the run). Treat "ads gone" as unproven until the human confirms it.

---

## 0. For the human: how to use this file

1. Use a Windows, Mac or Linux computer on **the same home Wi-Fi as the TV** (not a guest network, not on a VPN).
2. Open Claude Code (or any AI agent that can run shell commands on that computer) and paste: **"Read this whole file and do it for my TV. Ask me the questions in section 2 first, in one message."**
3. Be near the TV with the remote. Claude does the computer work; you do a few short steps on the TV (section 5, Phase C) and confirm at the end that ads are gone.

Cost: free. Time: about 20-40 minutes. Everything below is reversible (section 9).

---

## 1. Instructions to the agent (Claude): rules of engagement

**Goal:** remove YouTube ads on the user's TV as well as possible, with no risk of damaging the TV. Risks are acceptable only if reversible.

- **Do not root or jailbreak the TV.** It is not needed for this goal, carries brick/warranty risk, and no system-wide tool removes YouTube in-video ads (YouTube serves ads and video from the same domains, so DNS/hosts blocklists such as Pi-hole cannot separate them). If a research step is refused by a safety filter while looking at rooting tools, do not retry or rephrase it; rooting is out of scope.
- **Never use the `root` / port 22 / Homebrew-Channel-SSH instructions** you may see in READMEs. Use only the official Developer Mode route (user `prisoner`, port 9922).
- **Ask the questions in section 2 in ONE message, then proceed.** Only pull the user in where a human is truly required (the TV remote steps, the final viewing test).
- **Plain language with the user.** No jargon, no file paths. Tell them exactly what to press on the TV.
- **Secrets:** the TV's 6-character passphrase and the downloaded private key are private. Do not paste them into notes, commit them, or send them anywhere. Keep the key in the user's `~/.ssh` folder only.
- **Verify downloads:** compare each file's SHA-256 against GitHub's published value before installing.
- **Be honest about outcome:** do not say "ads are blocked" until the user has played a video that normally shows ads and reported back. Report skipped or failed steps plainly.
- **Windows note:** use PowerShell for scripting. If `python` is missing, do not install it; PowerShell is enough. If PowerShell refuses to run an `ares-*` command because of script policy, call the `.cmd` shim (for example `ares-install.cmd`).
- Keep a short notes file for the user (what was installed, the TV's IP, how to undo it, when to extend the session). No secrets in it.

---

## 2. Intake questions (ask all at once, then continue)

| # | Question to ask the user | Why it matters / how it changes the plan |
|---|---|---|
| 1 | What brand is the TV (LG, Samsung, Sony/Google TV, TCL, Hisense, Vizio, Roku, Fire TV, Apple TV, other)? | Picks the route in section 3. Only LG webOS (Route A) was done and tested. |
| 2 | Exact model name/number and year? For LG: *Settings > General > About This TV* — what are the **webOS version** and software version? | LG builds differ: webOS 3-6 = "Legacy" build, webOS 22-25 = "Modern" build. webOS 1-2 (2014-15) is unsupported. |
| 3 | Do you have more than one TV or streaming stick? Which one is it (name it in the TV's settings)? | Avoids installing on the wrong device. In our run there were two LG TVs on the network; only the "Master TV" was the target. |
| 4 | Which computer will run this (Windows / Mac / Linux), and is it on the same Wi-Fi as the TV (not guest Wi-Fi, no VPN)? | The computer talks to the TV over the home network. Router "AP isolation"/guest networks block it. |
| 5 | Will someone be at the TV with the remote for about 15 minutes, and can they read a 6-character code off the screen (or take a photo of it)? | Several steps can only be done on the TV itself. |
| 6 | Do you already have an LG developer account, or are you willing to create a free one (email + password)? (LG only) | The Developer Mode app asks you to sign in. |
| 7 | Are you OK with **replacing** the TV's stock YouTube app with the ad-free one? Do you sign in to YouTube with an account, or pay for YouTube Premium? | The ad-free app replaces the stock one (same app ID). Premium users usually don't need any of this. You sign in again inside the new app. |
| 8 | Is the TV yours (not a hotel/rental/landlord's) and are you OK with the warranty question below? | Developer Mode is LG's official feature, but the warranty effect is unconfirmed. |
| 9 | Do you also want an ad-free YouTube on your phone? Android or iPhone? | See Appendix A. |
| 10 | Do you want a short or detailed explanation while we work? | Match the user's preferred style. |

Things Claude should find out by itself rather than asking: whether Node.js is installed, the TV's IP address (read from the Developer Mode screen), and the newest release version.

---

## 3. Route decision (by TV platform)

| TV platform | Route | Status |
|---|---|---|
| **LG webOS (2016+; webOS 3, 4, 5, 6, 22, 23, 24, 25)** | **Route A: youtube-webos via LG Developer Mode** (sections 4-9) | Done and installed on a webOS 25 TV. Ad blocking still to be confirmed by the user. |
| Android TV / Google TV / Fire TV (Sony, TCL, Hisense, Philips, Nvidia Shield, Fire Stick) | SmartTube, free open-source YouTube client with ad blocking: [yuliskov/SmartTube](https://github.com/yuliskov/SmartTube) (active, last push 2026-10-28, latest release 32.68s on 2026-10-20) | **Not tested by us.** Agent: read the project's current README for install steps (sideloading an APK, usually via the "Downloader" app). |
| Samsung (Tizen) | TizenTube: [reisxd/TizenTube](https://github.com/reisxd/TizenTube) (active, v2.1.1, 2026-09-13) | **Not tested by us.** Needs Samsung Developer Mode and a PC install step. Agent: read the README, tell the user honestly if their model/year is unsupported. |
| Roku, Apple TV, Vizio SmartCast | No known ad-block app we could verify | Honest answer: no safe sideload route. Options: YouTube Premium, or plug in a cheap Google TV/Fire TV stick and use SmartTube there (check SmartTube's README that the device is supported). |

For any non-LG route: check the project's repo is not archived, check the latest release date, and read its open issues for your exact TV before installing. Do the same safety rules (no root, reversible, verify downloads).

The rest of this file is **Route A (LG)**.

---

## 4. Route A overview

1. **Phase A (Claude):** identify TV and webOS version, choose the right build.
2. **Phase B (Claude):** install the LG command-line tools on the computer.
3. **Phase C (human at the TV):** install LG's "Developer Mode" app, turn it on, start the Key Server, give Claude the IP and 6-character code, uninstall stock YouTube.
4. **Phase D (Claude):** fetch the key, register the TV, download and verify the ad-free app, install, launch.
5. **Phase E (human):** watch a video that normally has ads; report back.
6. **Phase F:** extend the session periodically; fallback and revert if needed.

What the app is: **youtube-webos** (the NicholasBly fork: [NicholasBly/youtube-webos](https://github.com/NicholasBly/youtube-webos)), which replaces the TV's YouTube app and injects ad-blocking scripts (plus optional SponsorBlock) into the YouTube TV web app. It supports webOS 3, 4, 5, 6, 22, 23, 24, 25. At writing, stable is **0.8.4** (2026-09-29); a 0.8.5 pre-release (2026-10-27) exists; do not use pre-releases unless the stable one fails.

---

## 5. Step-by-step

### Phase A: identify the TV (Claude)

- webOS version (from question 2) selects the build:
  - webOS 3, 4, 5, 6 -> **Legacy** build: `youtube.leanback.v4_<version>_all.ipk`
  - webOS 22, 23, 24, 25 -> **Modern** build: `youtube.leanback.v4_<version>_all_webOS22+.ipk`
- Rough year mapping: webOS 6 = 2021 models; 22 = 2022; 23 = 2023; 24 = 2024; 25 = 2025. If unsure, use the version shown on the TV, not the year.
- **Known problem:** on some 2024 (webOS 24) TVs, e.g. C4/G4 OLEDs, the built-in YouTube app (a newer "Cobalt" version) blocks installing under the same app ID: the install button is dead or ads remain ([issue #149](https://github.com/NicholasBly/youtube-webos/issues/149), open; its suggested fix is repackaging the app under a distinct app ID). If the user has a 2024 set and the stock app will not uninstall or the install does not take, stop, read that issue for the current workaround, and tell the user plainly; do not force it.
- Older sets (webOS 6.0) have reports of playback errors after 5-50 seconds ([#183](https://github.com/NicholasBly/youtube-webos/issues/183), [#185](https://github.com/NicholasBly/youtube-webos/issues/185)), possibly a Google-side block. Warn those users.

### Phase B: install the LG command-line tools (Claude)

```powershell
node --version                      # need Node.js; if missing, install the LTS version (nodejs.org) or use winget/brew/apt
npm install -g @webos-tools/cli     # we used version 3.2.6
ares-setup-device --version         # confirms the tools run
```

(Mac/Linux: same npm command; use `sudo` only if npm requires it.)

### Phase C: human steps on the TV (tell the user these in plain words)

1. On the TV, open the **LG Content Store** and install the app called **Developer Mode**.
2. Open it, sign in with the free LG developer account (create one at LG's developer site if needed).
3. Turn **Dev Mode Status** on. The TV restarts. Open the Developer Mode app again afterward.
4. Turn **Key Server** on. The screen shows a **6-character passphrase (case-sensitive)** and the TV's **IP address**. Give both to Claude (typing them, or a photo of the screen). **Leave this screen open** (see troubleshooting: the key page is only reachable while it is open).
5. **Uninstall the stock YouTube app** (long-press/edit apps on the home bar). The project's README requires this before installing. If it will not uninstall, stop (see Phase A).
6. Note the session timer shown in the app ("Extend Session Time" button). Ours showed 998 hours.

### Phase D: connect, verify, install (Claude)

**D1. Check the TV is reachable** (replace `TV_IP`):

```powershell
Test-NetConnection TV_IP -Port 9922    # must show TcpTestSucceeded True (Developer Mode SSH)
Test-NetConnection TV_IP -Port 9991    # True only while the Key Server screen is open
```
Mac/Linux: `nc -zv TV_IP 9922` and `nc -zv TV_IP 9991`.

**D2. Download the key from the TV** (only works while Key Server is on):

```powershell
New-Item -ItemType Directory -Force "$env:USERPROFILE\.ssh" | Out-Null
Invoke-WebRequest "http://TV_IP:9991/webos_rsa" -OutFile "$env:USERPROFILE\.ssh\webos_rsa"
```
Mac/Linux: `mkdir -p ~/.ssh && curl -o ~/.ssh/webos_rsa http://TV_IP:9991/webos_rsa && chmod 600 ~/.ssh/webos_rsa`

**D3. Register the TV** (pick a device name; we used `master`; the user and port are fixed by LG's Developer Mode: do not change them):

```powershell
ares-setup-device --add mytv -i "username=prisoner" -i "privatekey=webos_rsa" -i "passphrase=PASSPHRASE" -i "host=TV_IP" -i "port=9922"
ares-device --device mytv --system-info     # should print the TV model, webOS version, firmware
```
If a device with that name exists, use `--modify` instead of `--add`. Note: the registered device file may store the passphrase in plain text; treat it as private.

**D4. Download and verify the package.** Always check the newest *stable* release; do not hard-code versions:

```powershell
$h=@{'User-Agent'='x'}
$rel = Invoke-RestMethod "https://api.github.com/repos/NicholasBly/youtube-webos/releases/latest" -Headers $h
$rel.tag_name; $rel.assets | select name, size, digest
```
Choose the Legacy or Modern asset (section 5 Phase A). Download it into its own new folder, then compare:
```powershell
Invoke-WebRequest <asset browser_download_url> -OutFile <folder>\<asset name>
(Get-FileHash <folder>\<asset name> -Algorithm SHA256).Hash   # must equal the digest GitHub shows for that asset
```
Reference values from our run (0.8.4): Legacy `524570a54c391a1f5d40287c9462e9162840193df78e55fcae4c98eaef96eea3` (115,896 bytes); Modern `c56c4e4a1a72dddc6a12f0adb8a81b70dbeecc1dde04a68228a8483b0b590ab4` (89,896 bytes). If your hash does not match, stop.

**D5. Install and launch:**

```powershell
ares-install --device mytv "<folder>\youtube.leanback.v4_<version>_all_webOS22+.ipk"   # expect: Success
ares-launch  --device mytv youtube.leanback.v4                                          # expect: Launched application youtube.leanback.v4
ares-launch  --device mytv --running                                                    # shows the app running
```
(`--help` on each command confirms flag names on your CLI version.)

### Phase E: the human test (Claude must ask for this and wait)

Say: "Open YouTube on the TV (it is now the ad-free version). Play a video that normally shows ads, ideally one with a pre-roll. Check the app's settings for an ad-blocking option and make sure it is on. Tell me: **no ads / still ads / app won't open**." A sponsored tile on the home page can be cleared by refreshing; ad leaks are known ([#166](https://github.com/NicholasBly/youtube-webos/issues/166)).

Record the outcome in the notes file as verified or not.

### Phase F: keep it working

- **Extend the session:** open the Developer Mode app now and then and press **Extend Session Time**. If the session lapses and the TV restarts, LG removes developer-installed apps; an expired session cannot be extended. LG publishes no interval (reports vary from about 50 hours to about 1000), so go by the timer shown in the app. Claude can reinstall in minutes if it does lapse (repeat Phase C steps 3-4 and Phase D).
- **Updates:** YouTube changes can break ad blocking. When it breaks, check the repo's issues/releases and install the newer stable version the same way (D4-D5).
- **Fallback if ads still leak:** [RF1705/youtube-webos-cobalt-adfree](https://github.com/RF1705/youtube-webos-cobalt-adfree). It filters before the page loads. Its stable line is 1.2.5 (2026-09-12); the 2.x line is beta with reported boot-loop and freeze problems, so use stable only, and confirm the user can still revert before installing. Same ares-install method.

---

## 6. Troubleshooting (what we hit, and what usually goes wrong)

| Symptom | Cause / fix |
|---|---|
| Port 9991 "connection refused" | The Key Server screen is not open. Ask the user to open Developer Mode and enable Key Server, then retry. We hit exactly this. |
| Port 9922 closed | Developer Mode is not on, the TV was not restarted, or the computer is on another network (guest Wi-Fi/VPN). |
| `ares-setup-device` accepts it, but connection fails | Passphrase is case-sensitive; re-read it from the screen. The Developer Mode session may have expired (reopen the app). |
| Wrong TV | Confirm with `ares-device --system-info` that the model matches the TV the user means. |
| `ares-install` fails: app already exists / cannot replace | The stock YouTube app is still installed; uninstall it on the TV. If it cannot be replaced (2024 models, see issue #149 in Phase A), stop and read that issue. |
| App opens but videos stop after seconds (older sets) | Known on webOS 6.0 and similar; try the other build only if it matches the TV's webOS, or the fallback project. |
| Ads still appear | Check the in-app ad-block setting; refresh the home page; report the build and symptoms; use the fallback; check open issues ([NicholasBly](https://github.com/NicholasBly/youtube-webos/issues)). |
| `python` not found in Git Bash on Windows | Not needed; use PowerShell. |
| `gh` not installed | Not needed; the GitHub web API via `Invoke-RestMethod` works (60 requests/hour without a token). |
| `ares-*` blocked by PowerShell script policy | Use the `.cmd` shims. |

---

## 7. Known risks (state these to the user plainly)

- **Ad blocking is cat-and-mouse.** Both main projects have broken and been re-fixed after YouTube changes.
- **Developer Mode needs renewing** (section 5, Phase F).
- **Warranty/terms:** Developer Mode is an official LG feature, but the warranty effect is unconfirmed. Blocking ads may breach YouTube's terms of service.
- **No bricking reports** were found in the newest issues of either project. This is absence of evidence, not a guarantee.
- **Stock-app replacement:** the ad-free app replaces the stock one; you re-sign-in; some features differ.
- **Firmware updates** could change behavior (unverified).

---

## 8. What was deliberately NOT done (and why)

- No rooting (brick risk; LG updates can remove root; does not block YouTube in-video ads).
- No DNS/hosts blocking (cannot separate YouTube ads from video; can break the LG store).
- Nothing was installed on the user's second TV.
- No stock-app modifications; nothing outside LG's official Developer Mode.

---

## 9. Undo everything

1. On the TV, uninstall the ad-free YouTube app (long-press it on the home bar, or: `ares-install --device mytv --remove youtube.leanback.v4`).
2. Reinstall **YouTube** from the LG Content Store.
3. Optional: in the Developer Mode app, turn Dev Mode off (the TV restarts). Delete `~/.ssh/webos_rsa`, and remove the device with `ares-setup-device --remove mytv`.

---

## 10. Final message template (Claude -> user, short)

> "The ad-free YouTube app is installed on **[TV name]** and running. I can't see the screen, so please play a video that normally shows ads and tell me: **no ads / still ads / won't open**. Every now and then press **Extend Session Time** in the Developer Mode app (it showed [N] hours left). To undo: delete the app and reinstall YouTube from the LG store."

---

## Appendix A: ad-free YouTube on the phone

- **Android:** [NewPipe](https://github.com/TeamNewPipe/NewPipe), free and open source, no ads, no Google account. Latest at writing: v0.29.1 (2026-08-15): [APK](https://github.com/TeamNewPipe/NewPipe/releases/download/v0.29.1/NewPipe_v0.29.1.apk). Allow "install unknown apps" for the browser when asked. Trade-offs: no sign-in (import subscriptions from a Google export); playback occasionally breaks until NewPipe updates. Undo: uninstall. Check the releases page for a newer version first.
- **Do not use ReVanced:** its patches repository has been blocked on GitHub by a [takedown notice](https://github.com/github/dmca/blob/master/2026/03/2026-03-12-morpheapp.md) since March 2026.
- **iPhone:** we found no verified, safe, reliable option. Safari content blockers do not stop YouTube's in-video ads; sideloaded YouTube mods need extra setup and often break. Honest answer: YouTube Premium.

## Appendix B: sources checked (2026-10-31)

- [NicholasBly/youtube-webos](https://github.com/NicholasBly/youtube-webos) (README: uninstall stock app first; webOS 3-6, 22-25; Developer Mode setup with `prisoner`/port 9922)
- [RF1705/youtube-webos-cobalt-adfree](https://github.com/RF1705/youtube-webos-cobalt-adfree), [webosbrew/youtube-webos](https://github.com/webosbrew/youtube-webos) (original; v0.5.3 April 2026)
- [yuliskov/SmartTube](https://github.com/yuliskov/SmartTube), [reisxd/TizenTube](https://github.com/reisxd/TizenTube) (listed, repo status checked, not run)
- [LG webOS TV CLI installation](https://webostv.developer.lge.com/develop/tools/cli-installation)
- [TeamNewPipe/NewPipe](https://github.com/TeamNewPipe/NewPipe)
