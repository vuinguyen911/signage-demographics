# 📊 SignageEye — AI Demographic Analytics for Digital Signage

> **商用展開済み** | **~50 signage devices** deployed | Privacy-first design

[![Python](https://img.shields.io/badge/Python-AI%20Server-blue?logo=python)](https://python.org)
[![Android](https://img.shields.io/badge/Client-Android%20%2F%20Signage-3DDC84?logo=android)](https://developer.android.com)
[![PHP](https://img.shields.io/badge/Backend-PHP-777BB4?logo=php)](https://php.net)
[![MySQL](https://img.shields.io/badge/DB-MySQL-4479A1?logo=mysql)](https://mysql.com)
[![Privacy](https://img.shields.io/badge/Privacy-Image%20Free%20Design-green)](./docs/privacy.md)

---

## 📌 Overview

**SignageEye** is a cloud-based AI demographic analytics platform for digital signage. Existing Android devices or signage hardware with cameras send face images to a cloud AI server for age/gender estimation. Results are stored for marketing analytics — **images are never retained**.

Commercially deployed by **Avix Inc. (Japan)** across **~50 signage devices** for audience measurement and signage optimization.

```
Android Device / Signage Hardware (with camera)
                   ↓
      Person detected in frame
                   ↓
      Capture face → Base64 encode
      Send up to 5 images per 2 seconds
                   ↓  HTTPS
         [SignageEye Cloud AI Server]
         Age & Gender Estimation
                   ↓
         Store result → DELETE image
                   ↓
       [Analytics Dashboard]
       Demographics visualisation
```

---

## 🚀 Key Features

| Feature | Description |
|---|---|
| 📷 Smart Capture | Sends images only when person detected — zero idle traffic |
| 🧠 AI Inference | Age group + gender estimation from face images |
| 🔒 Privacy-First | Images deleted immediately after AI processing |
| 📊 Analytics | Demographic breakdown charts, time-series trends |
| 🎯 Signage Optimization | Data used to tailor displayed content to audience |
| 📡 Scalable | Handles ~50 concurrent devices with burst traffic |
| 🔔 Monitoring | Auto-restart + SNS/email alerts on server failure |

---

## 📐 System Architecture

```
┌────────────────────────────────────────────────┐
│         Android Device / Signage Unit           │
│                                                 │
│   Camera → Face Detection (on-device)           │
│                                                 │
│   IF person detected:                           │
│     capture up to 5 frames / 2 seconds          │
│     encode each frame as Base64                 │
│     POST to cloud API                           │
│                                                 │
│   IF no person:                                 │
│     no data sent (bandwidth optimized)          │
└──────────────────────┬─────────────────────────┘
                       ↓ HTTPS POST (Base64)
┌────────────────────────────────────────────────┐
│            SignageEye Cloud AI Server               │
│                                                 │
│   1. Receive Base64 image payload               │
│   2. Decode image (in memory only)              │
│   3. Face detection + alignment                 │
│   4. Age group estimation (AI model)            │
│   5. Gender estimation (AI model)               │
│   6. Save result to MySQL                       │
│   7. DELETE image from memory immediately       │
│                                                 │
│   ⚡ Average response time: < 500ms             │
└──────────────────────┬─────────────────────────┘
                       ↓
┌────────────────────────────────────────────────┐
│          PHP Backend + MySQL                    │
│                                                 │
│   Demographics data storage                     │
│   Aggregation by hour / day / week              │
│   Device management API                         │
│   JWT Auth + API key management                 │
└──────────────────────┬─────────────────────────┘
                       ↓
┌────────────────────────────────────────────────┐
│         Analytics Dashboard (JavaScript)        │
│                                                 │
│   Real-time viewer count                        │
│   Age group distribution charts                 │
│   Gender ratio trends                           │
│   Peak hour heatmaps                            │
│   Per-device breakdown                          │
│   CSV data export                               │
└────────────────────────────────────────────────┘
```

---

## 🛠 Tech Stack

**Device Client (Android / Signage)**
- Camera integration on existing Android hardware
- On-device face detection trigger (no upload when empty)
- Base64 encoding + HTTPS POST

**AI Server (Python)**
- Face detection + feature extraction pipeline
- Age group classification model
- Gender estimation model
- In-memory processing — zero image persistence
- `Python` + AI/ML libraries

**Backend (PHP)**
- `PHP` — REST API for data ingestion and retrieval
- `MySQL` — demographic analytics storage
- `JWT Auth` — secure device authentication
- `JavaScript` — dashboard frontend

**Infrastructure**
- `Amazon EC2 (Linux)` — cloud server
- `Apache / Nginx` — load-balanced web server
- Auto-restart monitoring + SNS/email on failure

---

## 📊 Traffic Design

One key engineering challenge was **optimizing network traffic** at scale:

```
50 devices × worst case calculation:
  → 1 person detected
  → 5 images per 2 seconds
  → Each image ~50KB Base64
  → Per device: 125KB/s burst traffic
  → 50 devices peak: ~6MB/s

Optimization:
  ✅ Send only when face detected (not on empty frames)
  ✅ Batch 5 frames per 2-second window
  ✅ Compress Base64 payload
  ✅ Server async processing queue
  ✅ Result: ~80% reduction vs naive implementation
```

---

## 🔒 Privacy Architecture

SignageEye was designed with **privacy-by-default** principles:

```
What IS stored:          What is NOT stored:
✅ Age group result      ❌ Face images
✅ Gender result         ❌ Face feature vectors
✅ Timestamp             ❌ Individual tracking IDs
✅ Device ID             ❌ Re-identification data
✅ Aggregate counts      ❌ Personal information
```

This design enables use in **public commercial spaces** (shopping malls, train stations, retail stores) while remaining compliant with Japan's Act on Protection of Personal Information (APPI).

---

## 📈 Analytics Dashboard

```
Real-time View:
  ├── Current viewers by device
  ├── Live age group distribution
  └── Today's running totals

Historical Analysis:
  ├── Hourly heatmap (peak traffic times)
  ├── Daily trends (weekday vs weekend)
  ├── Demographic breakdown per location
  └── Export: CSV / filtered date range
```

---

## 👤 My Role

- Designed the **cloud AI server** architecture (receive → infer → store → delete pipeline)
- Implemented **age/gender estimation** service with privacy-first image handling
- Built **analytics dashboard** with demographic visualizations
- Engineered **traffic optimization** for 50 concurrent devices
- Set up **server monitoring** with auto-restart and SNS/email alerting
- Coordinated with **Avix Japan** for requirements and deployment (Japanese/English)

---

## 📁 Repository Structure

```
SignageEye/
├── ai_server/               # Python AI inference service
│   ├── main.py
│   ├── inference.py         # Age/gender estimation
│   ├── pipeline.py          # Receive → infer → delete
│   └── models/              # AI model weights (not included)
├── backend/                 # PHP API server
│   ├── api/
│   │   ├── ingest.php       # Receive AI results
│   │   └── analytics.php    # Dashboard data API
│   └── models/
├── dashboard/               # Frontend analytics
│   ├── index.html
│   ├── js/
│   │   ├── charts.js        # ECharts visualizations
│   │   └── realtime.js      # Live update logic
│   └── css/
└── docs/
    ├── architecture.drawio  # Full system diagram
    ├── privacy_design.md    # Privacy compliance docs
    ├── traffic_analysis.md  # Network optimization notes
    └── deployment.md        # EC2 setup guide
```

---

## 📄 License

This is a **commercial product** deployed by Avix Inc. across Japan.  
Source code is proprietary. This repository is a **portfolio reference** demonstrating architecture, privacy design, and engineering decisions.

---

*Developed by [Nguyen Van Vui](mailto:vnv1004@gmail.com) — Fullstack Engineer | AI Systems | Japan*
