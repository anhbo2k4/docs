# Device Matrix

**Status:** Active  
**Owner:** QA Lead

The set of devices we support and test against. Defines tiers, matrix coverage, and which sprint introduces which device.

## OS support

- **iOS:** 14, 15, 16, 17, 18.
- **Android:** API 26 (8.0) → API 34 (14).

## Tiers

| Tier | Definition |
|---|---|
| **High-end** | Top tier, current generation |
| **Mid-range** | 2–3 year old flagship or current mid-tier |
| **Low-end** | Budget / older device |

## Devices

| Device | OS | Tier | RAM | Notes |
|---|---|---|---|---|
| iPhone 15 | iOS 18 | High | 6 GB | A17 |
| iPhone 14 | iOS 17 | High | 6 GB | |
| iPhone 12 | iOS 17 | Mid | 4 GB | Baseline mid |
| iPhone SE 2 | iOS 17 | Low | 3 GB | Smallest screen, 4 GB RAM unavailable |
| iPad (10th gen) | iPadOS 17 | Mid | 4 GB | Tablet smoke only |
| Pixel 8 | Android 14 | High | 8 GB | Tensor G3 |
| Pixel 6a | Android 14 | Mid | 6 GB | Baseline mid |
| Galaxy A13 | Android 13 | Low | 3 GB | Worst-case Android |
| Galaxy S22 | Android 14 | High | 8 GB | Samsung quirks |
| Xiaomi Redmi Note 12 | Android 14 | Mid | 4 GB | Custom MIUI quirks |

## Test coverage by tier

| Test type | High | Mid | Low |
|---|---|---|---|
| Smoke | yes | yes | yes |
| Full functional | yes | yes | sample |
| Performance NFRs | yes | yes (baseline) | yes (low bound) |
| Whisper benchmark | yes | yes | required |
| Background sync 24h | yes | yes | yes |

The mid tier is the **performance baseline**. NFR targets are defined against mid devices.

## Sprint introduction

| Sprint | New device entering matrix |
|---|---|
| S1 | Pixel 6a, iPhone 12 (mid) |
| S3 | Galaxy A13 (low Android) |
| S4 | iPhone SE 2 (low iOS) |
| S6 | Pixel 8, iPhone 15 (high) |
| S10 | Full matrix smoke + performance soak |

## Network conditions

Tested in S6, S7, S10:

- WiFi (typical office)
- 4G stable
- 3G slow (200 kbps)
- Captive portal
- Airplane mode toggle
- Server timeout
- Connection drop mid-upload

## Cloud test labs

- **Firebase Test Lab** (Android, automated): nightly run on top 5 Android devices.
- **AWS Device Farm** (iOS + Android, automated): nightly run on top 5 iOS + Android.
- **Manual lab** at office: physical devices for tactile test (camera, microphone).

## Pass criteria per device

A device passes pilot gate if:

- Cold start ≤ 2× the NFR target on mid (i.e. low-end may be 2× slower but still must complete).
- Crash-free over 7-day staging.
- Critical flows (login → shift detail → PPE → work result → sync) complete end-to-end.
- No layout breakage at default font size and at 200 % font scale.

## Documentation per device

For each device, `qa/device-reports/<device-slug>.md` holds:

- Manufacturer + model + OS at time of test.
- Date of last full pass.
- Open issues specific to the device.
- Workarounds.
