# AI 기반 신약개발 폐루프(Closed-Loop) 오케스트레이션 기술 동향 보고서

> **초점**: AlphaFold 계열 구조 예측 → AI가 합성 실험 시나리오 스케줄링/오케스트레이션 → 반복 실험·결과 검토·피드백으로 이어지는 '자가주행 실험실(Self-Driving Lab, SDL)' 및 '에이전틱 AI(Agentic AI)' 연구를 수행하는 해외·국내 기관 조사

작성일: 2026-09-04 | 수집 근거: 공개 논문, 국책과제 보도자료, 기관 공식 발표

---

## 목차

1. [개요: '질문 시나리오'가 실제 어떻게 구현되는가](#1-개요)
2. [핵심 기술 요소](#2-핵심-기술-요소)
3. [해외 사례](#3-해외-사례)
   - 3.1 스탠포드/Chan Zuckerberg Biohub — Virtual Lab *(구조예측+에이전트+실험검증 폐루프의 대표 사례)*
   - 3.2 Amgen - Tippy / Artificial — AI + 로봇 합성 + 결과 피드백
   - 3.3 FROGENT — 다중 에이전트 오케스트레이션
   - 3.4 Rhizome OS-1 — 반자율 탐색 OS
   - 3.5 OrchestRA — 약리·독성 피드백 기반 최적화 루프
   - 3.6 기타 합성 자동화/오케스트레이션 연구 (ChemOS, A-Lab, Coscientist, AutoLabs 등)
4. [국내 사례](#4-국내-사례)
   - 4.1 한국제약바이오협회 AI신약연구원 — AI 신약개발 자율실험실
   - 4.2 K-AI 신약개발 전임상·임상 모델개발사업 (복지부, 371억) — 아이젠사이언스 오케스트레이션
   - 4.3 K-MELLODDY — 연합학습 기반 플랫폼 (전주기 예측 기반)
   - 4.4 JW중외제약 — 제이웨이브(JWave) + 로봇 합성 자동화 (구조기반 + SDL)
   - 4.5 IBS/UNIST AI·로봇 기반 합성연구단 — 대규모 자동 합성
   - 4.6 국내 자율실험실(SDL) 협의체 & 산업부 ADC 자율랩
5. [질문 시나리오에 가장 부합하는 사례 요약](#5-질문-시나리오에-가장-부합하는-사례)
6. [참고문헌 및 링크](#6-참고문헌-및-링크)

---

## 1. 개요

사용자가 묻는 시나리오는 학계·산업계에서 크게 두 가지 흐름으로 구현되고 있다.

1. **가상 폐루프(Virtual Closed-Loop)**: AlphaFold·ESM·Rosetta 등 구조예측 도구를 다중 LLM 에이전트가 오케스트레이션하여 설계-평가-정제를 반복. 실험은 별도 검증.
2. **물리 폐루프(Physical Closed-Loop / SDL)**: 로봇·자동화 장비 + AI가 설계→합성→측정(HPLC 등)→분석 결과를 다음 설계에 피드백하는 '가설-실험-분석-재가설' 사이클을 사람 개입 없이 24시간 반복.

두 흐름 모두 "AI 스케줄링/오케스트레이션 + 반복 실험 + 결과 피드백"이라는 사용자 질문의 핵심과 정확히 일치한다.

---

## 2. 핵심 기술 요소

| 요소 | 역할 | 대표 사례 기술 |
|------|------|----------------|
| 구조 예측 | 표적 단백질·복합체 구조 예측 | AlphaFold3, AlphaFold-Multimer, ESM, Boltz-2, Rosetta |
| 생성/설계 | 신규 분자·항체·나노바디 설계 | RFdiffusion, ProteinMPNN, Makya, r1(diffusion-GNN) |
| 오케스트레이션 | 작업 분해·에이전트 협업·스케줄링 지휘 | Orchestrate/Supervisor/PI 에이전트, Tippy, Ilaka |
| 합성 계획 | 역합성(합성경로) 설계 | Spaya, retrosynthesis AI, DirectMultiStep |
| 실험 자동화 | 로봇 합성·시료처리·측정 | Chemspeed, Big Kahuna, LabOps, 언체인·로봇 |
| 평가/검증 | 도킹·결합에너지·ADMET/PK·PBPK 스코어링 | GNINA, AutoDock, QVina, ADMET-ai, Boltz-2 |
| 폐루프 제어 | 결과 피드백→다음 설계 반영 | Bayesian 최적화, 강화학습, 에이전트 의사결정 로직 |

---

## 3. 해외 사례

### 3.1 스탠포드 / Chan Zuckerberg Biohub — Virtual Lab (가장 부합하는 대표 사례)

- **게재**: *Nature* 646, 716-723 (2025-07-29). DOI: 10.1038/s41586-025-09442-9
- **연구진**: Kyle Swanson(Stanford CS), Wesley Wu, Nash L. Bulaong, John E. Pak, James Zou(교신) / CZ Biohub San Francisco
- **코드·데이터**: GitHub `zou-group/virtual-lab`, Zenodo DOI 10.5281/zenodo.15320491 / 15331309

#### 기술 아키텍처
- AI-human 협업 연구 수행을 위한 **다중 LLM 에이전트 팀**.
- 구조: **LLM Principal Investigator(PI) 에이전트**가 과학자 에이전트 팀을 이끌고, **인간 연구자**는 상위 레벨 피드백만 제공.
- 에이전트 설계 4요소(prompt): 제목(title), 전문분야(expertise), 목표(goal), 역할(role).
- 한 연구에 PI가 **생성하는 에이전트들**: 면역학(Immunologist), 계산생물학(Computational Biologist), 머신러닝(ML Specialist) + **비평가(Scientific Critic)** 에이전트(타 에이전트의 착오·누락 지적과 건설적 비판 전담).

#### 협업 방식: 팀 회의 + 개별 회의
- **Team meetings**: 모든 에이전트가 공동으로 광범위한 연구 질문 토론.
- **Individual meetings**: 단일 에이전트가 구체적 과업(예: ML 코드 작성) 수행, 다른 에이전트가 비판적 피드백.
- **병렬 회의(parallel meetings) merge 기법**: 동일 안건을 높은 LLM 온도(창의성)로 여러 번 병렬 실행 → merge meeting(낮은 온도·일관성)에서 최적 답안 병합.

#### 실제 적용: SARS-CoV-2 나노바디 설계 파이프라인
에이전트들이 스스로 결정/구축한 계산 워크플로:
1. **도구 선정**(팀 회의): ESM + AlphaFold-Multimer + Rosetta 채택.
2. **구현**(개별 회의): 각 도구용 파이썬/로제타스크립트 코드 작성.
   - ESM: 변이 시퀀스의 LLR(log-likelihood ratio) 계산.
   - AlphaFold-Multimer(LocalColabFold v1.5.5): 나노바디-스파이크 복합체 구조 예측 후 **ipLDDT**(결합 인터페이스 신뢰도) 계산용 144줄 파이썬 스크립트.
   - Rosetta: RosettaScripts XML로 복합체 relax 후 결합 에너지(dG) 계산.
3. **워크플로 설계**(PI 개별 회의): PI가 가중 스코어 발명
   - `WS = 0.2*(ESM LLR) + 0.5*(AF ipLDDT) − 0.3*(Rosetta dG)`
   - 4개 시작 나노바디 각각 → ESM으로 전체 단일 점돌연변이 평가 → 상위 20개 → AlphaFold-Multimer+Rosetta로 재평가 → 상위 5개를 다음 라운드 시작점으로. 총 4라운드 (반복 설계-평가-정제 루프).

#### 결과
- **92개 신규 나노바디 설계 → 실제 실험 검증**. 수용성(90%+), 결합 프로필 우수.
- 특히 JN.1/KP.3 변이에 결합력이 개선된 2개 나노바디 발굴(Uhan 파생물과 교차 결합 유지).
- 실험 데이터가 다시 AI 연구실로 피드백되어 분자 설계를 추가로 개선(진정한 폐루프).

> **의의**: "LLM PI 에이전트가 팀을 스케줄링→도구(AlphaFold 등) 오케스트레이션→반복 설계→실험 검증→결과 피드백"이라는 사용자 질문 시나리오를 거의 그대로 실현한 국제적 대표 사례.

---

### 3.2 Amgen - Tippy / Artificial — AI + 로봇 합성 + 결과 피드백

- **문헌**:
  - "Accelerating drug discovery with Artificial: a whole-lab orchestration and scheduling system for self-driving labs" — arXiv:2504.00986 (Fehlis, Mandel, Crain 등)
  - "Accelerating Drug Discovery Through Agentic AI: A Multi-Agent Approach to Laboratory Automation in the DMTA Cycle" — arXiv:2507.09023

#### Artificial (전 실험실 오케스트레이션 플랫폼)
- **습관 사이클 4단계**: Design(워크플로 설계) → Run(LabOps 실시간 장비·로봇·인력 스케줄링·모니터링) → Learn(모든 과학·공정 데이터 로깅) → Optimize(AI가 다음 실험 최적화). → 폐루프.
- **AI 통합**: NVIDIA BioNeMo(NIM 마이크로서비스)의 구조예측·접힘·분자상호작용·도킹·결합에너지 스코어링을 실험실 워크플로에 통합.
- **PoC(자가주행 가상 스크리닝)**: SARS-CoV-2 메인 단백질가수분해효소 대상. 분자 선별→(필요시)단백질 접힘→도킹→결합에너지 스코어링을 반복, 종료 기준(-1.4백만 결합 문턱값 / 10분자 이상)까지 자가 구동.

#### Tippy (DMTA 사이클 다중 에이전트, '최초의 프로덕션 레디' 구현)
- **5개 전문화 에이전트 + 안전 가드레일**:
  - **Supervisor Agent**: 사람 연구자와의 인터페이스, 프로젝트 컨텍스트·타임라인 관리, 과업 위임·에이전트 간 핸드오프 조정 → 오케스트레이터 역할.
  - **Molecule Agent**: 컴퓨터 화학(분자 설계·최적화) — Design 단계.
  - **Lab Agent**: Artificial 플랫폼과 연결, HPLC 분석 워크플로·합성 절차·실험실 잡 실행·스케줄 지정 — Make/Test 단계.
  - **Analysis Agent**: 실시간 데이터 처리·통계 분석(예: HPLC retention time) — Analyze 단계.
  - **Report Agent**: 전체 사이클 문서화.
- **폐루프 학습**: HPLC retention time을 분자 생성을 안내하는 핵심 스코어링 지표로 사용 → 분석 결과가 Molecule Agent에 피드백되어 다음 설계 반복. **"설계-합성-시험-분석→재설계"가 사람 개입 없이 전자동으로 순환.**

---

### 3.3 FROGENT — 다중 에이전트 오케스트레이션

- **문헌**: "FROGENT: An End-to-End Full-process Drug Design Multi-Agent System" — arXiv:2508.10760

#### 아키텍처
- **4개 코너스톤 에이전트**:
  - **Orchestrate Agent**(중앙 컨트롤러): ReAct-style 계획 분해→구조화된 태스크 그래프 생성→전문 에이전트에 디스패치→상태 관리→전역 피드백 루프 관리.
  - **Retrieve Agent**: 역동적 생화학 DB, 도구 라이브러리, MCP(Model Context Protocol)로 지식 수집.
  - **Forge Agent**(생성 코어): 분자 생성·리드 최적화·역합성 계획. 딥러닝+Docking(Tool Routing)+코드 실행(direct tool API).
  - **Gauge Agent**(검증 코어): ADMET-ai(ADMET 예측), QVina(소분자 도킹), MDockPeP2(펩타이드 도킹) 등 정량 스코어링.

#### 폐루프 최적화
- **"design–evaluate–refine"** 루프: Forge가 분자 생성/정제 → Gauge가 스코어링·검증 → Orchestrate가 결과를 소화해 Forge에 재태스킹 → 제약 만족 후보까지 다회 반복.
- **합성 수리(synthesis repair)**: 역합성 경로 실패 시 Orchestrate가 하위 작업으로 수리 경로 재지시.
- 8개 벤치마크에서 엔드투엔드 파이프라인(표적 식별→소분자 생성→펩타이드 최적화→역합성 계획) 검증.

---

### 3.4 Rhizome OS-1 — 반자율 탐색 OS

- **문헌**: "Rhizome OS-1: Rhizome's Semi-Autonomous Operating System for Small Molecule Drug Discovery" — arXiv:2604.07512

#### 아키텍처
- **다중 모달 AI 에이전트 = 다학제 팀**(전산화학·의약화학·특허 에이전트):
  - **구조 분석가(Structural analyst)**: 공동결정 구조·활성 데이터, 지문 클러스터링 코드 작성·실행, 의약화학 가설(예: "아마이드 링커를 1,3,4-옥사디아졸로 대체") 수립.
  - **생성기(Generator)**: 각 가설을 r1(generation primitive)으로 변환·실행.
  - **평가자(Evaluator)**: 비전 능력으로 분자 그리드를 육안 검사, 변형 고리/비합성가능 구조 플래깅, 합성 용이성·특허 자유운영(FTO) 평가.
  - **최적화기(Optimizer)**: 스크리닝 결과 프로그램 분석, 전략×시드 성능 히트맵 구축, 다음 시드 선정.
  - **오케스트레이터**: 파동(wave) 스케줄링, 토큰 배분, 에이전트 디스패치, **수렴 라이브러리(1,000~2,000분자/표적) 유지**.
- **생성 엔진**: r1 — 246M 파라미터 그래프 확산 모델(8억 분자 학습), 원자·결합을 표현(SMILES 대신 그래프), beam search 디코딩, fragment masking/scaffold decoration/linker design/graph editing 4가지 생성 원시.
- **스코어링 계층**: Boltz-2 결합 친화력 예측을 ChEMBL 활성 데이터로 보정(오류 보정, Spearman −0.53~−0.64, ROC AUC 0.88~0.93).
- **결과**: BCL6·EZH2 두 암 표적에서 5,231개 신규 분자 생성 → 약 91.9% Murcko 스캐폴드가 ChEMBL 신규.

---

### 3.5 OrchestRA — 약리/독성 피드백 기반 최적화 루프

- **문헌**: "OrchestRA: Orchestrated Rational drug design Agents" — arXiv:2512.21623

#### 아키텍처
- **인간-in-the-loop + 3개 특화 에이전트 + Orchestrator**:
  - **Biologist Agent**: >1,000만 연결 지식그래프(KG) 심층 추론으로 표적 식별.
  - **Chemist Agent**: 구조 기반 de novo 설계(생성 확산 모델 DiffSBDD) + ChEMBL/DrugBank 스크리닝/재배치. 계층 필터링(AutoDock Vina 물리 HTVS → Boltz-2 고충실도 검증).
  - **Pharmacologist Agent**: ADMET 예측 + 5-compartment PBPK 시뮬레이션(Gut/Liver/Kidney/Central/Non-eliminating tissue) 가상 어세이.
- **자율 Drug Optimization Cycle(폐루프)**:
  - Pharmacologist 진단 피드백(대사 안정성 부족·독성 등) → 즉시 Chemist에 전달 → 표적 구조 재최적화 유발.
  - Orchestrator가 결정 로직(`should_continue`, 사전 정의 승인 플래그 기반)으로 `is_approved=True` 또는 최대 반복까지 사이클 유지.
  - 혼합 전략: **Bayesian 최적화(BO) + 유전 알고리즘(GA)** 로 화학 공간 탐색, 독성/용해도 시 페널티 항을 Chemist 점수에 동적 반영.
- **검증**: 파라세타몰 임상 혈장 농도 프로파일 재현으로 PBPK 예측 신뢰도 확인.

---

### 3.6 기타 합성 자동화 / 오케스트레이션 연구

| 시스템 | 기관 | 개요 |
|--------|------|------|
| **ChemOS / ChemOS 2.0** | ETH Zurich / Zhiheng et al. | 화학 자가주행 실험실 오케스트레이션 오픈소스 |
| **A-Lab** | Lawrence Berkeley | 로봇·클라우드 랩 연계, 17일간 신규 무기화합물 41종을 71% 성공률로 자율 합성 |
| **Coscientist / ChemCrow** | Carnegie Mellon | LLM + 분광기·액체핸들러·크로마토그래피 연동, Suzuki/Sonogashira 반응을 4분 내 설계·첫 시도 성공 |
| **AutoLabs** | PNNL(오픈소스) | 자연어 → 고처리량 액체핸들러(Big Kahuna) XML 프로토콜 번역. 다중에이전트 + 자체교정, 복잡 다판 합성에서 F1>0.89, 수치 오류 85%+ 감소 |
| **RoboChem-Flex** | 학술 | 저가 모듈식 SDL(약 $5,000), Bayesian 최적화, 폐루프 반응 최적화 |
| **IvoryOS** | 학술 | 파이썬 기반 SDL 상호운용 웹 인터페이스 오케스트레이션 |
| **AlphaFlow** | 학술 | 강화학습 기반 자가구동 유체 실험실, 다단계 화학 자율 최적화 |

---

## 4. 국내 사례

### 4.1 한국제약바이오협회 AI신약연구원 — AI 신약개발 자율실험실

- **성격**: 인프라/테스트베드 (오픈이노베이션 생태계 조성, 업계 이론교육 260명 이상 실시).
- **구현**:
  - 로봇 자동화 + 알고리즘 기반 의사결정을 결합한 **폐루프(Closed-loop) 실험 자율화 시스템**.
  - 시료 전처리(피펫 작업) 등 단순 반복 공정을 로봇이 대체.
  - 결과 분석 → 목표 합성 수율 도달을 위한 **다음 실험 조건 자동 재설계 → 24시간 반복** 수행.
  - 장비 정지·이상 시 사전 안전규칙·제어 로직에 따라 대응.
- **환경**: 신약개발 데이터 보안 위해 외부 네트워크 분리된 오프라인 환경, 보안 거버넌스 구축.

### 4.2 K-AI 신약개발 전임상·임상 모델개발사업 (보건복지부, 약 371억원/4년 3개월)

- **주관 체계**: (총괄/1주관) 한국제약바이오협회 AI신약연구원 · (2주관) 서울대병원 · (3주관) 삼성서울병원 · (4주관) 한국생명공학연구원. 참여: 국가임상시험지원재단, **아이젠사이언스**, APACE, C&R리서치, 고려대, LG CNS, 삼진제약, 동아ST, 한미약품, 대웅제약 등.
- **아이젠사이언스의 역할 — 핵심 두 축**:
  1. **연합학습 기반 파운데이션 모델**: 방대한 전임상·임상 데이터를 연합학습 플랫폼(K-MELLODDY 경험 기반)에서 학습해 신약개발 전반에 활용되는 '핵심 두뇌' 모델 개발.
  2. **AI 에이전트 오케스트레이션(Agent Orchestration)**: 컨소시엄 내 다른 주관기관이 개발하는 중개연구 AI, 역이행 연구설계 AI, 동물실험 대체 AI 등 **총 6종+ 전문 AI 소프트웨어를 유기적으로 연동·지휘** — "오케스트라 지휘자" 비유.
- **목표**: 전임상-임상을 AI로 연계·가속화, AI 기반 임상시험 설계·지원 플랫폼 구축. 2단계(2028~2029)에서 IND 승인 등 6건 실증.

### 4.3 K-MELLODDY — 연합학습 기반 신약개발 가속화 프로젝트 (과기정통부·복지부, 348억/5년, 2024.04~2028.12)

- **배경**: EU MELLODDY(Merck, Pfizer, Novartis, AstraZeneca 등)에서 영감. **Korea Machine Learning Ledger Orchestration for Drug Discovery**.
- **3개 축**:
  - 세부1: 연합학습 기반 신약개발 플랫폼(FDD) 구축 — 에비드넷(연세대·코어시큐리티·한국전자기술연구원).
  - 세부2: 데이터 표준화·공급·활용.
  - 세부3: **FAM(Federated ADMET Model)** 개발 — in-vitro·in-vivo·임상(in-human) 데이터를 연계해 ADMET 및 임상 PK 파라미터까지 예측. 데이터 추가 시 자동·연속 성능 개선(진화형 모델). 참여: 고려대, 목암생명과학연구소, 숭실대, 아이젠사이언스, LG화학, 연세대, 전북대, KAIST, 온코크로스, 한국화학연구원 등.
- **의의**: 신약개발 '전주기' 예측(ADMET/PK)의 기반 모델을 구축—물리 실험 장비 대신 **데이터·예측 레벨에서의 폐루프**. (구조예측-합성 스케줄링보다는 전임상-임상 연계 예측 자동화에 초점)

### 4.4 JW중외제약 — 제이웨이브(JWave) + 로봇 합성 자동화 (구조기반 AI + SDL)

- **과제**: 보건복지부 2026 제1차 보건의료기술 R&D '구조 기반 AI신약개발지원' 주관연구개발기관.
- **아키텍처 (사용자 시나리오와 고도로 부합)**:
  1. **JWave(제이웨이브)** — 자체 AI 통합 플랫폼: 500여 종 세포주·오가노이드·질환 동물모델 유전체 정보 + 4만여 자체 합성 화합물 데이터 → **구조 기반 모델링 + 강화학습 등 AI모델 20여종**으로 표적 단백질 구조·결합부위 정밀 분석, 유효성·선택성·약물특성 고려 화합물 능동 설계.
  2. **로봇 기반 합성 자동화 시스템** — AI가 제안한 화합물을 로봇이 자동 합성·생산, 반복 합성 공정 자동화.
  3. **C&C신약연구소** — 도출 화합물의 유효성·약물특성 신속 검증 → 비임상 진입 항암 후보 발굴.
- **폐루프**: "설계→합성→평가"가 반복 연구 사이클로 효율화 — **구조예측(AI설계) → 로봇 합성 → 평가 피드백**의 국내 기업 사례.

### 4.5 IBS / UNIST — AI·로봇 기반 합성연구단 (그쥐보브스키 연구팀)

- **게재**: *Nature* (2025-09-25 온라인).
- **플랫폼**: 하루 약 1,000회 화학 실험을 자동 수행하는 AI·로봇 플랫폼. 수천 가지 반응 조건 동시 실험 → 결과를 **정밀 지도(네트워크)로 시각화**, 습은 반응 경로 발견, 원하는 물질 선택적 생성.
- **성과**:
  - 150년 전 '한츠슈 피리딘 합성반응' 재구성 → 기존 7종 외 신규 중간체·생성물 **9종 규명**.
  - 프러시안 블루 유사체(PBA) 금속 조성 756가지 합성 → 최적 조합 + 신규 생성물 4종.
- **의의**: 로봇으로 실험 데이터를 대규모 축적 → AI 학습·연계 → 미지 화학 영역 탐구 가속. **합성 단계 폐루프 데이터 생성**에 특화.

### 4.6 국내 자율실험실(SDL) 협의체 & 산업부 ADC 자율랩

- **자율실험실 산·학·연 협의체 출범** (한국기초과학지원연구원·SK하이닉스 등 **80개 기관/기업**, 2026):
  - 미션: 기술·플랫폼·표준 정립(1분과: 파크시스템스·에이치비솔루션·에이블랩스 등), 레퍼런스 SDL 구축·연구운영(2분과: 기계연·재료연·생명연·KIST), 수요기업 실증·확산(3분과: SK하이닉스·LG화학·현대차).
  - "**가설–실험–분석–재가설**" 폐루프 체계의 문화적·인프라적 기반을 국가 차원에서 구축.
- **경보제약 — 산업부 'AI 기반 ADC 제조 자율랩 기술개발'** (총 192억, 2029.12까지, 경보 24억 R&D비):
  - 한국기계연구원·고려대 등과 AI+로봇 자율 실험실·의약품 자동화 제조 시스템 구축, AI 기반 자율 제조장비로 공정 실시간 모니터링·제어.

---

## 5. 질문 시나리오에 가장 부합하는 사례

사용자의 질문(AI 폐루프 신약개발 오케스트레이션)을 가장 직접적으로 구현하는 사례:

| 순위 | 기관/시스템 | 부합도 | 이유 |
|------|------------|--------|------|
| 1 | **스탠포드/CZ Biohub Virtual Lab** | ★★★★★ | LLM PI 에이전트 팀이 AlphaFold를 포함한 도구를 오케스트레이션, 반복 설계-평가, 실험 검증, 결과 피드백까지 완전 폐루프 |
| 2 | **Amgen - Tippy/Artificial** | ★★★★★ | AI 오케스트레이션 + 로봇 합성 스케줄링 + HPLC 분석 결과를 다음 설계에 피드백하는 물리 폐루프 DMTA |
| 3 | **JW중외제약 (JWave)** | ★★★★☆ | 구조기반 AI 설계 → 로봇 합성 자동화 → 평가 피드백의 국내 기업 사례 |
| 4 | **아이젠사이언스 (K-AI 과제)** | ★★★★☆ | 연합학습 파운데이션 모델 + 6종+ AI 에이전트 오케스트레이션 (국내 국책과제) |
| 5 | **FROGENT / OrchestRA / Rhizome OS-1** | ★★★☆☆ | 다중 에이전트 가상 폐루프, 합성 계획(역합성) 포함, 실험 검증 단계는 가상/연동 수준 |
| 6 | **한국제약바이오협회 AI신약연구원** | ★★★☆☆ | 로봇 + 최적화 알고리즘 폐루프 합성조건 최적화 (구조예측 단계 미초점) |

---

## 6. 참고문헌 및 링크

### 해외 (논문/기술)
- Swanson K, Wu W, Bulaong NL, Pak JE, Zou J. **The Virtual Lab of AI agents designs new SARS-CoV-2 nanobodies.** *Nature* 646, 716–723 (2025). DOI: 10.1038/s41586-025-09442-9. GitHub: `github.com/zou-group/virtual-lab`.
- Fehlis Y, Mandel P, Crain C, Fuller D, et al. **Accelerating drug discovery with Artificial: a whole-lab orchestration and scheduling system for self-driving labs.** arXiv:2504.00986.
- **Accelerating Drug Discovery Through Agentic AI: A Multi-Agent Approach to Laboratory Automation in the DMTA Cycle** (Tippy). arXiv:2507.09023.
- Ji J, et al. **FROGENT: An End-to-End Full-process Drug Design Multi-Agent System.** arXiv:2508.10760.
- Brace X, et al. **Rhizome OS-1: Rhizome's Semi-Autonomous Operating System for Small Molecule Drug Discovery.** arXiv:2604.07512.
- Suzuki T, et al. **OrchestRA: Orchestrated Rational drug design Agents.** arXiv:2512.21623.
- **Coscientist** (Carnegie Mellon, *Nature* 2023); **A-Lab** (Lawrence Berkeley, *Nature* 2023).
- PNNL **AutoLabs**: "Cognitive multi-agent systems with self-correction for autonomous chemical experimentation", *Sci. Rep.* (2026); GitHub `pnnl/autolabs`.
- 관련 리뷰: *Nature Reviews Chemistry* "The past, present and future of self-driving laboratories" (2026); *Nature Synthesis* "Towards self-driving laboratories in the biopharmaceutical industry" (2026).

### 국내 (보도자료/기관)
- 한국제약바이오협회 — "AI 신약개발 자율실험실" (2025-12 보도).
- 보건복지부 **K-AI 신약개발 전임상·임상 모델개발사업** 총괄기관 선정 (2025-11 보도, 파마시안/메디파나).
- 아이젠사이언스 — K-AI 공동연구개발기관 선정 보도 (뉴스와이어 2025-11).
- **K-MELLODDY** 사업단 공식 사이트: `kmelloddy.org`.
- JW중외제약 — "제이웨이브(JWave) + 합성자동화 연계 자율연구 플랫폼" (약국신문/JW Bioscience).
- IBS/UNIST 그쥐보브스키 연구팀 — "AI·로봇 실험-생성 플랫폼" *Nature* (뉴시스 2025-09).
- "자율실험실 산·학·연 협의체 출범" (지디넷코리아 2026-08).
- 경보제약 — 'AI 기반 ADC 제조 자율랩' 산업부 과제 (아시아경제 2025-11).

---

*본 보고서는 공개 정보를 기반으로 작성되었으며, 정확한 기술 세부사항은 각 기관 논문·보도자료 원문을 참고하시기 바랍니다.*