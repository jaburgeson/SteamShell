# SteamShell (Steam BPM Focus + Window Helper)

## Screenshots

### Control Panel (live in Steam Big Picture)
![SteamShell Control Panel](images/control-panel.png)

## What it is

- An AutoHotkey v2 “console/kiosk helper” that launches Steam Big Picture Mode (BPM),
  keeps the right window in front (Steam vs games vs whitelisted apps), optionally
  recenters/maximizes windows, and provides a Control Panel plus controller-to-mouse/keyboard mode.

## Hotkeys

- Ctrl+Alt+Shift+E : Emergency exit to desktop / restore
- Ctrl+Alt+Shift+R : Reload SteamShellSettings.ini
- Ctrl+Alt+Shift+G : Run Game Foreground Assist (one-shot)
- Ctrl+Alt+Shift+P : Open Control Panel

## Controller chord

- LT + RT + LB + RB + L3 + R3 : Open Control Panel

## Major features

- Steam exit / desktop restore: When Steam closes, SteamShell restores the normal desktop state:
  - Unhides the taskbar
  - Temporarily sets Explorer as the shell (Winlogon Shell)
  - Starts/restarts Explorer so you get a usable desktop
  - Then reverts the shell setting back to SteamShell.exe for next boot

- Steam BPM boot + refocus: Keeps BPM usable as a “shell” after boot, with delay/cooldowns to prevent thrashing.
- Game Foreground Assist: Helps bring “fullscreen-ish” game windows forward when appropriate.
- AlwaysFocus list: A list of EXEs that should win focus over Steam when present.
- Window management: Center windows and maximize only when “large enough,” with an exclusion list.
- Cursor helpers: Optional auto-hide cursor and/or “mouse park” off-screen.
- Logging + live log viewer: Writes a log and provides an in-app live viewer for debugging.
- Hidden Startup Programs: Optional list of extra programs to launch hidden/minimized at boot.
- Controller mouse mode (hold View/Back): Right stick moves mouse, left stick scrolls, D-pad arrows.
  Buttons are configurable (Short/Long) via the Controller Mapping window.

## Explorer “ghost mode” (Game Bar / UWP compatibility)

SteamShell uses an “Explorer ghost mode” approach so you get a console-like experience **without** breaking
Windows features that depend on Explorer/UWP plumbing (notably **Microsoft Game Bar** and other UWP components).

What this means in practice:
- **Explorer is running in the background** to keep required Windows components happy.
- The **taskbar / shell UI is hidden**, so you still get a clean “Steam-first” kiosk feel.
- This helps keep **Game Bar** (Win+G) and other UWP-backed features working, even though you’re not using the
  normal Explorer desktop as your primary shell.

When Steam exits:
- SteamShell performs a full desktop restore (unhide taskbar + start/restart Explorer) so you land on a normal desktop.
- It then resets the Winlogon Shell setting back to SteamShell.exe so the next reboot returns to the SteamShell setup.

## Scoring (how SteamShell decides what to foreground)

SteamShell uses a simple “candidate scoring” approach each tick (and on some one-shot triggers) to decide which
window should be in the foreground. The goal is to avoid random focus-stealing while still recovering from common
“Steam/game/launcher didn’t grab focus” issues.

1) Candidate windows
SteamShell first builds a list of “candidates” from currently visible top-level windows. It then filters out windows
that should never be targeted, such as:
- Cloaked/hidden surfaces (DWM cloaked/UWP-style)
- Tooltips, transient popups, and known “don’t touch” classes
- SteamShell’s own GUI windows (Control Panel, mapping windows, etc.)
- Any EXEs/classes you’ve explicitly excluded in the INI

2) Hard gates (rules that override scoring)
Before scoring, SteamShell applies a few strong rules:
- If a game-like fullscreen window is detected and it’s the right time to help, Game Foreground Assist can win.
- If an AlwaysFocus EXE is running and its window is eligible, it can win over Steam.
- “Hands-Off” mode pauses most focus intervention.

3) Score signals (what increases/decreases score)
Each remaining candidate gets a score based on signals like:
- Fullscreen-ness / size: windows that are large and cover most of the display score higher
- Visibility: minimized/inactive/hidden windows score very low or are removed
- Process identity: Steam BPM, known game processes, and AlwaysFocus processes may add weight
- Recency/cooldowns: recently-activated windows are “cooled down” so SteamShell doesn’t thrash focus
- Optional audio signal: if enabled, processes actively producing audio can get a small boost (useful for launchers/video)

4) Selection + safety checks
After scoring, SteamShell picks the highest scoring eligible window, then applies safety checks before activating:
- It won’t repeatedly activate the same window every tick (cooldowns)
- It won’t fight user actions (Hands-Off, or recent manual focus changes)
- If logging is enabled, it will record WHY a window won (and optionally WHY others were skipped)

5) Why you might see “Skip” in logs
A “Skip” usually means SteamShell saw a candidate, but refused to activate it because:
- It’s excluded (INI exclude lists)
- It’s too small / not “game-like” enough to justify stealing focus
- It’s in cooldown (recently activated)
- Another higher-priority rule won (AlwaysFocus or Steam BPM refocus logic)
- Hands-Off mode is enabled

Tuning tips (safe defaults)
- If SteamShell feels too aggressive: increase tick intervals / cooldowns, reduce “fullscreen” sensitivity, or use Hands-Off.
- If Steam/games don’t come forward reliably: reduce cooldowns slightly or broaden AlwaysFocus to include the launcher EXE.
- Turn on logging temporarily to see the exact scoring reasons, then turn it back down once stable.

## Settings (SteamShellSettings.ini)

- The script auto-creates SteamShellSettings.ini on first run.
- Most settings are documented inline with ';' comments.
- Controller mappings are stored in a separate [ControllerMap] section (edited via the Mapping UI).

## Sample SteamShellSettings.ini (annotated)

```ini
; ==================================================================================================
; SteamShellSettings.ini (auto-generated)
; - Edit values, then reboot OR press Ctrl+Alt+Shift+R to reload.
; - Booleans: true / false
; - If you break things: delete this file and it will be recreated.
; - Comments start with ';' (everything after it is ignored by SteamShell).
; - Control Panel: Ctrl+Alt+Shift+P (session toggles / timed disable / live log viewer).
; ==================================================================================================

[Paths]
SteamPath=C:\Program Files (x86)\Steam\Steam.exe ; Full path to Steam.exe

[BPM]
BpmTitle=Steam Big Picture Mode ; Big Picture window title (must match your BPM title)

[Features]
EnableSplashScreen=true ; Show a black splash overlay during boot
EnableTaskbarHiding=true ; Hide taskbar & tray windows (kiosk feel)
EnableWindowManagement=true ; Center windows; maximize large windows (skips OSK + Steam KB)
EnableAutoHideCursor=true ; Hide cursor after MouseHideDelay inactivity
EnableSteamRefocusMode=true ; Refocus BPM after SteamRefocusDelay when nothing else is visible
EnableGameForegroundAssist=true ; Detect fullscreen-ish games and bring them to front
EnableAlwaysFocus=true ; Allow specific apps (ExeList) to always win focus over Steam
EnableMouseParkOnBoot=true ; Park cursor off-screen to the right at startup
EnableMouseParkEveryRefocus=true ; Park cursor off-screen whenever SteamShell activates a window

[Timing]
SplashScreenDuration=10000 ; Splash duration in milliseconds (0 disables fade)
WindowCheckInterval=500 ; How often to center/maximize new windows (ms)
SteamRefocusPollingInterval=1000 ; How often to run focus checks (ms)
MouseMonitorInterval=250 ; Mouse polling interval (ms)
MouseHideDelay=1000 ; Hide cursor after this many ms without movement
SteamRefocusDelay=3000 ; Wait this many ms before refocusing BPM (when no windows remain)

[Splash]
Mode=Black ; Black | Video
VideoPath= ; Full path to startup video (mp4/wmv). Leave blank for Black.
Mute=false ; Mute video audio
PlayFullDuration=true ; If true, play full video before fading out (safety timeout still applies)
SafetyMaxMs=15000 ; Failsafe max time to keep splash up in Video+PlayFullDuration mode. Set LONGER than your video length.
FadeOutMs=300 ; Fade out duration in ms (0 = no fade)
TopmostGuardMs=500 ; Re-assert topmost while visible (0 disables)

MpvPath= ; Full path to mpv.exe (required when Mode=Video)
ForceSDR=true ; Force MPV to output SDR (prevents HDR/Dolby Vision mode switching)
DebugOverlay=false ; If true, shows the reason video failed on the splash screen

[WindowManagement]
MinWidthPercent=0.20 ; Only maximize windows wider than this % of screen width
ExcludeExeList= ; Optional: Pipe-separated EXEs to skip (e.g. GameBar.exe|YourTool.exe)
ExcludeClassList= ; Optional: Pipe-separated window classes to skip (e.g. UnityWndClass|Chrome_WidgetWin_1)

[AlwaysFocus]
ExeList= ; Pipe-separated EXEs: Example SplitSecond.exe|EADesktop.exe
AlwaysFocusCooldownMs=1000 ; Minimum time between re-activating an AlwaysFocus window

[GameForegroundAssist]
GameCPUThresholdPercent=5.0 ; CPU % that adds a score bonus
FullscreenTolerance=0.98 ; Window must be >= this % of screen size to count as fullscreen-ish
FullscreenPosTolerancePx=2 ; Window position must be within this many px of 0,0 for fullscreen-ish
GameForegroundCooldownMs=1500 ; Minimum time between forced game activations
GameAllowZeroCpuAsCandidate=true ; If CPU samples 0, still allow candidate
GameRequireSteamForeground=true ; Only run game assist when Steam is foreground (safe default)
GameAssistLogEvenWhenSkipped=false ; Log candidates even when skipped due to Steam not foreground

EnableAudioAssist=true ; Add score if the process is outputting audio
ScoreAudioActive=30 ; Score bonus when audio peak is detected

ScoreFullscreen=70 ; Base score when fullscreen-ish
ScoreBorderlessLarge=45 ; Base score when large borderless but not at 0,0
ScoreTitleBonus=10 ; Score bonus if window has a non-trivial title
ScoreCpuAboveThreshold=20 ; Score bonus if CPU >= GameCPUThresholdPercent
ScoreCpuNonZeroBonus=15 ; Score bonus if CPU > 0 but below threshold
GameMinScoreToActivate=60 ; Minimum score required to activate the best candidate

[Logging]
EnableGameScoreLogging=false ; Write scoring/decisions to SteamShell.log
GameLogMode=OFF ; OFF|ACTIVATIONS|TOPN|DIAGNOSTIC
GameLogTopN=3 ; When TOPN/DIAGNOSTIC, log this many top candidates
GameLogIntervalMs=3000 ; Rate limit for TOPN/DIAGNOSTIC logging
GameLogIncludeTitles=true ; Include window titles in the log
GameLogRejectNearCandidates=true ; In DIAGNOSTIC, also log near-fullscreen rejects
GameLogRejectMinAreaPercent=0.85 ; Only log rejects with >= this % of screen area
GameLogRotateMaxKB=256 ; Rotate log when it exceeds this many KB
GameLogRotateBackups=2 ; Keep this many rotated backups

[MousePark]
MouseParkRightOffsetPx=50 ; Park cursor this many px beyond the right edge
MouseParkYPercent=0.50 ; Park cursor at this % of screen height

[StartupPrograms]
Enable=false ; Start additional user-defined programs at boot (hidden)
DelayMs=2000 ; Wait this long after SteamShell starts before launching them
; Define programs as Program1..Program20. Format:
; ProgramN="C:\Path\App.exe" --arg1 --arg2
; Quotes recommended if the path contains spaces. Everything after the exe is treated as arguments.
; Programs are started with Run(..., "Hide") and SteamShell will also attempt to hide any windows they create briefly.
Program1=
Program2=

[Controller]
EnableControllerMouseMode=true ; Enable controller mouse/keyboard mapping (hold View/Back)
ControllerIndex=0 ; 0=first controller
ControllerPollIntervalMs=16 ; Poll rate (ms)
ControllerDeadzone=8000 ; Stick deadzone (0-32767)
ControllerMouseSpeed=100 ; Pixels per poll tick at full deflection
ControllerMouseFastMultiplier=2.5 ; Multiplier while RT is held
ControllerScrollIntervalMs=80 ; Min ms between scroll ticks
ControllerScrollStep=1 ; Wheel notches per scroll tick
ControllerChordHoldMs=500 ; Long-press threshold (ms) for B/X/Y secondary actions

[ControllerMap]
; Controller button mappings (active ONLY while holding View/Back).
; Values can be:
;   Builtin:<Action>   (examples below)
;   Send:<AHK chord>   (example: Send:^!{Tab} for Ctrl+Alt+Tab)
; Optional display text for Send: bindings:
;   <Key>.Display=<Pretty label>

; Default examples (these match the script defaults unless you change them in the Mapping UI):
RB.Short=Builtin:LeftClick
RB.Long=Builtin:None

RT.Short=Builtin:RightClick
RT.Long=Builtin:None

LT.Short=Send:^+o
LT.Short.Display=Ctrl+Shift+O
LT.Long=Send:^+f
LT.Long.Display=Ctrl+Shift+F

LB.Short=Send:^!{Tab}
LB.Short.Display=Ctrl+Alt+Tab
LB.Long=Builtin:TaskManager

A.Short=Builtin:Enter
A.Long=Builtin:None

B.Short=Builtin:Esc
B.Long=Builtin:AltF4

X.Short=Builtin:TabTip
X.Long=Builtin:OSK

Y.Short=Builtin:WinG
Y.Long=Builtin:None

Start.Short=Builtin:None
Start.Long=Builtin:None
L3.Short=Builtin:None
L3.Long=Builtin:None
R3.Short=Builtin:None
R3.Long=Builtin:None

; Builtin actions supported by the script:
;   None, LeftClick, RightClick, Enter, Esc, AltF4, TabTip, OSK, WinG, TaskManager

```

## Notes / gotchas

- When Steam exits, SteamShell performs a desktop restore (taskbar + Explorer shell) and then resets the
  Winlogon Shell value back to SteamShell.exe so the next reboot returns to the console-style setup.
- SafetyMaxMs in [Splash] is a fail-safe maximum time (not padding). It prevents getting stuck if video playback fails.
- If you use video splash, mpv is required (the script is set up for mpv-based playback).
- Controller “Record shortcut” capture will also trigger the real Windows shortcut while recording (known UX limitation).
