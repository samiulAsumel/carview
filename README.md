# Daily Car Balance Tracker

**Mongla Port Authority • Traffic Department**

A zero-dependency, offline-first vehicle tracking system for managing daily car movements across multiple port locations. Built for deployment via `file://` protocol with Firebase cloud sync.

---

## Quick Start

1. Open `index.html` in any modern browser
2. Add daily delivery/receipt data via the **Daily Entry** tab
3. Enter rotation numbers in the **Rot No** column (click any cell to edit)
4. View analytics on the **Charts** tab
5. Generate reports on the **Reports** tab
6. Track inter-location movements on the **Car Transfer** tab
7. Export data as Excel via the top bar (current month or all months)

**No server required.** The app runs entirely from local files and syncs to Firebase when online.

---

## Features

### Daily Entry
- 8-location vehicle tracking (Warehouse-A/B, Yard-1/2, Shed-3/4/5/6)
- Opening/closing balance auto-calculation
- Holiday/weekend day marking
- Real-time summary cards and group totals
- **Rot No column** — inline-editable rotation number per row (up to 30 chars), included in Excel export

### Analytics Dashboard (Charts)
- 7 focused charts: daily receive vs delivery, closing balance, month comparison, location performance, balance trend, net flow
- KPI cards with range-based metrics
- Location summary table
- Period filtering (6m, 12m, all-time)

### Reports
- 13 collapsible report sections (expandable/collapsible on demand)
- Preset filters: This Month, Last Month, 3M, 6M, YTD, 12M, All
- Custom date range selector with previous-period / previous-year comparison
- Executive summary with auction delivery KPIs
- Monthly trends, location rankings, peak days, day-of-week patterns
- Group performance (Warehouse / Yard / Shed)
- Location efficiency ranking
- Daily operations log
- Year-over-year comparison
- Car transfer history
- Auction delivery analytics
- Print-optimized layout

### Car Transfer
- Track inter-location car movements
- Full transfer history with date, source, destination, and quantity

### Data Management
- **LocalStorage** primary storage (works offline)
- **Firebase Realtime Database** optional cloud sync
- Excel export (current month or all months)
- SHA-256 password hashing for admin access
- Role-based user authentication
- Auto-save with dirty-state tracking

### Security
- HTML escaping on all user-supplied data before rendering (XSS protection)
- Strict Content-Security-Policy (allowlisted CDN sources only)
- Subresource Integrity (SRI) hashes on all CDN scripts
- SHA-256 password hashing via the Web Crypto API (no plaintext)
- Minimum 8-character password requirement
- Warning banner when the default admin password is still in use

---

## File Structure

| File | Purpose | Lines |
|---|---|---|
| `index.html` | Application shell + UI | ~1,024 |
| `app.js` | All application logic | ~8,409 |
| `styles.css` | Responsive styling + print CSS | ~4,011 |
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
- **SheetJS (XLSX) 0.18.5** — Excel export
- **Firebase 9.22.0 SDK** — Cloud sync (optional)

All dependencies load from CDNs with Subresource Integrity (SRI) hashes. Password hashing uses the browser's built-in Web Crypto API (no extra library). The app degrades gracefully if CDN access is unavailable.

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
- [x] Strict CSP with SRI hashes on all CDN scripts
- [x] HTML escaping of user data (XSS protection)
- [x] Default-password warning at startup
- [x] Minimum 8-character passwords
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

**1.2.0** — Security hardening: XSS escaping on user data, strict CSP, SRI hashes on CDN scripts, 8-character password minimum, default-password warning (June 2026)

**1.1.0** — Added Rot No column, expanded report sections (June 2026)

---

**© 2026 samiulAsumel. All rights reserved.**
Built for Mongla Port Authority • Traffic Department
