# 🎬 Posty

## 🎓 Chung-Ang Univ. 2026 Spring Semester Capstone Design – Team 6

## 📘 Reference-Driven Automatic Short-Form Video Editor

*Note: Korean version of this document is available below. (한국어 버전은 아래에 있습니다.)*

## 🔖 1. Project Title

**“Posty”**

Give it **one reference Reels video + several of your own source clips**, and Posty produces an automatically edited **9:16 video that follows the editing style of the reference**.

> **Repository structure** — Frontend and backend live in **separate repositories**.
> - **[Posty_BE](https://github.com/2026-01-CAU-Capstone/Posty_BE)** — Backend + editing pipeline (Hono API + in-process job queue running the `lib/` pipeline asynchronously)
> - **[Posty_FE](https://github.com/2026-01-CAU-Capstone/Posty_FE)** — Frontend wizard UI (Vite + React + TypeScript). Communicates with the backend over HTTP only (default `http://localhost:8787`, override with `VITE_API_BASE`).

---

## 🧠 2. Abstract

Creating short-form videos that match a specific editing style is time-consuming. Even when a creator has a reference clip they want to emulate, manually replicating its cut rhythm, color grading, captioning, and music selection across their own footage requires significant editing skill and effort.

**Posty** is a service that **learns the editing style of a single reference Reels video and applies it to your own source clips automatically**. By analyzing the reference with multimodal AI and running a multi-stage editing pipeline, Posty handles scene cutting, color correction, subtitle burn-in, and background music selection without manual editing.

This allows creators to produce stylistically consistent 9:16 short-form content simply by providing a reference and their raw footage, dramatically lowering the barrier to polished video production.

---

## ❗ 3. Problem Statement

### 👤 3.1 Who

* **Primary**: Individual creators and students producing short-form content (Reels / Shorts / TikTok)
* **Secondary**: Small businesses and marketers who need consistent, on-brand short-form video without a dedicated editor

### ❓ 3.2 What

* Difficulty in **replicating a desired editing style** (cut rhythm, color tone, captioning, music feel) from a reference video onto one's own footage

### 🔍 3.3 Why

1. **High editing skill barrier**

   * Matching a reference's pacing, grading, and caption style by hand requires professional editing tools and experience

2. **Time-intensive manual workflow**

   * Scene selection, trimming, color grading, subtitle timing, and music matching each take significant manual effort

3. **Inconsistent output**

   * Without a systematic process, it is hard to keep a consistent style across multiple videos

### 📊 3.4 Evidence

* Short-form platforms reward stylistic consistency and posting frequency, but quality editing is the main bottleneck for individual creators
* Existing auto-editing tools apply generic templates rather than **learning the style of a specific reference the creator actually wants to emulate**
* No widely available tool maps **reference editing style → applied edit on the user's own clips** end-to-end

---

## 🎯 4. Objectives

### 🏆 Main Objective

To implement a web-based service that **analyzes a reference Reels video and automatically produces a 9:16 edited video from the user's own source clips in that style**.

### 📌 Sub Objectives

1. Analyze a reference video into a structured **style spec** using multimodal AI
2. Perform automatic scene-detection-based cut editing and curation of source footage (with auto-condensing for long sources)
3. Apply color grading, burned-in captions, and background music matched to the reference
4. Provide an intuitive wizard UI with real-time progress (percentage + ETA + animated mascot)

---

## 🛠️ 5. Proposed Solution

### 🔄 Pipeline (Stage 0–4)

| Stage | Role | APIs / Tools | Output |
|---|---|---|---|
| **0** | Reference analysis → style spec | **Gemini 2.5 Pro** (video understanding + JSON, 2-pass) | `0_spec/edit-spec.json` |
| **1** | Source cut editing (+ auto-condensing of long sources) | **FFmpeg scene detect** → **Gemini Flash** descriptions (batched for long sources) → **OpenAI embedding** matching → 30–60s curation → **FFmpeg** 9:16 cut/concat | `1_cut/cut.mp4` + `edit-plan.json` |
| **2** | Color grading | **FFmpeg signalstats** measurement → `eq` / `colorbalance` | `2_grade/graded.mp4` |
| **3** | Captions | **FFmpeg subtitles** (ASS / libass burn-in) | `3_caption/captioned.mp4` |
| **4** | Voice / BGM | **Internet Archive** auto BGM (+ optional **AudD** reference-track fingerprinting) + **Gemini TTS** (optional) + FFmpeg mix | `4_final/final.mp4` |

Each stage writes a checkpoint file, so a single stage can be re-run independently (`POST /api/run` with `mode:'stage'`).

### 🔄 User Flow

1. **Reference** (Instagram URL or file) → "Start analysis and continue →" → Posty the bear analyzes the reference in the background (Stage 0)
2. While analysis runs, add **source clips** (files / IG URLs) + **editing options** (subtitle language & frequency, tone, keywords, etc.)
3. **✨ Generate video** → runs Stages 1–4 with live **percentage + ETA + Posty animation**
4. Done → preview + download

### ⭐ Core Features

* Reference-style-driven automatic editing of your own clips
* Scene-detection-based cut editing with automatic condensing of long footage
* Color grading, burned-in captions, and BGM matching applied automatically
* Stage-based checkpoints allowing any single stage to be re-run

### 💡 Unique Selling Points (USP)

* **Style transfer, not templates**: Learns the editing style of a specific reference the creator chooses, rather than applying generic presets
* **End-to-end automation**: From raw clips to a finished 9:16 video with cuts, color, captions, and music
* **Lowered editing barrier**: Produces polished short-form content without professional editing skills
* **Modular & re-runnable**: Checkpoint-based pipeline makes iteration on a single stage fast and cheap

---

## 👥 6. Team Members & Roles

| Name | Student ID | Role | Responsibilities |
| ---- | ---------- | ---- | ---------------- |
| Guebeen Lee | 20235908 | AI | • Media research & presentation |
| Mingyu Lee | 20215143 | BE / PM | • Documentation, presentation, and backend development |
| Dongyun Ha | 20212007 | FE / BE / AI | • Overall project architecture, frontend & backend development |

---

## ⚙️ 7. Technology Stack

| Domain | Technology |
| ------ | ---------- |
| Frontend | Vite + React + TypeScript |
| Backend | Hono (API + in-process job queue) |
| Editing Engine | FFmpeg / FFprobe (scene detect, cut/concat, signalstats grading, ASS/libass subtitle burn-in, audio mix) |
| AI Models | - Reference analysis: **Gemini 2.5 Pro**<br>- Clip description: **Gemini Flash**<br>- Embedding / matching: **OpenAI embeddings**<br>- TTS (optional): **Gemini TTS** |
| External Services | - BGM: **Internet Archive**<br>- Reference-track fingerprinting (optional): **AudD**<br>- Instagram URL → mp4 (optional): FastAPI `ig-fetch` service |
| Requirements | Node 18+, FFmpeg/FFprobe on PATH |

---
---

# 🎬 Posty (KR Ver.)

## 🎓 2026년 1학기 중앙대학교 캡스톤 디자인 – 6조

## 📘 레퍼런스 기반 자동 숏폼 영상 편집기

---

## 🔖 1. Project Title (프로젝트 제목)

**“Posty”**

**레퍼런스 릴스 1개 + 내 소스 영상 여러 개**를 주면, **레퍼런스의 편집 스타일을 따라** 자동 편집된 **9:16 영상**을 만들어 줍니다.

> **저장소 구조** — 프런트엔드와 백엔드는 **저장소가 분리**되어 있습니다.
> - **[Posty_BE](https://github.com/2026-01-CAU-Capstone/Posty_BE)** — 백엔드 + 편집 파이프라인 (Hono API + in-process 작업 큐가 `lib/` 파이프라인을 비동기로 실행)
> - **[Posty_FE](https://github.com/2026-01-CAU-Capstone/Posty_FE)** — 프런트엔드 마법사 UI (Vite + React + TypeScript). HTTP 로만 백엔드와 통신 (기본 `http://localhost:8787`, `VITE_API_BASE` 로 변경)

---

## 🧠 2. Abstract (요약)

특정 편집 스타일에 맞춰 숏폼 영상을 만드는 일은 많은 시간이 든다. 따라하고 싶은 레퍼런스 영상이 있어도, 컷 리듬·색감 보정·자막·음악 선곡을 내 영상에 직접 재현하려면 상당한 편집 실력과 노력이 필요하다.

**Posty**는 **레퍼런스 릴스 1개의 편집 스타일을 학습해 내 소스 영상에 자동으로 적용**하는 서비스다. 멀티모달 AI로 레퍼런스를 분석하고 다단계 편집 파이프라인을 실행하여, 장면 컷·색 보정·자막 번인·배경음악 선곡을 수작업 없이 처리한다.

이를 통해 창작자는 레퍼런스와 원본 영상만 제공하면 스타일이 일관된 9:16 숏폼 콘텐츠를 만들 수 있어, 완성도 높은 영상 제작의 진입 장벽을 크게 낮춘다.

---

## ❗ 3. Problem Statement (문제 정의)

### 👤 3.1 Who (대상)

* **Primary**: 숏폼 콘텐츠(릴스 / 쇼츠 / 틱톡)를 제작하는 개인 창작자 및 학생
* **Secondary**: 전담 편집자 없이 일관된 브랜드 톤의 숏폼 영상이 필요한 소상공인·마케터

### ❓ 3.2 What (문제)

* 레퍼런스 영상의 **편집 스타일**(컷 리듬, 색감, 자막, 음악 느낌)을 내 영상에 **재현하기 어려움**

### 🔍 3.3 Why (원인)

1. **높은 편집 실력 장벽**

   * 레퍼런스의 페이싱·그레이딩·자막 스타일을 수작업으로 맞추려면 전문 편집 도구와 경험이 필요

2. **시간 소모적인 수동 작업**

   * 장면 선별, 컷 편집, 색 보정, 자막 타이밍, 음악 매칭 각각에 많은 수작업이 필요

3. **일관성 부족**

   * 체계적인 프로세스가 없으면 여러 영상에 걸쳐 일관된 스타일을 유지하기 어려움

### 📊 3.4 Evidence (근거)

* 숏폼 플랫폼은 스타일 일관성과 업로드 빈도를 보상하지만, 품질 편집이 개인 창작자의 가장 큰 병목
* 기존 자동 편집 도구는 **창작자가 실제로 따라하고 싶어 하는 특정 레퍼런스의 스타일을 학습하지 못하고**, 일반적인 템플릿만 적용
* **레퍼런스 편집 스타일 → 내 영상에 적용된 편집**을 처음부터 끝까지 매핑해 주는 도구는 시중에 없음

---

## 🎯 4. Objectives (목표)

### 🏆 Main Objective

* 레퍼런스 릴스 영상을 분석하여, 사용자의 소스 영상으로 **해당 스타일의 9:16 편집 영상을 자동 생성**하는 웹 기반 서비스 구현

### 📌 Sub Objectives

1. 멀티모달 AI로 레퍼런스 영상을 구조화된 **스타일 스펙**으로 분석
2. 장면 검출 기반 컷 편집과 소스 영상 큐레이션 자동 수행 (긴 소스는 자동 축약)
3. 레퍼런스에 맞춘 색감 보정, 자막 번인, 배경음악 적용
4. 실시간 진행 상황(퍼센트 + ETA + 마스코트 애니메이션)을 보여주는 직관적 마법사 UI 제공

---

## 🛠️ 5. Proposed Solution (제안하는 해결책)

### 🔄 파이프라인 (Stage 0–4)

| 단계 | 역할 | 사용 API / 도구 | 산출물 |
|---|---|---|---|
| **0** | 레퍼런스 영상 분석 → 스타일 스펙 | **Gemini 2.5 Pro** (영상 이해 + JSON, 2패스) | `0_spec/edit-spec.json` |
| **1** | 소스 컷편집 (+ 긴 소스 자동 축약) | **FFmpeg scene detect** → **Gemini Flash** 묘사(긴 소스는 batch) → **OpenAI embedding** 매칭 → 30–60초 큐레이션 → **FFmpeg** 9:16 cut/concat | `1_cut/cut.mp4` + `edit-plan.json` |
| **2** | 색감 보정 | **FFmpeg signalstats** 측정 → `eq` / `colorbalance` | `2_grade/graded.mp4` |
| **3** | 자막 | **FFmpeg subtitles** (ASS / libass burn-in) | `3_caption/captioned.mp4` |
| **4** | 음성 / BGM | **Internet Archive** 자동 BGM (+ 선택: **AudD** 레퍼런스 곡 지문인식) + **Gemini TTS**(선택) + FFmpeg 믹스 | `4_final/final.mp4` |

각 단계는 체크포인트 파일을 남겨서 한 단계만 다시 돌릴 수 있습니다 (`POST /api/run` `mode:'stage'`).

### 🔄 사용 흐름

1. **레퍼런스**(IG URL 또는 파일) 입력 → "분석 시작하고 다음 →" → 곰돌이 Posty 가 백그라운드에서 레퍼런스를 분석(Stage 0)
2. 분석이 도는 동안 **소스 영상** 추가(파일 / IG URL) + **편집 옵션**(자막 언어·빈도, 톤, 키워드 등) 입력
3. **✨ 영상 생성** → Stage 1–4 실행. 실시간 **퍼센트 + 남은시간(ETA) + Posty 애니메이션**
4. 완료 → 미리보기 + 다운로드

### ⭐ Core Features

* 레퍼런스 스타일 기반 내 영상 자동 편집
* 장면 검출 기반 컷 편집 + 긴 소스 자동 축약
* 색감 보정 · 자막 번인 · BGM 매칭 자동 적용
* 단계별 체크포인트로 특정 단계만 재실행 가능

### 💡 Unique Selling Point (USP)

#### ✅ 기대효과

* **템플릿이 아닌 스타일 전이**: 일반 프리셋이 아니라 창작자가 고른 특정 레퍼런스의 편집 스타일을 학습해 적용
* **엔드 투 엔드 자동화**: 원본 클립부터 컷·색·자막·음악이 입혀진 완성 9:16 영상까지
* **편집 진입 장벽 완화**: 전문 편집 실력 없이도 완성도 높은 숏폼 콘텐츠 제작
* **모듈식 · 재실행 가능**: 체크포인트 기반 파이프라인으로 특정 단계만 빠르고 저렴하게 반복

#### 🚀 확장성

* 다양한 플랫폼 대응: 릴스뿐 아니라 쇼츠·틱톡 등 9:16 숏폼 전반에 적용 가능
* 스타일 라이브러리로 발전: 여러 레퍼런스를 저장·재사용하는 개인화 스타일 프리셋 구조로 확장 가능
* 협업·브랜드 도구로 확장: 일관된 브랜드 톤이 필요한 소상공인·마케터용 도구로 발전 가능

---

## 👥 6. 팀원 목록 및 역할 분담

| 이름 | 학번 | 역할 | 상세 내용 |
| ---- | ---- | ---- | -------- |
| 이규빈 (Guebeen Lee) | 20235908 | AI | • Media Research 및 발표 담당 |
| 이민규 (Mingyu Lee) | 20215143 | BE, PM | • 문서화, 발표 및 백엔드 담당 |
| 하동윤 (Dongyun Ha) | 20212007 | FE, BE, AI | • 프로젝트 설계 총괄 및 FE, BE 담당 |

---

## ⚙️ 7. Technology Stack (기술 스택)

| 영역 | 기술 |
| ---- | ---- |
| Frontend | Vite + React + TypeScript |
| Backend | Hono (API + in-process 작업 큐) |
| 편집 엔진 | FFmpeg / FFprobe (scene detect, cut/concat, signalstats 색보정, ASS/libass 자막 번인, 오디오 믹스) |
| AI Model | - 레퍼런스 분석: **Gemini 2.5 Pro**<br>- 클립 묘사: **Gemini Flash**<br>- 임베딩 / 매칭: **OpenAI embeddings**<br>- TTS(선택): **Gemini TTS** |
| 외부 서비스 | - BGM: **Internet Archive**<br>- 레퍼런스 곡 지문인식(선택): **AudD**<br>- Instagram URL → mp4(선택): FastAPI `ig-fetch` 서비스 |
| 요구사항 | Node 18+, FFmpeg/FFprobe (PATH) |
