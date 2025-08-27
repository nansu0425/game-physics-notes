# Game Physics Notes

게임 프로그래밍에서 필요한 물리(Physics) 이론 · 기법 · 구현 아이디어를 체계적으로 정리하고, 반복 가능한 실험/코드 스니펫/메모를 축적하기 위한 저장소입니다. 이 저장소는 **정확한 개념 정리 + 실용적인 구현 관점 + 성능/안정성 트레이드오프**를 동시에 추구하며, 문서 작성과 아이디어 확장을 위해 **AI 도구를 적극 활용**합니다.

---
## 🎯 저장소 핵심 목표
1. 게임 물리의 핵심 토픽을 폭넓고 (필요 시) 깊게 구조화된 문서로 축적
2. 각 개념마다: 수학적 정의 → 직관적 설명 → 게임 적용 포인트 → 구현/최적화 팁 → 참고 문헌 순 흐름 유지
3. 실험 가능 코드(프로토타입)와 파생 노트 연결성 확보
4. AI가 문맥을 쉽게 이해하고 관련 답변/요약/확장을 돕도록 **메타데이터 + 일관 템플릿** 제공
5. 점진적 개선(Incremental refinement) 기록: 초안 → 검수 → 확정

---
## 📌 범위 (Scope)
포함 (In-Scope):
- 고전역학 기초 (질량-힘-가속도, 에너지, 운동량, 관성 모멘트)
- 수치적분 (Explicit/Implicit Euler, Semi-Implicit, Verlet, RK 계열, Position Based Dynamics 등)
- 충돌 검출 (Broad Phase, Narrow Phase, SAT, GJK, EPA, Spatial Partitioning)
- 충돌 반응 & 해결 (Impulse, Sequential Impulse, Constraint Solvers)
- 강체(Rigid Body) / 관절(Joint) / 제약(Constraint)
- 연성체(Soft Body), Cloth, Particle Systems, Fluid(기초)
- 안정성 & 튜닝 (떨림, 침투, 에너지 폭주 방지)
- 고정 시간 스텝 / 가변 시간 스텝 처리
- 네트워크 동기화(예: Client-side Prediction, Reconciliation)에서의 물리 고려
- 성능 최적화 (SIMD 개요, 캐시 친화 데이터 배치, 프로파일링 포인트)

제외 (Out-of-Scope / 별도 저장소 권장):
- 렌더링/셰이딩 세부 기술
- 일반 AI/머신러닝 기법 (단, 물리 파라미터 추정에 쓰일 때는 예외)
- 사운드 물리 (음향 전파 등)

---
## 🗂️ 제안 디렉터리 구조 (초기 가이드)
```
docs/                  # 개념 · 이론 · 정리 문서 (Markdown)
	fundamentals/        # 물리 기초
	dynamics/            # 운동학/동역학/적분
	collision/           # 충돌 검출/반응
	constraints/         # 제약/관절
	softbody/            # 연성체/Cloth/Fluid 기초
	optimization/        # 성능/안정성/튜닝
	networking/          # 네트워크 물리 동기화
	references/          # 참고 문헌/논문 요약

experiments/           # 실험/프로토타입 코드 + README (실행 방법)
	<topic-name>/
		notes.md
		impl/              # 간단 구현 (언어 자유: C++, Rust, Python, etc.)

snippets/              # 재사용 가능한 소규모 코드 조각
assets/                # 도식, 이미지, GIF, 수식 렌더 결과
meta/                  # 템플릿, 가이드, 태그 사전
```
실제 사용하면서 필요에 따라 조정/확장합니다.

---
## 🧩 문서 파일 네이밍 & 분할 원칙
- 한 파일은 하나의 “핵심 질문” 또는 “단일 명확한 개념”에 집중 (SRP)
- 파일명: `topic-subtopic-shortslug.md` (영문 소문자, 하이픈 구분)
- 너무 길어지면 `part-1`, `part-2` 등 분리
- 복합 개념 개요 vs 세부 파생: `overview` / `details` 명시 가능

---
## 🏷️ 메타데이터 (Front Matter) 규칙
모든 주요 문서는 상단에 YAML Front Matter 포함:
```yaml
---
title: Rigid Body Dynamics Basics
slug: rigid-body-basics
topics: [rigid-body, inertia, integration]
level: intermediate        # intro | intermediate | advanced
status: draft              # draft | review | stable
last_update: 2025-08-28
summary: 관성 모멘트와 회전 운동 방정식의 게임 물리 적용 개요.
refs: [featherstone, box2d, bullet]
ai_prompt_hint: "질문이 오면 에너지 보존 여부와 수치 안정성 고려를 함께 설명해줘."
---
```
AI 프롬프트 품질 향상을 위해 `summary`, `ai_prompt_hint` 필드를 적극 활용합니다.

상태 워크플로:
`draft` → (자가 1차 검수) → `review` → (수정/보강) → `stable`

---
## 🤖 AI 활용 가이드
AI에게 문맥을 제대로 전달하기 위한 기본 System Prompt 예시:
```
당신은 게임 물리 엔지니어 어시스턴트입니다. 이 저장소는 게임 물리 이론을
실용적으로 정리하고, 구현·성능·안정성 관점을 함께 다룹니다.
답변 시:
1) 용어를 엄밀히 정의하고
2) 필요하면 간단 수식(LaTeX) 첨부
3) 게임 개발 적용 주의점(성능, 튜닝, 수치 안정성) 언급
4) 관련 문서 slug 나열 (가능하면 링크 형태)
5) 불확실하면 가정 명시
```

프롬프트 패턴:
- 확장: "다음 초안 파일을 보강: <요약 붙여넣기> 기준으로 더 깊은 예시 2개 추가"
- 교정: "이 문서에서 개념적 오류/용어 혼동 포인트 목록화"
- 연결: "A 문서와 B 문서의 겹치는 개념과 차이점 표로 정리"
- 실험 제안: "이 이론의 수치적 거동 검증 위한 최소 실험 시나리오 3개 생성"

주의:
- AI가 제시한 수식/알고리즘은 가능하면 간단한 수치 테스트로 검증 기록
- 과도한 추상화 대신 구체적 파라미터/경계조건 명시

---
## 🧪 실험(Experiments) 문서 구조 제안
`experiments/<topic>/README.md` 또는 `notes.md` 형식:
1. 목적 (Hypothesis / 질문)
2. 관련 개념/참조 문서
3. 환경 & 파라미터 (dt, solver, iteration count 등)
4. 절차
5. 결과 (수치/스크린샷/GIF)
6. 해석 & 한계
7. 다음 개선 아이디어

---
## 🧱 코드 스니펫 스타일
- 프로그래밍 언어로 C++ 사용
- 물리적 단위 명확 (`// unit: meters`, `// seconds` 주석)
- 매직 넘버 → 상수 (`const float EPSILON = 1e-5f;`)
- 가능하면 결정적(deterministic) 재현 (seed 고정)
- 한 스니펫은 한 개념 (예: "Impulse Resolution (2D) 기본 형태")

---
## 📝 신규 문서 템플릿 (복사해서 사용)
```markdown
---
title: <Title>
slug: <kebab-slug>
topics: [ ]
level: intro
status: draft
last_update: 2025-08-28
summary: <한두 문장 요약>
refs: [ ]
ai_prompt_hint: ""
---

## 1. 개요
핵심 정의 및 문제 맥락.

## 2. 수식 / 핵심 개념
필요 수식 + 직관 해설.

## 3. 게임 적용 포인트
실용적 고려(성능/정확도/안정성).

## 4. 예시 / 의사코드
간단 구현 or pseudo.

## 5. 튜닝 & Pitfall
실수 사례 / 수치 폭주 조건.

## 6. 추가 확장
관련 심화 주제.

## 7. 참고
문헌 / 엔진 구현 레퍼런스.
```

---
## 📚 참고 리소스 (초기 셋)
- Box2D / Erin Catto GDC 강연 자료
- Bullet Physics Paper & 구현 노트
- Chris Hecker Rigid Body Articles
- David Baraff 강의 노트 (An Introduction to Physically Based Modeling)
- GJK / EPA 원 논문
- Position Based Dynamics (Müller et al.)
- Featherstone: Rigid Body Dynamics Algorithms

`docs/references/` 폴더에 논문 요약 및 구현 주의점을 점진적으로 추가합니다.

---
## 🔄 버전 관리 & 변경 로그
- 큰 구조 변화나 개념 재작성 시 `meta/CHANGELOG.md` 생성/갱신 권장
- 실험 재현성 깨뜨리는 변경 시 명시적인 주석 추가

---
## ❓ AI에게 질의할 때 권장 문장
"이 저장소의 문서 구조와 메타데이터 규칙을 따르면서, Semi-Implicit Euler와 Verlet 차이점을 안정성/성능/에너지 보존 관점 표로 만들어줘. 관련 slug 후보도 제안해줘."

---
## 🙋 문의 / 메모
내부 실험 중 떠오르는 아이디어나 TODO는 `meta/quick-notes.md`(빠른 덤프) → 추후 구조화된 문서로 승격.

---
이 README는 저장소 *문맥(높은 레벨 목적 + 구조 + AI 상호작용 규칙)* 을 AI와 사람이 모두 빠르게 파악할 수 있도록 설계되었습니다. 필요 시 언제든지 개선합니다.
