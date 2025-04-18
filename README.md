# Recipe Manager for Android Tablet: Pre-Development README

## 1. Project Overview

This project delivers a lightweight, "ugly-but-works" Android app running on a Samsung Tab S5e (LineageOS or stock Android) for commercial kitchen staff. It integrates with the MarketMan V3 API to fetch and display recipe definitions ("Preps") and guides cooks through real‑time weight checks via a USB–OTG–to–RS232 scale.

### Goals
- **Snappy, offline‑first access** to recipe data
- **Recipe consistency** via real‑time scale integration
- **KISS methodology**: function over form; minimal learning curve
- **Ease of use**: simple PIN gate, two‑column manage screen, straightforward recipe browser
- **Rapid MVP rollout**: direct APK distribution, manual monthly updates

## 2. Key Architectural Decisions

| Decision Area            | Choice                                                                                           |
|--------------------------|--------------------------------------------------------------------------------------------------|
| Authentication & API     | Retrofit + OkHttp interceptor for MarketMan `/Token` and `GetPreps`; persist `accessToken` in memory |
| Caching & Offline        | Room database (PrepEntity, SubItemEntity) with `isActive` flag; hourly WorkManager sync + manual refresh |
| UI Toolkit               | XML layouts + ViewBinding; MVVM with Kotlin Coroutines + LiveData/Flow                           |
| Scale Integration        | `usb-serial-for-android` via USB‑OTG; background Service polling at 200 ms; LiveData<Double> weight |
| PIN Gate                 | 4‑digit numeric PIN stored (hashed) in SharedPreferences; prompt on launch                       |
| Device Constraints       | Target API 23–33; Samsung Tab S5e (4 GB RAM, 10.5" display); no kiosk mode                       |
| Sync Strategy            | WorkManager: periodic hourly sync; manual one‑off sync with Snackbar feedback                    |
| Deployment & Updates     | Direct APK side‑load for one kitchen; manual monthly updates; defer in‑app update mechanism      |

## 3. High‑Level Architecture

```
┌──────────────────┐      ┌──────────────────┐
│   Android App    │◀────▶│  MarketMan API   │
│  (Kotlin + MVVM) │      └──────────────────┘
│                  │      (GetToken, GetPreps)
│  ┌────────────┐  │
│  │ Room DB    │  │  USB‑OTG
│  │ (Preps)    │  │───▶ RS‑232‑USB Adapter
│  └────────────┘  │      │
│                  │      └──▶ Scale Driver
└──────────────────┘
```

## 4. Component Breakdown

### 4.1 Data Layer
- **Entities**: `PrepEntity`, `SubItemEntity` (with `isActive`, `lastSynced`)
- **DAO**: `loadAllPreps()`, `loadActivePreps()`, `upsertPreps()`, `setActive()`
- **Local Cache**: Room for instant reads; offline‑first UX

### 4.2 Network Layer
- **Retrofit Interface**:
  - `POST /Token` → obtain `accessToken`
  - `POST /buyers/inventory/GetPreps` → fetch full Preps list
- **OkHttp Interceptor**: inject `Authorization: Bearer {token}`

### 4.3 Sync Strategy
- **Automatic**: hourly `PrepSyncWorker` via WorkManager
- **Manual**: pull‑to‑refresh or “Refresh” button enqueues a one‑off worker
- **Upsert Logic**: preserve `isActive` flags, update other fields and `lastSynced`

### 4.4 UI Flow
1. **PIN Entry**: 4‑digit PIN screen on launch
2. **Manage Recipes**: two columns (`All Preps` ⇄ `Device View`), multi‑select + arrow buttons
3. **Recipe Browser**: list of active Preps → detail screen with sub‑items

### 4.5 Scale Integration
- **Library**: `usb-serial-for-android`
- **Service**: background coroutine polling at 200 ms, publish `currentWeight: LiveData<Double>`
- **UI**: on ingredient step, compare `targetWeight` vs. `currentWeight` with color feedback

### 4.6 Deployment
- **Distribution**: direct APK side‑load (internal web folder or file share)
- **Updates**: manual monthly push, no Play Store or MDM

## 5. Next Steps & Backlog

1. **Room & DAO**: scaffold entities, DAO, and database instance
2. **Retrofit Client**: define API interface, token interceptor, and helper
3. **WorkManager**: implement `PrepSyncWorker`, schedule periodic + one‑off syncs
4. **PIN Screen**: build 4‑digit UI, SharedPreferences hashing, retry logic
5. **Manage Recipes UI**: XML layout, Adapters, ViewModel wiring
6. **Recipe Browser UI**: list + detail fragments/activities
7. **Scale Service**: serial‑port polling, LiveData feed, permission handling
8. **Sync Feedback**: Snackbar/Toast on manual sync results, display `lastSynced` timestamp

---

_Ready for pre‑development kickoff!_

