# afk-tab-keepalive
# AFK Tab Keep-Alive (v1.4.1)

A Chrome extension that adds a small floating **At keyboard / Away** pill to your pages. When you set it to
**Away** and your computer goes idle, it cycles through the tabs in that window and clicks near the top of each
page so the pages count you as "active".

> **Disclaimer.** This extension can bypass idle timeouts and session expiries that exist to protect you.
> It is **not recommended for websites that use one-time passwords (OTP / 2FA) or hold sensitive information**
> (banking, payments, email, healthcare, work admin consoles, anything personal or confidential). An unattended,
> logged-in session can be seen or used by anyone who reaches your computer. Use the domain lists (below) to
> keep sensitive sites out of the cycle. You are responsible for following the terms of the sites you use it on.
>
> Away mode **will not start** until you tick **"I understand and accept these risks"** on the settings page.

---

## 1. Install

Requires Chrome 120 or newer (or another Chromium browser with extension support).

1. Unzip `afk-tab-keepalive-v1.4.1.zip`. You get a folder named `afk-tab-keepalive` containing `manifest.json`.
2. Open `chrome://extensions` in Chrome.
3. Turn on **Developer mode** (top-right switch).
4. Click **Load unpacked** and select the `afk-tab-keepalive` folder (the one that contains `manifest.json`).
5. The settings page opens automatically on first install. Read the disclaimer and tick the checkbox.
6. **Refresh any tabs that were already open.** Pages open before install may need a reload to show the pill.
   (The extension tries to inject itself into open http/https tabs, but a refresh guarantees it.)

**Updating:** replace the folder contents, then click the reload icon on the extension card in
`chrome://extensions`, then refresh your open tabs. Old pages keep running the old widget until refreshed.

Don't delete or move the folder afterwards. Chrome loads the extension from it.

---

## 2. Using it

### The pill

| Look | Meaning |
|---|---|
| Green **At keyboard** | The extension is doing nothing. |
| Red **Away · starts after N idle** | Armed. Once the computer is idle for N, cycling begins. |
| Orange **Away · M:SS left** | Cycling is running; counts down to the hard stop. |

- **Click the pill** to switch between At keyboard and Away (press and release quickly).
- **Drag the pill** to move it. The position is remembered and is the same on every tab in that window.
- **Gear icon** opens the settings page.
- **Hover** the pill to see why the last session stopped.
- Clicks within 1.5 seconds of a tab being shown are ignored, so a tab switch can't toggle it by accident.

### One widget per window

Every tab in a browser window shows the same widget with the same state and position. Each window has its own
independent state (unless you choose "All windows together", see the Tabs section). Turning Away on in
window A does not affect window B.

### What happens when you go Away

1. You click the pill, so it turns red (armed).
2. Your system is idle for the configured idle time (or the screen is locked), so cycling starts.
3. In rounds, the extension activates a tab, waits for it to finish loading, pauses briefly, then clicks near the
   top of the page. It moves on to the next tab no sooner than your minimum switch gap.
4. Cycling stops when:
   - you use the mouse/keyboard (see "Stop cycling when"),
   - you click the pill,
   - the hard-stop time is reached.
5. When it stops, it returns you to the tab you were on (if enabled).

Only normal `http://` and `https://` tabs are touched. `chrome://` pages, the Web Store, and the New Tab page
are skipped and don't show the pill.

---

## 3. Settings reference

Open settings by clicking the gear on the pill, clicking the extension's toolbar icon, or via
`chrome://extensions` > Details > Extension options. Click **Save** after changing values.
Changes apply the next time you go Away (the idle time applies immediately). **Reset to defaults** restores
everything below but does **not** clear your disclaimer acceptance.

### Disclaimer
| Setting | Details |
|---|---|
| **I understand and accept these risks** | Required. Away mode is disabled until ticked. Saves immediately when you tick or untick it. Unticking disables Away mode again. |

### Timing
| Setting | Range / default | What it does |
|---|---|---|
| **Idle time before cycling starts** | 15-1800 s, default **120** | How long the system must go without mouse/keyboard input before cycling begins. Uses Chrome's system idle detection, so it covers all apps, not just the browser. A locked screen counts as idle. |
| **Stop after** | 1-120 min, default **30** | Hard stop. Cycling always ends this long after it started. |
| **Minimum seconds between tab switches** | 10-120 s, default **10** | The extension never switches from one tab to the next faster than this. Raise it for slower, gentler cycling. |
| **Seconds between rounds** | 30-240 s, default **30** | How often a new round starts. 30 is the lowest Chrome allows for repeating alarms. Each round visits several tabs (see below). |
| **Max gap between visits to the same tab** | 1-10 min, default **4** | Target for how long any single tab may go unvisited. Keep it under the shortest "are you still there?" check-in of your sites. A warning appears at 5+ minutes. |
| **Pause on tab before clicking** | 0.5-10 s, default **1.5** | After a tab becomes active and finishes loading, how long to wait before clicking. |

**How rounds work.** Each round visits at least enough tabs to cover everything within the max gap, and also
enough to fill the round at your minimum switch gap. Examples with the defaults (10 s gap, 30 s rounds):

- 3 tabs: 3 switches per round, so a switch every 10 s, nonstop.
- 40 tabs, 4-min max gap: covering everything needs 5 tabs per round, so each round makes 5 switches and
  takes about 40-50 s, longer than the 30 s round interval. If a round is still running when the next alarm
  fires, that alarm is skipped. Switches never go faster than the minimum gap, but with many tabs, revisits
  may take longer than your max gap.
- A window with a single tab doesn't switch to itself.

### Click
| Setting | Range / default | What it does |
|---|---|---|
| **Horizontal position** | 0-100 %, default **50** | Where across the page width to click (50 = center). |
| **Distance from top** | 0-500 px, default **10** | How far down from the top of the viewport to click. |

The click is a simulated mouse move + press + release at that point. To avoid triggering anything, if the spot
is on a link, button, input, label, or similar, it clicks the page body instead. It never clicks the pill itself.

### Cancel
| Setting | Options | What it does |
|---|---|---|
| **Stop cycling when** | *I use the mouse/keyboard in the browser* (default) or *The system reports any activity* | **Browser input:** stops only when it sees real mouse movement, clicks, key presses, scrolling, or touches inside a page. Synthetic clicks from the extension itself are ignored, as are events within 1.5 s of a tab switch. This avoids false stops from flaky OS idle signals. **System activity:** stops as soon as Chrome reports the system is active again, including activity in other apps. |

The settings page also shows **Last stop**, the reason and time cycling last ended.

### Tabs
| Setting | Options / default | What it does |
|---|---|---|
| **Which tabs** | *Each window independently* (default) or *All windows together* | **Independently:** each window has its own Away state, widget, and cycle. **Together:** one toggle (or one stop) controls every normal window. |
| **Return to original tab when finished** | on by default | When cycling stops, switches each window back to the tab that was active when cycling started. |
| **Skip pinned tabs** | off by default | Leaves pinned tabs out of the cycle. |
| **Only these domains** | one per line, empty by default | If you list any, **only** matching sites are cycled. Empty means all http/https tabs. |
| **Never touch these domains** | one per line, empty by default | These sites are never switched to or clicked. **Takes priority** over "Only these domains". |

Domain entries match the site and its subdomains: `example.com` also covers `www.example.com` and
`app.example.com`. You can paste a full URL; `https://` and paths are stripped. **Put any bank, email, payment,
or OTP-protected site in "Never touch these domains".**

---

## 4. Good starting setups

- **Default (gentle):** 120 s idle, 30 min stop, 10 s tab gap, 30 s rounds, 4 min max gap.
- **Slower and quieter:** raise "Minimum seconds between tab switches" to 30-60.
- **Only specific sites:** put them in "Only these domains" and leave the other list for anything to exclude.
- **Many windows:** leave "Each window independently" so each can be turned on or off separately.

---

## 5. Troubleshooting

| Problem | Fix |
|---|---|
| No pill on a page | Refresh the page. The pill doesn't appear on `chrome://` pages, the Web Store, or the New Tab page. |
| Clicking the pill just opens settings | You haven't ticked the disclaimer checkbox yet. |
| Pill is on some tabs but stale/old | You reloaded the extension; refresh those tabs. |
| Cycling never starts | Make sure the pill is red (Away), and wait for the full idle time with no mouse/keyboard input. A very short idle time can be limited by Chrome's 15 s minimum. |
| Cycling stops right away | Something is producing real input in the browser (mouse resting on a jittery surface, etc.). Try "The system reports any activity" mode, or check "Last stop" for the reason. |
| Switches slower than expected | Check the minimum switch gap, and the number of tabs. Many tabs make rounds run long. |
| Tabs reload when visited | Chrome discarded them to save memory. The extension waits up to 10 s for them to load before clicking. |
| Nothing happens while the computer sleeps | The extension can't run while the machine is asleep; it only works while the browser is running. |

---

## 6. Files

| File | Purpose |
|---|---|
| `manifest.json` | Extension definition and permissions (storage, idle, alarms, tabs, scripting, all URLs). |
| `background.js` | Per-window state, idle detection, tab cycling, click injection. |
| `content.js` | The floating pill on each page. |
| `options.html` / `options.js` | The settings page and disclaimer. |
