# Tang Shiyan · Portfolio

MTech in Artificial Intelligence Systems candidate at the National University of Singapore. I work on AI application engineering and full-stack development — putting engineering guardrails around what a large model can do, so it ships as a product that actually runs in production, has real users, and stays affordable.

**Contact** ｜ e1554229@u.nus.edu ｜ GitHub [@SereneTang02](https://github.com/SereneTang02)

### 🌐 [中文](README.md)

![GitHub contribution record](images/gh-profile.jpg)

---

## At a glance

| Project | Role | Keywords | Try it |
|---|---|---|---|
| [**WenYan Jinnang**](#1-wenyan-jinnang--an-ai-driven-classical-chinese-vocabulary-mini-program) | Solo build, live in production | Spring Boot · GLM · SSE streaming · three-tier cost control · CI/CD auto-deploy | **Search "文言锦囊" in WeChat**, 668 cumulative users |
| [**Weave**](#2-weave--an-ai-travel-guide-consolidation-app) | Solo project | Multimodal screenshot extraction · four-layer consolidation · anti-hallucination constraints · model switching | 48 unit tests, all passing |
| [**TIMO**](#3-timo--a-showcase-platform-for-a-medical-deep-research-agent) | Group lead (4 people) | Event protocol · SSE live / offline replay · 1,034-question trajectory pipeline | Showcase for Tsinghua AIR's medical agent |
| [**Seamless**](#4-seamless--a-cloud-edge-road-crack-detection-system) | Team lead & algorithm developer | YOLO architecture customisation · cloud-edge split · on-vehicle inference | First-author conference paper · covered by 32 media outlets |

---

## 1. WenYan Jinnang — an AI-driven classical Chinese vocabulary mini-program

**Nov 2025 – present ｜ solo build ｜ live in production, 668 cumulative users**

> **Search "文言锦囊" in WeChat to try it.**

A WeChat mini-program that teaches middle-school students the 150 highest-frequency classical Chinese content words, wrapping generative AI into a closed *learn → test → practise → review* loop with separate student and teacher roles. The server is a self-built Java 17 + Spring Boot 3 + MySQL service (JWT auth, Flyway migrations, Nginx + HTTPS); the AI capabilities are built on Zhipu's GLM API. Product design, front and back end, AI integration, automated testing, CI/CD and production operations — all of it is mine.

### ① Proof it is live

The WeChat mini-program console: as of September 2026 it has served **668** real users, grown organically since launch with no paid acquisition.

![WeChat mini-program console](images/wyjn-console.jpg)

### ② Vocabulary and learning modules

A graded wordbook system: students pick a book by school year, and each word carries its full set of senses, example sentences and sources. Favourites and custom word lists let each student build their own path through the material.

![Home and wordbooks](images/wyjn-home.jpg)

![Word detail](images/wyjn-word.jpg)

### ③ The AI: explanations and "micro classical texts"

This is what makes the product different. On the left is AI explanation, returned in four structured sections — etymology, typical usage, common pitfalls, mnemonic — and streamed over SSE so it renders as it generates. On the right is the original **micro classical text** feature: the student picks 4–8 words to memorise plus a style, and the model streams back a short classical passage containing all of them, each target word annotated with its exact sense in that context — putting isolated definitions back inside a story. After generation, **every target word is verified as actually used**; a passage that fails the check is not accepted.

![AI explanation and micro classical text](images/wyjn-ai.jpg)

**Cost control** is what lets this feature stay switched on. AI calls run through a three-tier scheme: an explanation is **reused globally** (the next person to look up the same word hits the cache), a **per-user snapshot** makes repeat views free and quota-exempt, and each feature has its **own daily allowance** with membership overrides — alongside token metering, cost estimation and a balance dashboard.

### ④ Practice and the mistake loop

Configurable adaptive practice (question count, scope, format). Wrong answers flow into a mistake notebook colour-graded by error count, which drives targeted re-practice.

![Practice](images/wyjn-practice.jpg)

![Mistake notebook](images/wyjn-errors.jpg)

### ⑤ Learning data and motivation

Study statistics, personal reports, an achievement system, daily check-ins and class competition leaderboards — data feedback and light competition to keep students coming back.

![Study statistics](images/wyjn-stats.jpg)

![Achievements and competition](images/wyjn-gamification.jpg)

### ⑥ Teacher side

Teachers get their own workspace: create and manage classes, generate invite codes, assign homework, and inspect both class-level completion and individual student detail — closing the loop from *teacher assigns* to *student completes* to *data returns*.

![Teacher workspace](images/wyjn-teacher.jpg)

### ⑦ Admin console

Beyond the two user-facing roles, I built a separate web admin console to run the thing in production: a metrics dashboard, user management (role and year adjustments, AI quota resets, membership grants), a payments ledger, and AI usage and cost billing. Launching is not the finish line — a product that genuinely runs in production has to be operable and affordable, and this console is where that lives.

![Admin console](images/wyjn-admin.jpg)

### ⑧ Quality and delivery

This is the part I most want to show.

- **122 automated tests** across authentication, AI explanation and streaming, quotas and membership, and learning progress, split into unit tests (`*Test`) and integration tests (`*IT`); the integration tier spins up **a real MySQL 8 container via Testcontainers in CI** and runs real SQL and Flyway migrations rather than mocking the database away
- **Six parallel GitHub Actions jobs**: backend build and test, frontend logic tests, integration tests, Gitleaks secret scanning, Trivy dependency scanning, Semgrep SAST
- Once every gate is green, deployment to Alibaba Cloud ECS is automatic. The deploy script backs up the running jar, swaps and restarts, then polls a health endpoint — and **rolls back automatically** on failure

![CI pipeline](images/wyjn-actions.jpg)

![Layered test suite](images/wyjn-tests.jpg)

![Commit history](images/wyjn-commits.jpg)

![Cloud server status](images/wyjn-ecs.jpg)

> **In short**: WenYan Jinnang demonstrates more than full-stack ability — it is the whole set of engineering judgements it takes to put an AI product in front of real users: what counts as done, how that gets verified automatically, how cost is contained, and how the thing is operated once it is live.

---

## 2. Weave — an AI travel-guide consolidation app

**Sep 2026 – present | Solo project, designed and built independently (AI-assisted coding)**

Weave is for independent travellers who screenshot dozens of guides on Xiaohongshu and Dianping, then can't use them on the ground. You drop the screenshots into a trip; a multimodal model reads out the suggestions; a four-layer consolidation engine de-duplicates them, normalises place names and surfaces conflicts; and the result is attached to an AI-generated itinerary skeleton that keeps improving as you feed it. The guiding idea: consolidation matters more than generation. Built with TypeScript + Expo / React Native (one codebase for iOS and web), Expo API Routes on the server, and a model layer that switches between Claude and Qwen.

![Weave screens: trips, to-do, itinerary, inbox](images/weave-screens.jpg)

### ① A four-layer consolidation engine

Every layer is a pure function — same input, same output:

- **L0 duplicates**: byte fingerprints catch identical screenshots; text similarity ≥ 0.9 catches the same post captured twice
- **L1 place names**: "better to miss a merge than make a wrong one" — exact/alias, similarity ≥ 0.9, and a 0.7–0.9 band the user confirms; chains are split into brand + branch so two branches never collapse into one
- **L2 semantic de-dup**: text recall, auto-merge above 0.85, an LLM judge for the grey zone, and a fallback to text similarity when the model is unavailable
- **L3 conflicts**: first check whether two tips apply under mutually exclusive conditions (six dimensions: time of day, season, companions, transport and more), then give a weighted, explainable leaning (condition fit, number of sources, recency, specificity, with a penalty for likely ads)

### ② Stopping hallucination at the source

Invented opening hours and ticket prices are the worst failure in a travel app, so the model may only quote, not improvise:

- every extracted tip must carry its source text; if under 80% of that source appears in the screenshot's text, the tip is dropped, and numbers may only come from the source
- the itinerary schema simply has no fields for exact times, restaurants or prices, and deterministic rules catch anything that leaks into free text
- a vendor-neutral LLM layer runs Claude and Qwen through the same loop: structured output → zod validation plus rule checks → feed the failure reasons back and retry, with code-level repairs for small issues to save a call

### ③ Engineering

- Domain logic is decoupled from the UI (about 4,700 lines of pure TypeScript) and covered by **48 vitest unit tests**, all passing
- Steps 0–5 of the roadmap shipped in five weeks — about 17,000 lines of TypeScript plus four design documents (about 1,300 lines) recording the thresholds and trade-offs

> **In short**: Weave is less about making the model say more and more about making every sentence traceable to a source.

---

## 3. TIMO — a showcase platform for a medical deep-research agent

**Oct 2026 | Lead of a four-person group | Front end and data pipeline**

TIMO is a medical search agent developed by a research team at Tsinghua AIR (the model and paper are theirs). Given a medical question, it loops through think → search → read → think again and answers with evidence. Our group built the platform that makes this process visible — step-by-step replay and live streaming — for an academic talk at the Beihai forum and the model's open-source release. It went from kickoff to delivery in one week.

![TIMO live research view: conversation, research process, source library](images/timo-live.png)

### ① One event protocol, two data sources

Replay (offline trajectories) and live mode (SSE) would normally need two UIs. I designed a shared event-stream protocol, documented it (377 lines) and set compatibility rules: add fields, never rename; bump the schema version for breaking changes. Both modes then ran on the same React front end without UI changes, and a teammate could build the live back end in parallel.

### ② Trajectory pipeline

A Python pipeline converts raw agent trajectories for **1,034 medical benchmark questions across six benchmarks** into front-end events. Along the way I found that when the model opened several pages at once they were being treated as a single URL (2,417 calls affected) and fixed it. I also used the NCBI and OpenAlex APIs to fill in journal, citation count and DOI for cited papers, rate-limited to 10 requests per second.

### ③ Replay engine and source library

![Source library: papers collected and read during research](images/timo-library.jpg)

I built the replay engine (1–8× speed, jump by round, typewriter effect, compact and real-time pacing, shortcuts and deep links), a searchable source-library modal linked from the research steps, and a bilingual Chinese/English UI. During integration I traced a failure in English mode to the model's safety filter misclassifying medical drug content, where a single failed translation took down the whole page.

> **In short**: as group lead I merged 17 PRs in two days, wrote about 4,500 lines myself, and generated the presentation video programmatically with Remotion.

---

## 4. Seamless — a cloud-edge road crack detection system

**May 2023 – Jun 2025 ｜ team lead & algorithm developer ｜ founder, Yunfan Tech Team at Chongqing University**
National Natural Science Foundation of China project ｜ Chongqing Undergraduate Innovation Training Programme, awarded distinction on completion

Road crack inspection has long depended on manual patrol — slow, and with a high miss rate. Seamless is a cloud-edge platform for automated road quality inspection: an on-vehicle edge box performs real-time crack inference and alerting while the vehicle drives, and a WeChat mini-program lets users upload photos or video for on-demand cloud analysis. Both ends run the same detection model, so the system serves both the immediacy an inspection vehicle needs and the accessibility an ordinary user needs.

### ① Algorithm: a detection architecture tuned for hairline cracks

Hairline cracks are the classic hard case in road inspection. I built the dataset and the model myself: roughly **5,000** raw crack images collected, expanded to over **20,000** samples through affine transforms and other augmentation, spanning different road surfaces, lighting and weather. To attack the miss-rate problem I customised the YOLO detection architecture — adaptive anchor optimisation to match the long, thin geometry of a crack; a redesigned FPN+PAN fusion structure to stop multi-scale features being lost; and GIoU loss in place of the standard bounding-box regression loss to sharpen localisation. The final model reaches **97.8%** overall recognition accuracy under complex environmental interference, and classifies crack type as well.

![Detection output](images/road-detection.jpg)

**Paper**: first author — Shiyan Tang, Zekun Li. *"Road Surface Crack Detection and Classification Based on YOLOv5"*, International Conference on Communication, Internet of Things, Automation and Architecture (ICCITAA 2024).

### ② The cloud-edge system

![On-vehicle edge device](images/road-edge.jpg)

![Mini-program upload and cloud analysis](images/road-miniprogram.jpg)

### ③ Deployment and impact

I led a 30-person cross-functional team through deployment and commercialisation: pilot agreements signed with Chongqing Kenrui Intelligent Building Research Institute and Tianjin Jingran Sensing Technology, multiple third-party test reports validating system reliability, and coverage by **32 mainstream media outlets** including Sina and China.com.

![Media coverage](images/road-media.jpg)

![Completion certificates and partnership agreements](images/road-certificates.jpg)

> **In short**: this is the project I spent the longest on, from my second undergraduate year through to graduation. Technically it was the first time I went inside a deep learning model rather than calling an API — changing the architecture, swapping the loss function, running ablations against the metrics round after round. It left me with a habit: whenever someone claims an "AI capability", I ask first what distribution it was measured on and where its boundaries are. Beyond the technical side, taking a lab prototype through to enterprise pilots with a 30-person team taught me how much unglamorous but necessary work sits between "it runs" and "it is usable".

---

*Every screenshot on this page comes from a real project I worked on.*
