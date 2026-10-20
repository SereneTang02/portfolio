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
| [**Weave**](#2-weave--a-multi-agent-travel-guide-consolidation-app) | Solo project | Multi-agent orchestration · in-house agent harness · map / weather / search tools · anti-hallucination checks | 48 unit tests, all passing · offline regression eval |
| [**TIMO**](#3-timo--an-open-lightweight-model-for-medical-deep-research) | Engineering lead, Tsinghua AIR TIMO team | Medical deep research · event protocol · SSE live / offline replay · trajectory pipeline | 27B: 48.5% / 4B: 40.4% · 4B and 27B open-sourced |
| [**Seamless**](#4-seamless--a-cloud-edge-road-crack-detection-system) | Team lead & algorithm developer | YOLO architecture customisation · cloud-edge split · on-vehicle inference | First-author conference paper · covered by 32 media outlets |
| [**Lumie**](#5-lumie--a-live2d-ai-companion-app) | AI engineering & mobile, 2-person team | Knowledge-graph memory · graph-augmented retrieval · prompt governance & evaluation · Live2D | **[lumie.love](https://lumie.love)** |

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

## 2. Weave — a multi-agent travel-guide consolidation app

**Sep 2026 – present | Solo project, designed and built independently**

### ① Product and what makes it different

People screenshot dozens of guides on Xiaohongshu and then can't use them on the ground: the tips repeat, contradict each other, spell place names five different ways, and mix in outdated information and ads. In Weave you drop those screenshots into a trip; the system reads them, de-duplicates, flags conflicts and attaches everything to an AI-generated itinerary skeleton that keeps improving as you feed it.

The defining rule is **"the model may quote, not invent"**: every tip traces back to a screenshot or web source, and the details models most like to make up — exact times, ticket prices — can only come from the source text or a live lookup. Built with TypeScript + Expo / React Native (one codebase for iOS and web), with Claude and Qwen (including vision models) routed per task.

![Weave screens: trips, to-do, itinerary, inbox](images/weave-screens.jpg)

### ② Multi-agent system

An orchestrator and five specialist agents share the work:

| Agent | Responsibility |
|---|---|
| **Orchestrator** | Works out whether the input is new screenshots, a one-line note or a change request such as "make day three easier"; splits it into sub-tasks, dispatches them in dependency order and assembles a proposal for the user |
| **Reader** | Uses a vision model to extract tips from screenshots, each with its source text; long screenshots are read in segments |
| **Places** | Normalises place names: exact/alias, similarity ≥ 0.9, and a 0.7–0.9 band the user confirms; chains are split into brand + branch — better to miss a merge than make a wrong one |
| **Consolidator** | Semantic de-dup (auto-merge above 0.85, LLM judge for the grey zone) and conflict detection across six applicability dimensions with a weighted, explainable leaning |
| **Itinerary** | Generates the itinerary skeleton and turns natural-language edits into structured change proposals |
| **Verifier** | Final gate: drops anything whose source coverage is below 0.8, allows numbers only from the source, and blocks schema-forbidden fields |

A full run: screenshots → reader agents in parallel → places → consolidator against the existing itinerary → verifier → the orchestrator reports "N new, M mergeable, K conflicting", and **nothing is written until the user taps "Use"**. The model suggests; the user decides.

### ③ Agent harness

A language model only reads and writes text; the harness is the runtime around it that offers tools, executes the calls the model makes and feeds the results back until it answers. Weave's harness is built in-house:

- **Tool registry**: map / POI lookup, weather and web search, plus internal tools for reading screenshots, matching places, merge judging and itinerary reads/writes, all with zod-defined inputs and outputs; agents can only call registered tools, arguments are validated first, and external APIs get timeouts, caching and rate limits
- **Execution loop with budgets**: each agent has step, token and time limits and wraps up with what it has when it runs out
- **Structured output and feedback retries**: output goes through zod and deterministic rule checks; failures are fed back with their reasons, and small issues are repaired in code to save a call
- **Fallback**: if any agent or tool fails, the flow falls back to the rule-based path (for example text-similarity merging) and keeps going
- **Model routing**: vision models for reading, text models for judging and proposals; Claude and Qwen switch per task
- **End-to-end tracing**: every step records the agent, tool, prompt version, model, latency and tokens, so any run can be replayed

### ④ Engineering

- Domain logic is decoupled from the UI as pure functions, covered by **48 unit tests, all passing**
- **Offline regression evaluation** on a fixed sample set: after changing a prompt or model, re-run and compare de-dup, conflict detection and source coverage
- About 17,000 lines of TypeScript and four design documents recording every threshold and trade-off

> **In short**: Weave is less about making the model say more and more about making every sentence traceable. The agents split the work cleanly; the harness keeps every model call controlled, recoverable and replayable.

---

## 3. TIMO — an open, lightweight model for medical deep research

**TIMO project team, Institute for AI Industry Research (AIR), Tsinghua University | Engineering lead | Ongoing involvement**

### ① What the project is

Uneven computing resources and strict data-privacy rules make it hard to deploy large models in hospitals. TIMO (TIMO-OSS-Search-Preview), developed at Tsinghua AIR, aims to give **privately deployable lightweight models** real medical deep-research ability. Given a complex medical question, it works like a researcher — think → search → read guidelines and papers → reason again — and returns a verifiable, evidence-based answer.

The project has three strands: evidence-driven training-data synthesis (knowledge-graph question generation, distillation of complex-question trajectories, multi-stage rejection sampling), co-evolution of the model with its retrieval harness, and a showcase platform for academic talks and the open-source release. The model ships in 4B and 27B sizes.

![TIMO live research view: conversation, research process, source library](images/timo-live.png)

### ② What the engineering group does, and what I do

The team has an algorithm group (data synthesis, training, the paper) and an engineering group, which turns the model's research ability into **systems that can be demonstrated and delivered**: the demo platform, the trajectory data pipeline and the hook-up to live inference, kept in step with every new model and harness version.

I lead the engineering group — architecture, task split, code review and liaison with the algorithm group. Specifically:

- **One event protocol, two data sources**: replay (offline trajectories) and live mode (SSE) would normally need two UIs. I designed and documented a shared event-stream protocol with compatibility rules (add fields, never rename; bump the schema version for breaking changes), so both modes run on the same React front end and the live back end could be built in parallel.
- **Trajectory pipeline**: a Python pipeline converts raw agent trajectories for 1,034 questions across six medical benchmarks into front-end events. I found and fixed a bug where opening several pages at once was treated as one URL (2,417 calls affected), and used the NCBI and OpenAlex APIs to fill in journal, citation count and DOI, rate-limited to 10 requests per second.
- **Replay engine and source library**: 1–8× speed, jump by round, typewriter effect, compact and real-time pacing, shortcuts and deep links; a searchable source library; a bilingual UI. During integration I traced a failure in English mode to the model's safety filter misclassifying medical drug content, where one failed translation took down the whole page.
- The group merged 17 PRs; I wrote about 4,500 lines myself.

![Source library: papers collected and read during research](images/timo-library.jpg)

### ③ Results

Reached 48.5% (27B) and 40.4% (4B) overall accuracy on 1,034 hard questions from six medical benchmarks, ahead of closed-book Claude-Opus-5 (42.9%) and GPT-6-astra (36.1%) respectively and of comparable open medical models; gained 4.55 percentage points over the first version by co-evolving the harness with the model; selected about 3,500 high-quality training trajectories from more than 220,000 candidate questions; presented the work as an oral talk and poster at the 2nd Beijing-Tsinghua Health AI Summit (2026), took part in CCL 2026 (the 25th China National Conference on Computational Linguistics), open-sourced the 4B and 27B models on GitHub and Hugging Face, and delivered the showcase platform for the live demo and release.

> Model training and the paper are the algorithm group's work, and the accuracy figures are team results. This page shows interface screenshots only — no code or benchmark questions.

> **In short**: doing engineering inside a research team means keeping pace with the model: every new model or harness version has to drop straight into the showcase.

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

## 5. Lumie — a Live2D AI companion app

**Apr 2026 – present ｜ AI engineering & mobile development, 2-person team**

> **Try it at [lumie.love](https://lumie.love).**

A mobile AI product for the emotional-companionship market: a Live2D avatar whose expressions track the conversation in real time, end-to-end realtime voice through Volcengine, and a long-term memory system that sustains a continuous relationship across days and topics. Built on a decoupled dual-repository architecture (mobile: Vue 3 + Vite + Capacitor + Live2D; server: Hono + Postgres + Neo4j + admin console), with the team collaborating through a multi-branch PR workflow.

### ① The Live2D character and conversation

Appearance, expression and lip-sync follow the emotion of the conversation in real time, with multiple characters and scene backgrounds. I own the last mile from model output to interface — stripping internal control markers out of replies, mapping emotion words onto the avatar's live expressions, and rendering answers word by word as they stream.

![Live2D conversation](images/lumie-chat.jpg)

### ② Long-term memory: a knowledge graph with graph-augmented retrieval

Pure vector retrieval cannot reach three things: multi-hop association ("that friend of hers who's always working late"), contradiction resolution (loved cats three months ago, scratched and afraid of them last week), and temporal anchoring ("a year ago today you'd just defended your thesis"). I designed and built the Neo4j-backed memory graph and its retrieval pipeline:

- **Extraction** — a regex pre-classifier first decides whether a turn is worth extracting at all; on a hit, an LLM structured-extraction step distils open-domain conversation into person / event / preference / emotion triples, merged idempotently into the graph
- **Storage** — the vector index is built directly on the graph nodes, so embeddings and relationships share one store and cannot drift apart
- **Retrieval** — entities are extracted from the incoming turn, then recall runs down two parallel paths, multi-hop graph neighbourhood and vector similarity, fused by a ranking over similarity, importance, time decay and hop distance before assembly into the memory injection block
- **Contradiction resolution** — when a new fact conflicts with an old one, a `SUPERSEDES` edge is created rather than an overwrite: the current answer uses the latest state, but "you used to love cats" stays in the graph as relationship depth

### ③ Making memory legible

I led the product rework of the memory module: what the backend stores as scattered rows was reorganised along a *how you see her / how she sees you* split into a four-part narrative, so the user opens a growing relationship rather than a database listing.

![Memory page](images/lumie-memory.jpg)

### ④ Prompt governance and offline evaluation

I consolidated the product's entire LLM prompting layer — strings hard-coded across 12+ backend services — into a database-backed registry of **21 typed slots** (persona, dialogue style, streaming and JSON output contracts, memory extraction and injection, Cypher generation, proactive openers, realtime-voice style), each editable live from the admin console, with variable interpolation, per-slot declared variables, and a shipped-in-code default that every slot degrades to. A broken or emptied edit falls back to the original prompt instead of injecting a blank system message. Prompt iteration went from a code change plus redeploy down to a console edit.

![Editing prompts from the admin console](images/lumie-prompts.jpg)

On top of that I built a ~750-line **offline evaluation harness** that turns prompt writing from taste into comparison. One script generates several stylistic candidates per slot; the other fills each candidate from a shared fixture library of realistic inputs and **actually runs it**, in two modes — *generative*, where the filled prompt is the request and the model's output is the artefact, and *behavioural*, where the candidate is injected as the system prompt and then probed with scripted user turns (a pushback, a distress message, a memory-worthy aside). Each slot carries its own temperature and token budget, the live production prompt is run on the identical input as a control, and everything lands in one side-by-side report that decides what ships.

### ⑤ Emotional features and progression

Mood logging, relationship milestones, a store and a wallet — the pieces that give companionship a sense of growth.

![Mood, milestones and store](images/lumie-features.jpg)

![Lumie repository contributions and PRs](images/lumie-contributions.jpg)

> **In short**: Lumie stands for a different capability — delivering continuously inside a real team codebase spanning multiple repositories and a multi-branch PR workflow. Cross-repository interface contracts, keeping mobile and server consistent, closing the loop on prompt iteration: none of these problems exist in a one-person, one-repository project.

---

*Every screenshot on this page comes from a real project I worked on.*
