# Daily Car Balance Tracker

**Mongla Port Authority • Traffic Department**

A zero-dependency, offline-first vehicle tracking system for managing daily car movements across multiple port locations. Built for deployment via `file://` protocol with Firebase cloud sync.

---

## Quick Start

1. Open `index.html` in any modern browser
2. Add daily delivery/receipt data via the Daily Entry page
3. View analytics on the Dashboard tab
4. Generate reports on the Report tab
5. Export data as Excel via the top bar

**No server required.** The app runs entirely from local files and syncs to Firebase when online.

---

## Features

### Daily Entry
- 8-location vehicle tracking (Warehouse-A/B, Yard-1/2, Shed-3/4/5/6)
- Opening/closing balance auto-calculation
- Holiday/weekend day marking
- Real-time summary cards and group totals

### Analytics Dashboard
- 7 focused charts: daily receive vs delivery, closing balance, month comparison, location performance, balance trend, net flow
- KPI cards with range-based metrics
- Location summary table
- Period filtering (6m, 12m, all-time)

### Reports
- 12 collapsible report sections (expandable/collapsible on demand)
- Custom date range selector
- Executive summary with auction delivery KPIs
- Monthly trends, location rankings, peak days, day-of-week patterns
- Year-over-year comparison
- Car transfer history
- Auction delivery analytics
- Print-optimized layout

### Data Management
- **LocalStorage** primary storage (works offline)
- **Firebase Realtime Database** optional cloud sync
- Excel export (current month or all months)
- SHA-256 password hashing for admin access
- Role-based user authentication
- Auto-save with dirty-state tracking

---

## File Structure

| File | Purpose | Size |
|---|---|---|
| `index.html` | Application shell + UI | ~50KB |
| `app.js` | All application logic | ~320KB |
| `styles.css` | Responsive styling + print CSS | ~140KB |
| `service-worker.js` | PWA offline caching | ~3KB |
| `manifest.json` | Web app manifest | ~1KB |
| `car.png` | App icon/logo | Image |

---

## Architecture

```
┌─────────────────────────────────────────────┐
│                  Browser                     │
│  ┌──────────┐  ┌──────────┐  ┌───────────┐  │
│  │ index    │  │ styles   │  │ app.js    │  │
│  │ .html    │  │ .css     │  │ (logic)   │  │
│  └──────────┘  └──────────┘  └───────────┘  │
│         │                        │           │
│         ▼                        ▼           │
│  ┌──────────────┐  ┌──────────────────┐     │
│  │ Service      │  │ LocalStorage     │     │
│  │ Worker (SW)  │  │ (primary store)  │     │
│  └──────────────┘  └──────────────────┘     │
│                              │               │
│                              ▼ (when online) │
│                    ┌──────────────────┐      │
│                    │ Firebase RTDB    │      │
│                    │ (cloud sync)     │      │
│                    └──────────────────┘      │
└─────────────────────────────────────────────┘
```

### External Dependencies (CDN)
- **Chart.js 4.4.0** — Data visualization
- **SheetJS (XLSX)** — Excel export
- **jsSHA** — SHA-256 hashing for passwords
- **Firebase 9 SDK** — Cloud sync (optional)

All dependencies load from CDNs. The app degrades gracefully if CDN access is unavailable.

---

## Configuration

### Firebase Setup (Optional)

Edit the Firebase config block in `index.html`:

```javascript
const firebaseConfig = {
  apiKey: "...",
  authDomain: "...",
  databaseURL: "https://YOUR-PROJECT.firebaseio.com",
  projectId: "...",
  storageBucket: "...",
  messagingSenderId: "...",
  appId: "..."
};
```

Without Firebase, all data stays in `localStorage`.

### Location Configuration

Edit `LOCS` array and `LOC_CFG` object in `app.js` to customize location names and colors.

---

## Production Checklist

- [x] Error boundaries on all render functions
- [x] Null guards on DOM element lookups
- [x] Database validation on load (corruption detection)
- [x] Memory leak prevention (chart destruction)
- [x] `prefers-reduced-motion` accessibility support
- [x] User-facing error overlay with recovery
- [x] `file://` protocol compatibility (CSP configured)
- [x] Print-optimized CSS
- [x] Responsive design (mobile-first breakpoints)
- [x] Service worker for offline caching
- [x] SHA-256 password hashing (no plaintext)
- [x] Session timeout for authenticated users
- [x] Auto-save with dirty-state tracking
- [x] Input sanitization on all data entry

---

## Browser Support

| Browser | Support |
|---|---|
| Chrome 90+ | ✅ Full |
| Firefox 90+ | ✅ Full |
| Safari 14+ | ✅ Full |
| Edge 90+ | ✅ Full |
| Mobile Chrome | ✅ Full |
| Mobile Safari | ✅ Full |

---

## Deployment

### Option 1: Direct File Access
Simply open `index.html` in a browser. No server needed.

### Option 2: Static Hosting
Deploy to any static host:
- Firebase Hosting
- GitHub Pages
- Netlify
- Vercel
- Nginx/Apache

### Option 3: Firebase Hosting
```bash
firebase init hosting
firebase deploy
```

---

## Known Limitations

- PWA install prompt requires HTTPS (not available on `file://`)
- Service worker requires a server context (not available on `file://`)
- Firebase sync requires internet connection
- Export requires SheetJS CDN to load

---

## Version

**1.0.0** — Production release (May 2026)

---

**© 2026 samiulAsumel. All rights reserved.**
Built for Mongla Port Authority • Traffic Department
