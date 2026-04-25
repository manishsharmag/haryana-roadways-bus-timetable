# 🚌 Haryana Bus Info

> A comprehensive, open bus timetable database and public information platform for Haryana state bus services — built for commuters, researchers, and developers.

🌐 **Live Site:** [haryanabusinfo.in](https://haryanabusinfo.in)

---

## 📖 About

**Haryana Bus Info** is a free, public resource that provides accurate and up-to-date bus schedules for routes operated across Haryana by Haryana Roadways. The platform allows commuters to search bus timings by source station, destination, route number, and operator — all from a fast, mobile-friendly website.

This repository documents the open data pipeline behind the project — including the scraper, data cleaning tools, and CSV processing scripts used to maintain the timetable database.

---

## ✨ Features

- 🔍 **Search by source & destination** — Find direct buses between any two stations in Haryana
- 🕐 **Full timetable data** — Departure times, via stops, route numbers, bus type, and operator
- 🗺️ **Multi-stop route support** — Via stations extracted and indexed for intermediate stop searches
- 📱 **Mobile-friendly** — Optimized for users on smartphones with slow connections
- ⚡ **Fast & lightweight** — Cached results with sitemap support for full Google indexability
- 🔄 **Regularly updated** — Data pipeline actively maintained to reflect schedule changes

---

## 🗂️ Data Pipeline Overview

The timetable data powering this site is collected and cleaned through a multi-stage Python pipeline:

```
Haryana Roadways Website
        │
        ▼
┌─────────────────────┐
│  haryana_scraper.py │  ← Playwright-based multithreaded scraper
│  (MD5 dedup hash)   │     with stable MD5-based deduplication
└─────────────────────┘
        │
        ▼
┌─────────────────────┐
│   json_to_csv.py    │  ← JSON → CSV converter
│                     │     Via stop extraction, zero-padded time normalization
└─────────────────────┘
        │
        ▼
┌─────────────────────┐
│     merge.py        │  ← Deduplication merge script
│                     │     Normalized time comparison to prevent false duplicates
└─────────────────────┘
        │
        ▼
┌─────────────────────────────────┐
│  step1_find_names.py            │  ← Station name audit
│  step2_apply_fixes.py           │     Fuzzy matching via rapidfuzz
│  (3-tier auto-classification)   │     Auto-classify: exact / fuzzy / manual
└─────────────────────────────────┘
        │
        ▼
┌─────────────────────┐
│   MySQL Database    │  ← WordPress custom table
│   (WordPress)       │     Served via haryana-bus-schedule plugin
└─────────────────────┘
        │
        ▼
   haryanabusinfo.in
```

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| Frontend / CMS | WordPress |
| Search Plugin | Custom PHP (`haryana-bus-schedule.php`) |
| Admin Tools | Custom PHP (`bus-insert.php`) |
| Database | MySQL (custom table schema) |
| Scraper | Python + Playwright (multithreaded) |
| Data Cleaning | Python + rapidfuzz |
| Deduplication | MD5-based stable hashing |
| Hosting | Linux VPS |

---

## 📦 Data Schema

Each bus schedule record contains the following fields:

| Field | Description |
|---|---|
| `route_number` | Official Haryana Roadways route number |
| `source` | Origin station name (normalized) |
| `destination` | Final destination station name (normalized) |
| `via` | Intermediate stops (pipe-separated) |
| `departure_time` | Scheduled departure time (HH:MM, 24-hour) |
| `bus_type` | Type of bus (Ordinary, Express, Volvo, etc.) |
| `operator` | Operating depot / division |
| `days_of_operation` | Days the service runs (if applicable) |

---

## 🔧 Key Pipeline Details

### Scraper (`haryana_scraper.py`)
- Built with **Playwright** for handling JavaScript-rendered pages
- Multithreaded for faster collection across route pages
- Uses **MD5-based hashing** for reliable deduplication across runs (avoids Python's `PYTHONHASHSEED` randomization issue with the built-in `hash()`)

### JSON → CSV Converter (`json_to_csv.py`)
- Extracts and normalizes via stop lists from raw scraped JSON
- Applies zero-padded time normalization (`9:5` → `09:05`)
- Handles edge cases: missing fields, malformed times, empty via lists

### Merge & Dedup (`merge.py`)
- Compares records using normalized time strings
- Prevents false duplicates caused by formatting inconsistencies
- Produces a clean, merge-ready CSV for database upload

### Station Name Cleaning (`step1_find_names.py` + `step2_apply_fixes.py`)
- **Step 1:** Audits all unique station names in the dataset; flags potential variants
- **Step 2:** Uses `rapidfuzz` fuzzy matching with a 3-tier classification system:
  - ✅ **Exact match** — auto-applied
  - 🟡 **High-confidence fuzzy match** — auto-applied above threshold
  - 🔴 **Manual review** — flagged for human decision

### Admin Tool (`bus-insert.php`) — V14
- Secure WordPress admin interface for uploading and managing schedule data
- Features: inline row editing, bulk insert, magic wand route assist, Force Cache & Sitemap Refresh button
- XSS-hardened with proper output escaping throughout

---

## 📊 Coverage

- **State:** Haryana, India
- **Operator:** Haryana Roadways (State Transport)
- **Routes:** Thousands of intra-state and inter-state bus routes
- **Stations:** Hundreds of bus stands across all Haryana districts

---

## 🤝 Contributing

This is primarily a solo-maintained project, but contributions are welcome:

- **Data corrections** — Found a wrong timing or station name? Open an issue
- **Missing routes** — Know of routes not in the database? Let us know
- **Bug reports** — If search results look wrong, please file an issue with route details

---

## 📄 License

- **Code & scripts:** MIT License — free to use, adapt, and build on
- **Timetable data:** Sourced from public Haryana Roadways information. Not for commercial resale.

---

## 🙋 Contact & Feedback

For corrections, suggestions, or partnership inquiries:

- 🌐 Website: [haryanabusinfo.in](https://haryanabusinfo.in)
- 📬 Use the contact form on the site

---

## ⭐ Support the Project

If this tool helps you catch your bus on time, consider:
- Sharing the site with fellow commuters
- Starring this repository
- Reporting any incorrect data so we can fix it

---

*Built with ❤️ for Haryana commuters*
