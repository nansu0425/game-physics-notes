---
title: Elastic Collision Basics
slug: elastic-collision-basics
topics: [collision, impulse, momentum, restitution]
level: intro
status: draft
last_update: 2025-08-28
summary: 1D/2D 탄성 충돌의 운동량/에너지 보존, 반발계수=1 가정 하 Impulse 계산과 게임 물리 적용 개요.
refs: [box2d, erin-catto, klasik-mechanics]
ai_prompt_hint: "충돌 질문이 오면: (1) 상대속도 법선 성분 계산 → (2) 탄성( e=1 ) 혹은 일반 e 적용 → (3) 질량/관성 고려한 impulse → (4) 에너지/속도 클램프 안정성 체크 순으로 설명해줘."
---

## 1. 개요
탄성 충돌(perfectly elastic collision)은 계(system)의 총 선운동량(momentum)과 총 운동에너지(kinetic energy)가 모두 보존되는 이상적 충돌을 말한다. 게임 물리에서는 대부분 완전 탄성보다는 부분 탄성(반발계수 e<1)을 사용하지만, 이 문서는 e=1인 기초 공식을 정리한 뒤 일반화(e≤1) 방향을 함께 언급한다.

핵심 질문:
- 두 물체 질량과 초기 속도를 알 때 충돌 후 속도를 어떻게 계산하는가?
- 1차원(충돌 법선 방향) 문제로 축소하는 일반 절차는?
- 수치 안정성과 부동소수점 오차를 어떻게 관리하는가?

## 2. 수식 / 핵심 개념
### 2.1 보존 법칙 (1D)
두 물체 질량 \(m_1, m_2\), 충돌 전 (법선 방향 성분) 속도 \(u_1, u_2\) → 충돌 후 \(v_1, v_2\).

1) 선운동량 보존:
\[
m_1 u_1 + m_2 u_2 = m_1 v_1 + m_2 v_2
\]
2) 운동에너지 보존 (탄성):
\[
	frac{1}{2} m_1 u_1^2 + \tfrac{1}{2} m_2 u_2^2 = \tfrac{1}{2} m_1 v_1^2 + \tfrac{1}{2} m_2 v_2^2
\]

해를 풀면 (표준 결과):
\[
v_1 = \frac{(m_1 - m_2) u_1 + 2 m_2 u_2}{m_1 + m_2}, \qquad
v_2 = \frac{2 m_1 u_1 + (m_2 - m_1) u_2}{m_1 + m_2}
\]

### 2.2 Impulse 관점 (1D 법선 축)
법선 단위 벡터 \(\mathbf{n}\), 상대속도 \(v_r = (\mathbf{v}_2 - \mathbf{v}_1) \cdot \mathbf{n}\) (충돌 직전). 탄성 (\(e=1\)) 에서 필요한 속도 변화 \(\Delta v\) 는 상대속도의 법선 성분을 완전히 반전.

Coefficient of Restitution (일반화):
\[
e = - \frac{ (\mathbf{v}_2' - \mathbf{v}_1') \cdot \mathbf{n} }{ (\mathbf{v}_2 - \mathbf{v}_1) \cdot \mathbf{n} }
\]
완전 탄성: \(e = 1\).

선형 Impulse \(J\) (스칼라):
\[
J = -\frac{(1+e)\, (\mathbf{v}_2 - \mathbf{v}_1)\cdot \mathbf{n}}{1/m_1 + 1/m_2}
\]
탄성 (\(e=1\)):
\[
J_{e=1} = -\frac{2 (\mathbf{v}_2 - \mathbf{v}_1)\cdot \mathbf{n}}{1/m_1 + 1/m_2}
\]

속도 갱신:
\[
\mathbf{v}_1' = \mathbf{v}_1 + \frac{J}{m_1} \mathbf{n}, \qquad
\mathbf{v}_2' = \mathbf{v}_2 - \frac{J}{m_2} \mathbf{n}
\]

### 2.3 2D / 3D 일반 절차
1. 충돌 법선 n (정규화) 계산
2. 상대속도 relVel = v2 - v1
3. relNormal = relVel · n
4. 분리(separating) 상태(relNormal > 0)이면 충돌 처리 스킵
5. J 계산 (위 식)
6. 각 물체 속도 업데이트

### 2.4 회전(Angular) 포함 간단 확장
접촉점 \(p\) 에서의 상대속도는 선속도 + 각속도에 의한 접선 성분 포함:
\[
\mathbf{v}_a = \mathbf{v}_{\text{lin},a} + \boldsymbol{\omega}_a \times \mathbf{r}_a
\]
충돌 법선 성분에 대한 유효 질량 (Effective Mass) \(M_{\text{eff}}\) 는 관성텐서 반영:
\[
M_{\text{eff}} = \frac{1}{\left(\frac{1}{m_1}+\frac{1}{m_2}\right) + \Big( ((I_1^{-1}(\mathbf{r}_1 \times \mathbf{n})) \times \mathbf{r}_1) \cdot \mathbf{n} \Big) + \Big( ((I_2^{-1}(\mathbf{r}_2 \times \mathbf{n})) \times \mathbf{r}_2) \cdot \mathbf{n} \Big)}
\]
\(J\) 식 분모를 \(M_{\text{eff}}\) 로 교체 → 회전 포함 정확한 충돌 반응.

### 2.5 에너지 검증 (선택)
계산 후 총 운동에너지 합을 비교하여 큰 오차(예: 상대적 1e-4 이상) 발생 시 보정(재정규화) 가능.

## 3. 게임 적용 포인트
- 완전 탄성은 종종 비현실(너무 튀는 느낌). e=0.2~0.8 범위 현실감 ↑.
- 프레임 별 침투(penetration) 보정(Position Correction, Baumgarte 등)과 분리된 단계 → Impulse 계산 단순화.
- 작은 질량 vs 큰 질량 (m1 << m2)에서는 수치적 분모 안정성 체크 (1/m1 항이 지배적).
- 부동소수점 누적 오차로 인해 운동에너지 미세 증가(energy drift) 가능 → 주기적 damping (매우 작게) 또는 재정규화.
- 고속 물체: Tunneling → 연속 충돌 검출(TOI / CCD) 필요 (탄성 공식 자체보다 검출이 관건).

## 4. 예시 / 의사코드 (C++ 스타일)
```cpp
struct Body {
    Vec2 pos;
    Vec2 vel;
    float mass;   // > 0, 무한 질량은 별도 플래그
};

void ResolveElasticCollision(Body& A, Body& B, const Vec2& normal /*정규화*/, float restitution /*e*/) {
    // 1. 상대속도 법선 성분
    float relNormal = Dot(B.vel - A.vel, normal);
    if (relNormal > 0.0f) return; // 이미 멀어짐

    // 2. 유효 질량 (무한 질량은 0 기여) 
    float invA = (A.mass > 0) ? 1.0f / A.mass : 0.0f;
    float invB = (B.mass > 0) ? 1.0f / B.mass : 0.0f;

    // 3. Impulse 크기
    float J = -(1.0f + restitution) * relNormal / (invA + invB);

    // 4. 속도 갱신
    A.vel -= (J * invA) * normal; // 음수? 위 relNormal<0라 J>0 → OK
    B.vel += (J * invB) * normal;

    // 5. 안정성 수치 클램프 (선택)
    // if (Length(A.vel) > MAX_V) A.vel = Normalize(A.vel) * MAX_V;
}
```
핵심: normal은 접촉 지오메트리에서 결정(예: 원 vs 원: 방향 = (B.pos - A.pos)).

## 5. 튜닝 & Pitfall
| 이슈 | 설명 | 대응 |
|------|------|------|
| 침투 발생 | 프레임 단위 이산 샘플링 | 위치 보정 단계(분리) 추가 |
| 고속 통과 | CCD 미적용 | Sweep 테스트, TOI 처리 |
| 에너지 Drift | 누적 floating error | 미세 damping, 주기적 재정규화 |
| e > 1 사용 | 게임적 과장으로 종종 사용 | 의도적이면 문서화, 네트워크 동기화 주의 |
| 매우 작은/큰 질량비 | 분모 민감 | 최소/최대 질량 클램프 또는 스케일링 |

## 6. 추가 확장
- 부분 탄성(e<1) 마찰(tangential impulse) 결합 처리
- 연쇄 충돌(다수 접촉)에서 Sequential Impulse 반복
- 회전 포함 2D 박스 vs 박스 (Contact Manifold)
- 에너지 보존 검증 및 Drift 보정 알고리즘 비교(Log, PBD 스타일) 

## 7. 참고
- Erin Catto, Box2D GDC Presentations (Impulse-based resolution)
- David Baraff, "An Introduction to Physically Based Modeling" (Collision response)
- Chris Hecker, Rigid Body Articles (Game Developer Magazine)
- Classical Mechanics (Taylor / Goldstein) – 1D elastic collision formula derivation

---
문서 상태: draft. 개선 아이디어(예: 회전 포함 예제 코드, CCD 링크)를 `meta/quick-notes.md`에 기록 후 추후 확장.
