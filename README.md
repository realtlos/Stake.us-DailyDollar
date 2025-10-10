# Stake.us Daily Dollar Auto-Opener (macOS + Windows)

This is a silly project i made that sets up a small automation that **opens the Stake.us Daily Dollar page** in your browser every day at a scheduled time (default = **10:00 PM**).  
👉 It **does not auto-claim**—only opens the page so you can click **Claim** yourself (keeps you within Stake.us ToS).

**Target URL**
```
https://stake.us/promotions/promotion/daily-dollar?tab=dailyBonus&currency=btc&modal=wallet
```

---

## Contents

- `mac/com.stake.daily.plist` — launchd job (macOS)
- `windows/create-task.ps1` — creates a daily Task Scheduler job (Windows)
- `windows/delete-task.ps1` — removes the scheduled task (Windows)

You can keep the same structure in your repo for clarity.

---

## macOS Setup (launchd)

### 1) Install
Save the plist to your LaunchAgents folder:

```bash
mkdir -p ~/Library/LaunchAgents
curl -L -o ~/Library/LaunchAgents/com.stake.daily.plist https://raw.githubusercontent.com/YOURNAME/REPO/main/mac/com.stake.daily.plist
```

*(Or copy the file contents below into `~/Library/LaunchAgents/com.stake.daily.plist`.)*

**`mac/com.stake.daily.plist`**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Label</key>
  <string>com.stake.daily</string>

  <key>ProgramArguments</key>
  <array>
    <string>open</string>
    <string>https://stake.us/promotions/promotion/daily-dollar?tab=dailyBonus&amp;currency=btc&amp;modal=wallet</string>
  </array>

  <key>StartCalendarInterval</key>
  <dict>
    <key>Hour</key>
    <integer>22</integer> <!-- 22 = 10 PM -->
    <key>Minute</key>
    <integer>0</integer>
  </dict>

  <key>RunAtLoad</key>
  <true/>
</dict>
</plist>
```

> 💡 Change the time by editing `<integer>22</integer>` to any **24-hour** hour (e.g., 9 AM = `9`, 6 PM = `18`).

### 2) Load & Test
```bash
launchctl unload ~/Library/LaunchAgents/com.stake.daily.plist 2>/dev/null || true
launchctl load  ~/Library/LaunchAgents/com.stake.daily.plist
launchctl start com.stake.daily   # test now (should open your browser)
```

### 3) Uninstall
```bash
launchctl unload ~/Library/LaunchAgents/com.stake.daily.plist
rm ~/Library/LaunchAgents/com.stake.daily.plist
```

---

## Windows Setup (Task Scheduler)

> Works on Windows 10/11. No admin required if you run in your user context.

### 1) Create the task (PowerShell)
Create `windows/create-task.ps1` with the contents below, or run these lines directly in **PowerShell**:

**`windows/create-task.ps1`**
```powershell
# Config
$TaskName = "Open Stake.us Daily Dollar"
$Url      = "https://stake.us/promotions/promotion/daily-dollar?tab=dailyBonus&currency=btc&modal=wallet"

# Daily at 10:00 PM local time
$Time     = "22:00"

# Use schtasks.exe to avoid credential prompts
$cmd = "schtasks /Create /SC DAILY /ST $Time /TN `"$TaskName`" /TR `"cmd /c start `"`"`" $Url`"`" /F"
Write-Host "Creating task:" $cmd
cmd /c $cmd

Write-Host "Done. You can test with: schtasks /Run /TN `"$TaskName`""
```

Run it:
```powershell
# From the repo root or the windows folder:
powershell -ExecutionPolicy Bypass -File .\windows\create-task.ps1
```

> 💡 Change the time by setting `$Time` to another **24-hour** time (e.g., `"09:30"` for 9:30 AM).

### 2) Test immediately
```powershell
schtasks /Run /TN "Open Stake.us Daily Dollar"
```

### 3) Uninstall
Create `windows/delete-task.ps1` or run:

**`windows/delete-task.ps1`**
```powershell
$TaskName = "Open Stake.us Daily Dollar"
schtasks /Delete /TN "$TaskName" /F
```

Run it:
```powershell
powershell -ExecutionPolicy Bypass -File .\windows\delete-task.ps1
```

---

## Notes & Tips

- **Focus/Popups:** Some browsers may open in the background. Pin the tab or enable “Switch to new tabs immediately” in your browser if desired.
- **macOS XML:** In plist XML, `&` must be written as `&amp;`. The file above already handles that.
- **Timezones:** Both launchd (macOS) and Task Scheduler (Windows) use your **local system time**.
- **Login state:** You must already be logged into Stake.us in your browser for the process to be frictionless each day.

---

## How It Works

- **macOS:** `launchd` runs `open <URL>` at the scheduled time.
- **Windows:** Task Scheduler runs `cmd /c start "" "<URL>"`, which opens your default browser.

---

## Disclaimer

This repo is for **educational purposes**. It only opens a web page; it does not automate clicks or claim bonuses. You’re responsible for complying with Stake.us Terms of Service. If you have any questions feel free to contact @realtlos <3

---
