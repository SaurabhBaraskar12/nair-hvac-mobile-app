# N.Air HVAC Solutions — Mobile App (Capacitor wrapper)

A thin WebView app that loads the Frappe site running on your PC.

- **App name:** N.Air HVAC Solutions
- **App ID:** `com.nair.hvac`
- **Loads URL:** `http://192.168.31.236:8000`  (your PC's Wi-Fi LAN IP)
- **Project:** `~/n_air_mobile_app`  (Android project in `android/`)
- Icons = logo swirl; splash = full logo (light + dark). App name + config already set.

Everything is scaffolded. Only the final `.apk` **compile** is left — that needs the
Android toolchain (JDK + Android SDK), which is NOT for Python/Frappe, only for building
Android binaries.

---

## STEP A — Networking (REQUIRED, do this once) — run on Windows as Administrator

Frappe runs inside WSL2. A phone on your Wi-Fi cannot reach WSL2 directly, so forward the
port from Windows → WSL and open the firewall.

1. Find the current WSL IP (it can change after a reboot):
```bash
wsl hostname -I
```
(right now it is `172.20.55.186` — use whatever the command prints)

2. In **Admin PowerShell** (replace the IP if step 1 changed it):
```powershell
netsh interface portproxy add v4tov4 listenport=8000 listenaddress=0.0.0.0 connectport=8000 connectaddress=172.20.55.186
New-NetFirewallRule -DisplayName "Frappe 8000" -Direction Inbound -Action Allow -Protocol TCP -LocalPort 8000
```

3. Make Frappe answer requests that arrive by IP (not just `mysite.local`):
```bash
wsl bash -lc 'cd ~/frappe-bench && ~/.local/bin/bench use mysite.local'
```

4. Keep the dev server running (bind is already 0.0.0.0):
```bash
wsl bash -lc 'cd ~/frappe-bench && export PATH=$HOME/.nvm/versions/node/v18.20.8/bin:$HOME/.local/bin:$PATH && bench start'
```

5. Test from the **phone's browser** (same Wi-Fi): open `http://192.168.31.236:8000` —
you should see the login page. If yes, the app will work.

> If your PC's Wi-Fi IP ever changes, update `server.url` in `capacitor.config.json`,
> then run `npx cap sync android`. If the WSL IP changes, redo the `portproxy` (delete old:
> `netsh interface portproxy reset`).

---

## STEP B — Build the APK (pick ONE)

### Option 1 — Android Studio (easiest, GUI)
1. Install **Android Studio** (bundles JDK + Android SDK): https://developer.android.com/studio
2. Open the project folder: `\\wsl.localhost\Ubuntu\home\hp\n_air_mobile_app\android`
   (or copy `~/n_air_mobile_app` to Windows first — recommended for speed).
3. Let Gradle sync, then: **Build → Build Bundle(s)/APK(s) → Build APK(s)**.
4. Output: `android/app/build/outputs/apk/debug/app-debug.apk` — copy to your phone and install
   (enable "Install unknown apps").

### Option 2 — Command line in WSL (no GUI)
Install once, then build:
```bash
# JDK 17
sudo apt update && sudo apt install -y openjdk-17-jdk
# Android cmdline-tools + SDK (see https://developer.android.com/tools) then:
cd ~/n_air_mobile_app/android && ./gradlew assembleDebug
# APK: app/build/outputs/apk/debug/app-debug.apk
```

### Option 3 — Cloud build (no local install)
Push `~/n_air_mobile_app` to GitHub and use **GitHub Actions** or **Codemagic** to run
`./gradlew assembleDebug`; download the APK artifact. Ask and I'll add a ready workflow file.

---

## STEP C — iOS (Mac only)
```bash
npm install @capacitor/ios && npx cap add ios && npx cap sync ios
```
The iOS project can be added on any OS, **but the actual `.ipa` can only be built on macOS
with Xcode** (or a Mac cloud CI like Codemagic). It cannot be built on Windows/WSL.

---

## Notes
- This points to a **dev server on your LAN** — only works while your PC + `bench start` are
  running and the phone is on the same Wi-Fi. For real distribution, host Frappe on a server
  with a domain + HTTPS and set that as `server.url`.
- To change app behavior later: edit `capacitor.config.json`, then `npx cap sync android`.
