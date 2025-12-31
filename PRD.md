# PRODUCT REQUIREMENTS DOCUMENT (PRD)

**Product:** AI Photo Cleaner & Organizer
**Author:** *Drafted by ChatGPT (based on user direction)*
**Version:** 1.0
**Date:** 2025-12-31

---

## 1 — Executive Summary

Users struggle with overloaded photo libraries — duplicates, blurry images, random screenshots, long videos, and unorganized albums. This product is a **local-first AI photo cleaner** that scans a user's **Google Photos** library, identifies unwanted content, and batches it for **approval-based auto removal**. It also **organizes meaningful images** by detecting faces & food content, prompting the user to **create or name albums**.

Phase 1 = **local-only toolkit (runs on laptop)**
Phase 2 = **cloud-hosted Vercel web UI** with synchronized cleanup jobs

---

## 2 — Personas

| Persona          | Description                                     | Needs                                              |
| ---------------- | ----------------------------------------------- | -------------------------------------------------- |
| Casual User      | Has 20,000+ images, little tech knowledge       | Wants quick cleanup without reviewing each image   |
| Photographer     | Needs high control and low risk of bad deletion | Wants to preview batches and adjust sensitivity    |
| Power User (you) | Developer using Claude Code to iterate          | Needs modular architecture & scriptable components |

---

## 3 — Product Goals

### Primary Goals

* Automatically detect **duplicates, blurry photos, screenshots**, and **long videos**
* Move flagged items → **"Trash Review" holding folder** (requires approval)
* Detect **faces**, batch-group by similarity, and allow **naming albums**
* Identify **food photos** → move into a single **"Food"** album
* Present a **localhost UI** that:
  * Shows batch cleanup suggestions
  * Shows proposed albums
  * Requests approval (not individual photo-clicking)

### Non-Goals

* Auto-delete without user approval
* Automatic human identity labeling (names require user input)
* Image editing or enhancement

---

## 4 — Functional Requirements

### 4.1 — Core Features

| Feature                 | Description                           | Local? | Cloud?                     |
| ----------------------- | ------------------------------------- | ------ | -------------------------- |
| Google OAuth2 Login     | Login + request read/write access     | Yes    | Yes                        |
| Photo Scanning Engine   | Crawl all Google Photos + metadata    | Yes    | Yes                        |
| Duplicate Detection     | Perceptual hashing + similarity check | Yes    | Yes                        |
| Blurry Detection        | Local model; strict threshold default | Yes    | Cloud weight tuning        |
| Food Detection          | Lightweight ML classifier             | Yes    | Yes                        |
| Face Clustering         | Local face embedding model            | Yes    | Yes                        |
| User Approval Dashboard | Localhost webpage (batch-based)       | Yes    | Cloud redesigned UI        |
| SQLite Persistence      | Token storage, settings, logs         | Yes    | Cloud migrates to Postgres |

### 4.2 — User Settings Page

User may control:

* Blur sensitivity (slider 1–100)
* Duplicate threshold (% similarity)
* Long-video cutoff (seconds)
* Whether food detection is enabled
* Whether face album creation is enabled
* Auto-delete toggle after approval

---

## 5 — Technical Architecture

### Phase 1 (Local-Only System Diagram)

```
[User Browser]  ←→  Localhost Web UI (React)
                         ↓
                Internal REST API (Python FastAPI)
                         ↓
              Local AI Inference (ONNX models)
                         ↓
                    SQLite Database
                         ↓
          Google Photos API (read/write + moves)
```

### Phase 2 Cloud Migration

```
Browser → Vercel UI → REST API Gateway → Worker Queue → Google Photos API
                                          ↓
                                     PG Database
                                          ↓
                                Cloud Vision Models
```

---

## 6 — Data Models (SQLite Draft Schema)

```sql
TABLE settings (
  id INTEGER PRIMARY KEY,
  blur_threshold INT,
  duplicate_threshold FLOAT,
  long_video_seconds INT,
  auto_delete_after_approval BOOLEAN
);

TABLE audit_log (
  id INTEGER PRIMARY KEY,
  item_id TEXT,
  item_type TEXT,
  action TEXT,
  timestamp DATETIME DEFAULT CURRENT_TIMESTAMP
);

TABLE oauth_tokens (
  id INTEGER PRIMARY KEY,
  refresh_token TEXT,
  expiry DATETIME
);
```

---

## 7 — UX Wireframes (ASCII)

### Localhost Dashboard – Home

```
+---------------------------------------------------------+
| AI PHOTO CLEANER – DASHBOARD                            |
+---------------------------------------------------------+
| Detected Issues:                                        |
|  [ ] 62 blurry photos        Review >                   |
|  [ ] 17 duplicates           Review >                   |
|  [ ] 4 long videos           Review >                   |
+---------------------------------------------------------+
| Suggested Albums:                                       |
|  - 54 photos (face cluster A)   [Name Album] [Approve]  |
|  - 13 photos (face cluster B)   [Name Album] [Approve]  |
|  - 39 food photos               (Auto → Food Album)     |
+---------------------------------------------------------+
| [Run Scan Again]  [Settings]  [Export Reports]          |
+---------------------------------------------------------+
```

### Batch Review Screen

```
+----------------------------------------------+
| REVIEW – 62 BLURRY PHOTOS                    |
+----------------------------------------------+
| Preview Grid (4x4 thumbnails)                |
| Scroll to view summary stats                 |
+----------------------------------------------+
|  ACTION:   [Approve Removal]   [Cancel]      |
+----------------------------------------------+
```

---

## 8 — Acceptance Criteria (Test Plan)

| Requirement                   | Acceptance Test                                          |
| ----------------------------- | -------------------------------------------------------- |
| Detects blurry images         | Upload sample dataset → ≥95% of blurry photos flagged    |
| Strict blur setting works     | Lower quality photos always flagged                      |
| Duplicate identification      | Photos with ≥90% visual similarity → grouped             |
| Long video threshold UI works | User changes slider → cutoff changes in scan             |
| Face grouping works           | Similar faces clustered with >85% match confidence       |
| Album naming                  | Album isn't created until user enters a string           |
| Auto deletion                 | After approval → photos removed from Google Photos trash |
| Web UI batching               | More than 500 images do NOT require per-photo clicking   |

---

## 9 — Risks & Mitigations

| Risk                               | Mitigation                                                  |
| ---------------------------------- | ----------------------------------------------------------- |
| Google API quota limits            | Batch requests + caching                                    |
| False positives delete good photos | Hold items in "Trash Review" folder before permanent delete |
| Local ML models large (~200MB)     | Provide modular download / optional components              |
| Slow scan performance              | Multithread scanning + async requests                       |

---

## 10 — Timeline (Agile Estimate)

| Phase                         | Deliverables                           | Duration  |
| ----------------------------- | -------------------------------------- | --------- |
| Phase 1 – Local MVP           | Scan + batch preview + SQLite settings | 2–3 weeks |
| Phase 1.5 – Deletion + Albums | Auto delete + naming modal             | 1 week    |
| Phase 2 – Cloud migration     | Vercel UI + remote workers             | 3–5 weeks |

---

# Summary

This PRD defines a **scalable, AI-driven photo cleaning system**, starting local for privacy and later expanding to a hosted SaaS-style experience.

---

## Next Step (Recommended)

Would you like:

**A)** A Claude Code **directory + file scaffold** (React + FastAPI + SQLite + scripts)
**B)** A local-only **Python MVP script**
**C)** A dev roadmap + GitHub issue breakdown
**D)** All of the above

Reply **A, B, C, or D** and I'll generate the materials.
