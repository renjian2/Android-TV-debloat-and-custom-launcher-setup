# CleanYourTV - Skyworth SWTV-24AE-4K Setup Guide

This document records the modifications made to your **Skyworth SWTV-24AE-4K NF** (device `eleonas`, MediaTek `mt5896`, Google TV 14, build `UKN4.250714.064`).

---

## 1. Summary of Changes

### Launcher
- Installed the custom, open-source Apple TV 4K style native Leanback launcher (`com.gathin.tvhome`).
- Configured 4 dock slots:
  1. **SmartTube** (`org.smarttube.stable`)
  2. **Netflix** (`com.netflix.ninja`)
  3. **Prime Video** (`com.amazon.amazonvideo.livingroom`)
  4. **Sony LIV** (`com.sonyliv`)
- Granted location permission for Wi-Fi network name display in the top bar pill.
- Speed-compiled with `cmd package compile -m speed -f com.gathin.tvhome`.
- Successfully set as default Home activity (`cmd package set-home-activity`).

### Debloating
Disabled 23 system telemetry, ad-tracking, bloatware, and unselected services (`pm disable-user --user 0`). Nothing was uninstalled.

#### Telemetry & Tracking
- `com.smartdevice.recordactivation` (first activation tracking)
- `com.smartdevice.aftersale` (silent background updater/installer)
- `com.google.android.feedback` (crash reports)
- `com.android.tv.feedbackconsent` (feedback consent dialogs)

#### Ad Services & Recommendations
- `com.android.adservices.api` (Android ad platform)
- `com.android.ondevicepersonalization.services` (ad targeting)
- `com.android.federatedcompute.services` (ad metrics)
- `com.smartdevice.recommendation` (Coocaa recommendation rows & Sensors Data)
- `com.smartdevice.leanbacklauncher.customizer` (sponsored row injector)

#### Broadcast & System Bloat
- `tv.mopa.ginga` (Brazilian interactive TV middleware)
- `org.obs.mhegservice` (UK MHEG broadcast middleware)
- `com.android.healthconnect.controller`
- `com.android.health.connect.backuprestore`
- `com.Smartdevice.emanual` (on-screen manual)
- `com.smartdevice.dailymanage` (TV Guard cleaner)
- `com.smartdevice.myapp` (Daily Key app manager)
- `com.google.android.play.games`

#### Media & Streaming Apps Disabled (Per User Choice)
- `com.smartdevice.dtv.icast` (Miracast / DLNA screen mirroring)
- `com.google.android.youtube.tv` (Stock YouTube app — user uses SmartTube)
- `com.google.android.youtube.tvmusic` (YouTube Music)
- `in.startv.hotstar` (Disney+ Hotstar)
- `com.community.mbox.tv` (Mbox TV)
- `com.tcl.browser` (TCL Browser / BrowseHere)

#### Stock Home Screen Disabled
- `com.google.android.apps.tv.launcherx` (Stock Google TV Launcher)
- `com.google.android.tungsten.setupwraith` (Setup wizard)

---

## 2. Kept Services & Core Components

The following essential services were strictly preserved:
- **HDMI / AV Input Drivers:** `com.mediatek.tis`
- **Remote & Hot Keys:** `com.mediatek.tv.agent`, `com.smartdevice.global.service`, `com.mediatek.autopair`, `com.smartdevice.dailyshortcut`
- **Voice Assistant:** `com.google.android.katniss`
- **Chromecast Built-in:** `com.google.android.apps.mediashell`
- **System OTA Updater:** `com.mediatek.tv.systemupdater`
- **Keyboards:** `com.google.android.inputmethod.latin`, `org.liskovsoft.androidtv.rukeyboard`
- **Screensaver:** `com.google.android.apps.tv.dreamx`
- **Core Android Framework:** `com.android.systemui`, `com.android.settings`, `com.android.tv.settings`, Play Store & Play Services.
- **Your Selected Apps:** SmartTube, Netflix, Prime Video, Sony LIV, Zee5, Discovery+, Crunchyroll, Netmirror, Playbox TV, MX Player.

---

## 3. How to Customize Dock, Wallpapers & Themes

You can push themes and wallpapers to the launcher at any time over ADB without rebuilding the APK.

### Available Themes
- `themes/aurora`: Dark teal & violet aurora wallpaper with matching ambient background.
- `themes/glass`: Dark navy wallpaper with cinematic frosted-glass dock and app-color glow.
- `themes/scenes`: 14 dark illustrated landscapes that cycle every hour.
- `themes/default`: The built-in pitch-black Apple TV look.

### Applying a Theme
```powershell
$env:TV_HOST = "192.168.1.16"
powershell -ExecutionPolicy Bypass -File tools\theme.ps1 themes\glass
```
Or in Bash / Linux:
```bash
TV_HOST=192.168.1.16 tools/theme.sh themes/glass
```

### Changing Dock Apps & Order
Edit `themes/<themename>/config.json` or create your own custom theme folder with a `config.json`:
```json
{
  "columns": 4,
  "dock_apps": [
    "org.smarttube.stable",
    "com.netflix.ninja",
    "com.amazon.amazonvideo.livingroom",
    "com.sonyliv"
  ]
}
```
And push it using `tools/theme.ps1`.

---

## 4. How to Re-enable Any App or Restore Back to Stock

### Re-enable an Individual Package
If you ever want to bring back an app (for example, Disney+ Hotstar or Stock YouTube):
```powershell
$env:TV_HOST = "192.168.1.16"
python tools/tv.py "pm enable --user 0 in.startv.hotstar"
```

### Full Restore to Stock Google TV
To revert everything back to factory condition and re-enable the Google TV stock launcher:
```powershell
$env:TV_HOST = "192.168.1.16"
powershell -ExecutionPolicy Bypass -File backup\restore-custom.ps1
```
And re-enable stock launcher & setup wizard:
```powershell
python tools/tv.py "pm enable --user 0 com.google.android.apps.tv.launcherx"
python tools/tv.py "pm enable --user 0 com.google.android.tungsten.setupwraith"
python tools/tv.py "cmd package set-home-activity --user 0 com.google.android.apps.tv.launcherx/.home.HomeActivity"
```

---

## 5. Security & Closing ADB (Important Final Step)

Now that setup is complete, close network debugging on your TV to prevent unauthorized network access:
1. On your TV, navigate to **Settings → Device Preferences (or System) → Developer options**.
2. Turn **OFF** **Wireless debugging** and **USB debugging**.
3. Select **Revoke USB debugging authorizations**.
4. Reboot your TV.
5. Check from your computer:
   ```powershell
   python tools/adb-closed.py 192.168.1.16
   ```
   It should confirm that port `5555` refuses connections.
