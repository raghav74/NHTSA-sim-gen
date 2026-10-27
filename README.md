# Simulating Real Crashes from NHTSA Data Using World Models

**Team 19** — Raghav Jadia, Rohith Kumar Senthil Kumar
Northeastern University, Data Science Program

---

## Abstract

Real crash footage paired with ground-truth sensor data is scarce, which limits the training and testing of autonomous-driving perception and safety systems. The U.S. National Highway Traffic Safety Administration (NHTSA) holds thousands of thoroughly investigated real crashes, but only as text narratives, coded data tables, and vehicle sensor recordings, with no video. This project investigates whether a video world model can turn these structured records into realistic crash videos that are consistent with the recorded evidence and useful for downstream tasks such as collision anticipation and damage detection.

Using NVIDIA Cosmos 3 and the NHTSA Crash Investigation Sampling System (CISS), we decode and restructure real crash cases into machine-readable records and characterize how much control three conditioning methods — text prompts, start-frame plus prompt, and start-frame plus prompt plus an action trajectory — give over the generated crash. We find that the conditioning method must match the crash type: fully specified structured prompts can stage collisions between visible vehicles, while a crash involving the camera (ego) vehicle requires an explicit action trajectory, because the ego vehicle is the camera rather than a rendered object. We report clean control of forward motion, road-bounded control of heading, and the remaining gaps — most importantly that NHTSA records contain no start frame, which must therefore be synthesized.

---

## Problem

No existing dataset provides realistic crash video grounded in verified, per-crash physical evidence:

- **nuScenes** — 1,000 driving scenes with 6 cameras, lidar, and radar, but zero collisions.
- **Car Crash Dataset (1,500 videos), DoTA (4,677 videos)** — single-view dashcam clips from the web, with no sensor data or ground truth.
- **NHTSA CISS** — thousands of thoroughly investigated real crashes, but only as narratives, coded tables, and EDR sensor traces, with no video.

**Goal:** convert structured NHTSA crash records into synthetic crash videos that are consistent with the known evidence and controllable enough to reconstruct a specific crash.

---

## Data

- **Source:** NHTSA Crash Investigation Sampling System (CISS), data years 2017–2024.
- **Why CISS:** the only NHTSA source pairing, for the same crash, an investigator narrative, coded crash fields, scene photos/diagrams, EDR sensor data (when available), and injury information.
- **Scope:** crashes involving exactly two light vehicles.
- **EDR coverage (2024, two light vehicles):** of 2,225 cases, 1,026 have pre-crash EDR for one vehicle and 451 for both — a key constraint that motivates EDR imputation.

> CISS data is public and must be downloaded from NHTSA. This repository does **not** redistribute NHTSA data.

---

## Approach

1. **Data pipeline** — scrape NHTSA public data, decode CISS `sas7bdat` tables via their embedded data dictionaries to CSV, extract EDR tables from the PDF reports (the `.cdrx` format is proprietary), and assemble one standardized JSON record per case.
2. **Generation** — NVIDIA Cosmos 3 (Nano / Super). Conditioning is studied at three levels:
   - **Prompt (+ start frame):** text or structured-JSON prompt; the start frame fixes the scene, vehicle positions, and viewpoint.
   - **Prompt + start frame + action:** a 9D per-frame action trajectory (3 translation + 6D rotation, following Zhou et al.) drives the ego vehicle. The 9D format was verified via Cosmos 3 inverse dynamics.
3. **Evaluation (current stage)** — qualitative controllability characterization and numerical inspection of action vectors; downstream evaluation planned for the next stage.

---

## Key Findings

- Fully specified **structured prompts** overcome the model's collision-avoidance tendency and stage collisions between visible (non-ego) vehicles; Super and an elevated traffic-camera view give the most coherent results.
- The **ego vehicle cannot be crashed through text alone** — it is the camera, not a rendered object — so an **action trajectory** is required for ego-involved crashes.
- **Forward control is clean; heading control is road-bounded** and degrades under strong steering commands.
- **NHTSA records contain no start frame**, so the start frame required by the controllable methods must be synthesized — an open component of the pipeline.

---

## Status

Current stage: CISS decoding, per-case record assembly, exploratory analysis, and controllability experiments across the three conditioning methods.

**Next:** per-case prompt generator; trajectory model and EDR-imputation model from NHTSA data; start-frame synthesis; full-setting video generation with consistency checks; and a collision-anticipation model evaluated on real dashcam data (Nexar) with and without the synthetic videos.

---

## Acknowledgements / Stakeholders

Course project for the Northeastern University Data Science Program. Intended beneficiaries include autonomous-driving and automotive-safety researchers.

## References

- NVIDIA. *Cosmos 3: Omnimodal World Models for Physical AI.* 2026.
- Y. Zhou et al. *On the Continuity of Rotation Representations in Neural Networks.* CVPR, 2019.
- U.S. NHTSA. *Crash Investigation Sampling System (CISS).*
- H. Caesar et al. *nuScenes.* CVPR, 2020.
- W. Bao et al. *Car Crash Dataset.* ACM MM, 2020.
- Y. Yao et al. *DoTA.* IEEE TPAMI, 2022.
