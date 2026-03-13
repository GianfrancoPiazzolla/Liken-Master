<div align="center">

# ⚡ LikenMaster · EV Cost Intelligence

**A powerful, offline-ready Progressive Web App (PWA) for comparing real electric vehicle charging costs across multiple stations — with live breakeven analysis against internal combustion engine (ICE) vehicles.**

[![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=for-the-badge&logo=html5&logoColor=white)](https://developer.mozilla.org/en-US/docs/Web/HTML)
[![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)](https://developer.mozilla.org/en-US/docs/Web/JavaScript)
[![Chart.js](https://img.shields.io/badge/Chart.js-FF6384?style=for-the-badge&logo=chartdotjs&logoColor=white)](https://www.chartjs.org/)
[![PWA](https://img.shields.io/badge/PWA-5A0FC8?style=for-the-badge&logo=pwa&logoColor=white)](https://web.dev/progressive-web-apps/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)](https://opensource.org/licenses/MIT)

</div>

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Features](#-features)
  - [🚗 Vehicle Profiles](#-vehicle-profiles)
  - [⚡ Charging Stations](#-charging-stations)
  - [📊 Cost Analysis Charts](#-cost-analysis-charts)
  - [📈 Smart Stats Dashboard](#-smart-stats-dashboard)
  - [🎨 UI & Experience](#-ui--experience)
  - [💾 Data Persistence & PWA](#-data-persistence--pwa)
- [Physics & Calculation Engine](#-physics--calculation-engine)
- [Tech Stack](#-tech-stack)
- [Getting Started](#-getting-started)
- [App Structure](#-app-structure)
- [Screenshots / UI Overview](#-screenshots--ui-overview)

---

## 🔍 Overview

**LikenMaster** is a single-file, zero-dependency (except Chart.js) web application that answers a deceptively simple question:

> *"How much does it actually cost me to charge my EV at this station — and is it really cheaper than petrol?"*

It goes beyond naive price-per-kWh comparisons by computing the **effective charging cost** — accounting for the energy spent driving *to* and *from* the charging station, and your battery's real usable charge window (min/max SOC limits). This gives a true, net cost per kWh that lands in your battery and propels you forward.

---

## ✨ Features

---

### 🚗 Vehicle Profiles

LikenMaster supports multiple vehicle profiles so you can compare different cars or driving scenarios side by side.

#### 🆔 Profile Identity
- **Custom name** — name your profile anything (e.g. "My BEV", "Wife's BEV")
- **Vehicle type** — choose between `⚡ BEV` (Battery Electric Vehicle) or `🛢️ ICE` (Internal Combustion Engine)
- **Color tag** — assign a color from a predefined palette for visual identification across all charts and UI elements
- **Profile pill switcher** — a horizontal scrollable pill bar at the top of the Vehicle tab lets you switch between profiles instantly

#### ⚙️ Profile Management (via the 👤 modal)
- **Create** a new profile with name, type (BEV/ICE), and color
- **Edit** an existing profile's name and color
- **Delete** a profile (last profile is protected from deletion)
- **Reorder** profiles with ▲/▼ buttons (order affects chart colors and ranking display)
- **Default profile** — on first launch, a "My Tesla" BEV profile is created automatically

#### 🔋 BEV-Specific Parameters (interactive sliders)
| Parameter | Range | Unit | Description |
|---|---|---|---|
| Battery Capacity | 10 – 200 | kWh | Total nominal battery size |
| Min SOC Limit | 0 – 90 | % | Lower charging/usage boundary |
| Max SOC Limit | 0 – 100 | % | Upper charging limit (e.g. 80% for daily) |
| EV Efficiency | 1 – 15 | Km/kWh | Real-world driving efficiency |

**Derived chips (auto-computed in real time):**
- 🔋 **Usable Capacity** — `capacity × (maxSOC − minSOC) / 100` in kWh
- 📐 **SOC Window** — the active charge window in %
- 🛣️ **Estimated Range** — `usable kWh × Km/kWh` in Km
- ⚡ **Wh/Km** — inverse efficiency for reference

#### 🛢️ ICE Reference Parameters (present on both BEV and ICE profiles)
| Parameter | Range | Unit | Description |
|---|---|---|---|
| Fuel Price | 0.50 – 5.00 | €/L | Current pump price |
| ICE Efficiency | 1 – 40 | Km/L | Combustion vehicle fuel economy |

**Derived chips:**
- 💶 **ICE Cost per Km** — `fuelPrice / kmPerL` in €/Km
- 🎯 **BEV Breakeven price** — the maximum €/kWh you can pay for electricity and still be cheaper than petrol: `(fuelPrice / iceKmPerL) × bevKmPerKwh`

> All sliders show a **live fill gradient** (blue → teal) and update all derived values and charts in real time as you drag.

---

### ⚡ Charging Stations

The **Stations** tab manages all your charging locations. Each station is independent and color-coded automatically.

#### ➕ Adding & Managing Stations
- **Add** a station via the dashed `+ Add Charging Station` button (defaults: 0.40 €/kWh, 10 Km each way)
- **Rename** by typing directly in the station name input field
- **Duplicate** a station (📋 copy icon) — useful to test what-if pricing scenarios
- **Delete** a station (🗑️ trash icon)
- Stations are listed in order; their **color** is assigned from a rotating palette matching the chart series

#### 🔧 Station Parameters (interactive sliders per station)
| Parameter | Range | Unit | Description |
|---|---|---|---|
| Charge Price | 0.00 – 2.00 | €/kWh | Per-kWh price at this station |
| Distance (one-way) | 0 – 100 | Km | Distance from home to station (outward trip) |
| Return Distance | 0 – 100 | Km | Distance from station back home (can differ) |

> Outward and return distances are kept **separate** because a route is often not perfectly reversible (different charging at destination vs. mid-trip charging, for example).

#### 📊 Station Summary (auto-computed footer on each card)
Each station card shows a live 3-column summary row:
- **Effective Cost** — true net €/kWh accounting for energy consumed in transit (see [Physics](#-physics--calculation-engine))
- **vs ICE** — difference vs breakeven in €, colored 🟢 green (cheaper than ICE) or 🔴 red (more expensive)
- **Return SOC** — estimated battery state-of-charge upon arriving home after charging and driving back

---

### 📊 Cost Analysis Charts

The **Graphs** tab contains four switchable chart views, all powered by **Chart.js 4.4**. Charts update live as you adjust any parameter.

> 💡 Charts only work when a **BEV** profile is active. Selecting an ICE profile shows a prompt.

#### 1️⃣ Effective Cost Chart
- **X-axis:** Battery level (% SOC) across the full min→max charge window, sampled at 80 steps
- **Y-axis:** Effective cost in €/kWh (0 – 1.5 range)
- **Per-station curves** — one smooth line per station, color-coded
- **ICE Breakeven line** — a horizontal dashed orange line showing the ceiling price
- **Best-cost fill zone** — a teal-shaded area highlighting the minimum-cost envelope across all visible stations
- **Return SOC markers** — a triangle marker on each station's curve at the exact SOC% you'd return home with, giving a visual sense of "where you'll land"
- **Y range:** Fixed 0–1.5 €/kWh

#### 2️⃣ Savings vs ICE Chart
- Shows **€ saved per 100 Km** versus driving the ICE reference vehicle, plotted against battery SOC
- Positive = BEV wins, negative = ICE is cheaper at that SOC point
- A dashed zero-line marks the breakeven threshold
- **Smart Zoom toggle button** — switches between:
  - 🔍 **Smart Zoom** — clips the Y-axis at the 15th–85th percentile with 30% padding, making curves readable even when low-SOC values spike exponentially
  - ⛶ **Full view** — shows the complete unclipped range
- Area fill under each curve for quick visual ranking

#### 3️⃣ Range Analysis Chart
- **X-axis:** Battery level (% SOC)
- **Y-axis:** Estimated driving range in Km
- Shows a single curve for the active BEV profile based on its efficiency (Km/kWh)
- A horizontal reference line shows **minimum viable range** (50 Km threshold)
- Useful for visualising how much real range different SOC windows actually provide

#### 4️⃣ Station Ranking Table
- Switches from a canvas chart to a **sortable ranking table** view
- Ranks all stations by effective charging cost (lowest = best)
- Rank badge colors: teal (#1), blue (#2), orange (#3), others dimmed
- Columns: Rank, Station Name, Effective Cost (€/kWh), vs ICE delta (€), Return SOC (%)
- vs ICE delta is color-coded green/red inline

#### 🗂️ Interactive Legend
All chart views (except Ranking) display a **clickable legend** below the chart:
- Click any legend item to **toggle** that data series on/off
- Hidden series appear with strikethrough text and reduced opacity
- The best-cost fill zone in the Effective Cost chart updates dynamically when you hide stations
- A small hint text guides the user: *"Tap a legend item to show/hide"*

---

### 📈 Smart Stats Dashboard

The **Stats** tab provides a rich summary dashboard for the active BEV profile.

#### 🔑 Key Metrics (top grid)
- 🏆 **Best Station** — name of the cheapest effective-cost station
- 💶 **Best Effective Cost** — lowest €/kWh from your stations (highlighted chip)
- ⚡ **ICE Breakeven** — the threshold price, always visible for reference
- 🛡️ **vs Breakeven** — how many €/kWh below the breakeven the best station sits

#### ⚡ Station Efficiency Meter
A visual bar chart comparing all stations:
- Each station gets a labeled row with a proportional fill bar and its effective cost value
- The best station bar is colored teal (accent), others are blue

#### 💰 Yearly Savings Estimator
- Input field for **daily Km driven** (range: 1–2000 Km/day), with ±1 step buttons
- Auto-computes:
  - 💵 **Weekly savings** in €
  - 📅 **Monthly savings** in €
  - 📆 **Yearly savings** in € (with a visual progress bar)
  - **Annual Km** driven (dailyKm × 365), displayed for reference
- Uses the best station's savings-per-km vs the ICE reference cost

> ⚠️ The savings section only renders when a BEV profile is active and at least one station beats the ICE breakeven price.

---

### 🎨 UI & Experience

#### 🌗 Theme System
- **Dark mode** (default) — deep navy/charcoal palette (`#070b14` background) with teal accent (`#00f5c4`)
- **Light mode** — soft blue-grey palette with adjusted accent tones
- Toggle via the 🌙 / ☀️ button in the header; theme persists via `localStorage`
- Smooth CSS variable transitions on all color changes (0.35s)

#### 🎞️ Animated Background
- **Grid overlay** — subtle CSS grid lines using `linear-gradient`, 48×48 px cells
- **Floating glow orbs** — two large radial-gradient blobs that slowly float up/down (`floatGlow` keyframe animation, 8s and 10s cycles), creating ambient depth without distraction

#### 📱 Mobile-First Layout
- Fixed header (60px) + fixed bottom nav (76px) + scrollable content area between
- `env(safe-area-inset-bottom)` for iPhone notch/home-bar safe area
- All interactive elements sized for thumb-friendly tapping (38–44px minimum hit targets)
- `viewport-fit=cover` for full-bleed edge-to-edge on modern phones

#### 👆 Swipe Navigation
- Horizontal swipe gestures on the main content area switch between tabs
- Threshold: 60px horizontal travel, with angle verification (`|dx| > |dy| × 1.5`) to avoid accidental triggers on vertical scrolls
- Works in both directions (left = next tab, right = previous tab)
- Swipe dot indicator shows current tab position

#### 🪟 Modal System
- Smooth sheet modals (slide up from bottom on mobile, scale-in on desktop ≥480px)
- Backdrop blur overlay with 70% black dimming
- Touch handle bar on mobile for visual affordance
- ESC / Cancel closes modals cleanly
- Used for: Profile Manager, Profile Edit, About

#### 🍞 Toast Notifications
- Brief non-blocking feedback toasts appear above the nav bar
- Triggered by: profile create/delete, station add/remove/duplicate, etc.
- Auto-dismiss after ~2 seconds with fade animation

#### ♿ Accessibility
- All interactive elements use semantic HTML (`<button>`, `<input>`)
- Focus ring via border/box-shadow on focused inputs
- `title` attributes on icon buttons for tooltip hints
- `role="dialog"` on modal overlays

---

### 💾 Data Persistence & PWA

#### 🗃️ Local State (`localStorage`)
All application state is serialized to `localStorage` under a single key (`likenmaster_state`), including:
- All profiles (id, name, type, color, all numeric parameters)
- All stations (id, name, price, outward, returnDist)
- Active profile ID
- Current theme (dark/light)
- Daily Km value for the savings estimator

State is saved on **every change** (slider move, name edit, station update, etc.) via a `saveState()` function and reloaded via `loadState()` on `init()`.

#### 📲 Progressive Web App (PWA)
- `manifest.json` linked in `<head>` for installability (Add to Home Screen)
- `favicon.ico` referenced for browser tab icon
- **Service Worker** (`sw.js`) registered on page load — enables offline caching and app-shell patterns
- Registration is silent-fail (logs success/failure to console without UI disruption)

---

## 🧮 Physics & Calculation Engine

The core mathematical model is what sets LikenMaster apart from simple price comparators.

### Effective Cost Formula

```
effectiveCost(station, vehicle, soc%) =
    totalCost / netUsableEnergy

  energyAvailable   = capacity × soc / 100                 [kWh in battery now]
  energyToStation   = outwardKm / (Km/kWh)                  [kWh to drive there]
  energyAtStation   = energyAvailable − energyToStation     [kWh left on arrival]
  maxCharge         = capacity × maxSOC/100 − energyAtStation  [kWh added at station]
  totalEnergyPaid   = maxCharge + energyToStation           [kWh paid for including transit]
  totalCost         = totalEnergyPaid × pricePerKwh         [€ spent]
  returnEnergy      = returnKm / (Km/kWh)                   [kWh for return trip]
  netUsableEnergy   = maxCharge − returnEnergy              [kWh that actually benefits you]

  effectiveCost     = totalCost / netUsableEnergy           [€/kWh true cost]
```

> If `energyAtStation < 0` (can't reach station) or `netUsableEnergy ≤ 0` (charge used up entirely on return), the function returns `null` and the station is shown as unreachable.

### Return SOC Formula

```
returnSOC = max(maxSOC − (returnKm / (Km/kWh) / capacity × 100), 0)   [%]
```

### Savings per km Formula

```
savingsPerKm = (fuelPrice / iceKmPerLitre) − (effectiveCost / bevKmPerKwh)   [€/Km]
```

### Yearly Savings Formula

```
yearlySavings = savingsPerKm × dailyKm × 365   [€/year]
```

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| 🏗️ Runtime | Vanilla HTML5 / CSS3 / JavaScript (ES2020+) — **zero build step** |
| 📊 Charts | [Chart.js 4.4.1](https://www.chartjs.org/) via cdnjs CDN |
| 🔤 Fonts | Google Fonts: `Outfit` (display), `JetBrains Mono` (monospace) |
| 💾 Persistence | Browser `localStorage` (JSON serialization) |
| 📲 PWA | Web App Manifest + Service Worker |
| 🎨 Styling | Pure CSS custom properties (design tokens), no framework |
| 📦 Bundler | None — single `index.html` file |

---

## 🚀 Getting Started

Since LikenMaster is a single self-contained HTML file, deployment is trivially simple.

### Option A — Open locally
```bash
# Just open the file in any modern browser
open index.html
```

### Option B — Serve via any static host
```bash
# Python quick server
python3 -m http.server 8080

# Node.js (npx)
npx serve .

# Then navigate to http://localhost:8080
```

### Option C — Deploy to GitHub Pages / Netlify / Vercel
Drop `index.html` (plus `manifest.json`, `sw.js`, `favicon.ico`) into the repo root and enable static hosting. No build pipeline required.

### ⚠️ Requirements
- Any modern browser with ES2020 support (Chrome 85+, Firefox 79+, Safari 14+, Edge 85+)
- JavaScript enabled
- `localStorage` available (standard in all browsers, may be blocked in private mode on some browsers)

---

## 🗂️ App Structure

```
index.html          ← Entire app: HTML structure + CSS + JavaScript
manifest.json       ← PWA manifest (name, icons, display mode, theme color)
sw.js               ← Service Worker for offline caching
favicon.ico         ← Browser tab icon
```

### JavaScript Architecture (all inline in `index.html`)

```
State Management
  └── state {}         Global state object
  └── loadState()      Deserialize from localStorage
  └── saveState()      Serialize to localStorage
  └── init()           Bootstrap on page load

Profile System
  └── renderProfilesBar()      Pill switcher UI
  └── renderVehicleSection()   Slider cards for active profile
  └── updateVehicleChips()     Derived value display
  └── selectProfile()          Switch active profile
  └── createProfile()          New profile from modal
  └── deleteProfile()          Remove profile
  └── moveProfile()            Reorder profiles
  └── openEditProfile()        Edit name/color

Station System
  └── renderStations()         Station card list
  └── updateStationSummary()   Live footer chips per station
  └── addStation()             Default new station
  └── removeStation()          Delete station
  └── duplicateStation()       Clone station

Calculation Engine
  └── calcEffectiveCost()      Core physics formula
  └── calcReturnSOC()          Return battery level
  └── calcSavingsPerKm()       Per-Km saving vs ICE
  └── recomputeSavings()       Stats tab yearly savings

Charts
  └── updateChart()            Dispatcher for active chart type
  └── buildChart()             Chart.js instance manager
  └── renderRankingTable()     HTML table for ranking view
  └── renderLegend()           Interactive legend items
  └── hiddenSeries (Set)       Tracks hidden chart series

Navigation
  └── switchTab()              Tab switcher + animation trigger
  └── swipe gesture listeners  Touch left/right navigation

UI Helpers
  └── openModal() / closeModal()
  └── showToast()
  └── sliderField()            Slider HTML generator
  └── updateSliderFill()       Gradient fill on sliders
  └── applyTheme()             Dark/light toggle
  └── esc()                    HTML escape utility
```

---

## 🖼️ Screenshots / UI Overview

| Tab | Description |
|---|---|
| 🚗 **Vehicle** | Profile switcher pills + interactive sliders for battery/efficiency/ICE params + live derived chips |
| ⚡ **Stations** | Station cards with price/distance sliders + live effective cost / vs-ICE / return SOC summary footer |
| 📊 **Graphs** | 4-tab chart view: Effective Cost curves, Savings vs ICE, Range Analysis, Station Ranking table |
| 📈 **Stats** | Best station highlight, efficiency meter bars, yearly savings estimator with daily Km input |

---

> 💡 **Pro tip:** Install LikenMaster to your phone's home screen via your browser's "Add to Home Screen" option for a full native-app feel with offline support.

---

## 📜 License

Distributed under the **MIT License**. Feel free to use, modify, and share.

---

> 🌱 *Developed for the sustainable mobility community. If you find this tool helpful, please leave a ⭐ on GitHub!*


