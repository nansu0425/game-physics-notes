# 탄성 / 비탄성 충돌 (Elastic & Inelastic Collision)

## 1. 문제 배경 / 직관
게임에서 물체들이 부딪힐 때 현실감과 재미 있는 반응을 만들기 위해 "얼마나 튀어오를지" (탄성) 를 조절해야 합니다. 공이 바닥에 맞고 튀는 정도, 캐릭터가 벽에 부딪힐 때 멈춤/반동, 총알이 표면에 스치며 튀는 모습 등은 모두 충돌 후 속도가 어떻게 변하느냐에 달려 있습니다.

충돌은 크게 두 부분으로 나누어 생각합니다:
- (1) 서로 겹치지 않게 하는 기하학적 처리 (접촉 점, 법선 계산)
- (2) 물리적 반응: 속도를 바꿔주는 순간적인 충격(Impulse) 적용

여기서는 (2)에 집중하며, 1차원(1D)에서 시작해 2차원(2D)/3차원으로 확장하는 일반 공식을 유도하고, **탄성 계수 (Coefficient of Restitution, e)** 로 탄성 정도를 제어하는 방법을 설명합니다. 또한 **비탄성(부분 탄성) 및 완전 비탄성(e=0)** 상황도 다룹니다.

## 2. 핵심 개념 정리
| 용어 | 설명 |
|------|------|
| 충돌 법선(Normal) $\mathbf{n}$ | 접촉점에서 한 물체에서 다른 물체로 향하는 단위벡터 (보통 침투를 밀어내는 방향) |
| 상대 속도(Relative Velocity) $\mathbf{v}_{rel} = \mathbf{v}_B - \mathbf{v}_A$ | 두 물체 속도 차이 |
| 법선 성분 속도 $v_{rel,n} = (\mathbf{v}_B - \mathbf{v}_A)\cdot \mathbf{n}$ | 충돌 반응에 직접적으로 영향을 주는 성분 (탄성은 법선 방향에서만 적용) |
| 탄성 계수(Restitution) e | 튀어오름 정도. e=1 완전 탄성, 0<e<1 부분 탄성, e=0 완전 비탄성 |
| 충격량(Impulse) j | 매우 짧은 시간 동안 작용해 속도를 순식간에 바꾸는 양. 단위: N·s |
| 역질량(Inv Mass) $m^{-1}$ | 질량이 클수록 가속이 적다는 것을 연산적으로 쉽게 처리하기 위해 사용 |

가정(기본):
- 점(질점) 혹은 회전이 무시 가능한 상황 (1D / 축 방향 충돌)부터 시작.
- 마찰 및 회전은 후속 단계에서 분리 처리 (여기선 법선 반응만).
- 시간 dt 동안의 세밀한 힘 적분 대신 순간 Impulse 모델 사용.

## 3. 수식 유도
### 3.1 1D 완전 탄성 충돌 (e = 1)
두 물체 A, B (질량 $m_A, m_B$, 속도 $u_A, u_B$ -> 충돌 후 $v_A, v_B$). 1차원에서 완전 탄성 조건:
1. 운동량 보존: $m_A u_A + m_B u_B = m_A v_A + m_B v_B$
2. 상대 속도 반전: $v_B - v_A = -(u_B - u_A)$

두 식을 연립하면 (고전적 결과):
$$
v_A = \frac{m_A - m_B}{m_A + m_B} u_A + \frac{2 m_B}{m_A + m_B} u_B, \quad
v_B = \frac{2 m_A}{m_A + m_B} u_A + \frac{m_B - m_A}{m_A + m_B} u_B.
$$

### 3.2 1D 일반 (부분 탄성) – 탄성 계수 e 도입
완전 탄성의 상대 속도 반전 조건을 일반화:
$$
v_B - v_A = - e (u_B - u_A), \quad 0 \le e \le 1.
$$
운동량 보존 식은 동일. 연립하여 얻는 충돌 후 속도:
$$
v_A = u_A + \frac{-(1+e) m_B}{m_A + m_B}(u_A - u_B), \qquad
v_B = u_B + \frac{ (1+e) m_A}{m_A + m_B}(u_A - u_B).
$$
위 식을 정리하면 더 익숙한 형태:
$$
v_A = \frac{m_A - e m_B}{m_A + m_B} u_A + \frac{(1+e) m_B}{m_A + m_B} u_B, \quad
v_B = \frac{(1+e) m_A}{m_A + m_B} u_A + \frac{m_B - e m_A}{m_A + m_B} u_B.
$$

### 3.3 Impulse(충격량) 관점 (게임 구현 친화)
1D에서 상대 속도의 법선 성분을 $v_{rel,n} = u_B - u_A$ 라 하면, 충돌 후 원하는 법선 상대 속도:
$$
v'_{rel,n} = - e \, v_{rel,n}.
$$
속도 변화량(법선 방향) 조건:
$$
\Delta v_{rel,n} = v'_{rel,n} - v_{rel,n} = -(1+e) v_{rel,n}.
$$
Impulse j 적용 시 (각 물체 속도 변화: $\Delta u = \pm j / m$):
$$
\Delta v_{rel,n} = -\frac{j}{m_A} - \frac{j}{m_B} = - j \left(\frac{1}{m_A} + \frac{1}{m_B}\right).
$$
따라서
$$
j = \frac{(1+e) v_{rel,n}}{\frac{1}{m_A} + \frac{1}{m_B}} = (1+e) \frac{v_{rel,n}}{m_A^{-1} + m_B^{-1}}.
$$
그러나 구현에서는 **접근 중인지 검사**가 필요: 만약 $v_{rel,n} < 0$ (서로 접근) 일 때만 반응. (법선 방향 정의에 따라 부호 주의; 아래 2D 파트에서 일반화.)

### 3.4 2D (및 3D) 로 일반화
충돌 반응은 **법선 방향 성분만** 수정. 접선(마찰 제외 시) 성분은 유지. 두 물체 속도 $\mathbf{v}_A, \mathbf{v}_B$, 단위 법선 $\mathbf{n}$:
$$
v_{rel,n} = (\mathbf{v}_B - \mathbf{v}_A)\cdot \mathbf{n}.
$$
접근 중인지 검사: $v_{rel,n} > 0$ 이면 서로 멀어짐 → 충돌 처리 생략. (여기서는 $\mathbf{n}$ 을 A → B 방향으로 가정.)

Impulse 스칼라 값:
$$
j = - (1 + e) \frac{v_{rel,n}}{m_A^{-1} + m_B^{-1}}.
$$
속도 갱신:
$$
\mathbf{v}_A' = \mathbf{v}_A - j \; m_A^{-1} \mathbf{n}, \quad
\mathbf{v}_B' = \mathbf{v}_B + j \; m_B^{-1} \mathbf{n}.
$$

완전 비탄성($e=0$)일 경우 법선 상대 속도는 0이 되므로 두 물체는 법선 방향 상대 운동을 잃고(끈적하게 붙는 듯한 효과) 접선 운동만 남습니다.

### 3.5 에너지 관점
완전 탄성: 운동 에너지 총합 보존.  
부분 탄성: 손실된 에너지 비율은 $1 - e^2$ 와 관련 (1D 두 물체 특수 경우).  
실제 게임에서는 너무 높은 e 값이 수치 오차와 겹쳐 에너지 폭주처럼 보일 수 있어 0.0~0.9 사이 제한하거나, 상대 속도가 작을 때 e 를 낮추는 "velocity threshold" 기법을 사용합니다.

## 4. 이산화 (시뮬레이션 관점)
프레임 단위 시뮬레이션에서는 힘 적분 대신 충돌 시점에 즉시 속도를 조정하는 Impulse 모델을 사용합니다.

절차(단일 접점):
1. Broad/Narrow Phase 로 접촉(침투 깊이, 법선, 접촉점) 계산.
2. 침투 깊이가 0 이하이면 충돌 없음.
3. 상대 속도 법선 성분 계산, 분리 중이면 조기 반환.
4. 탄성 계수 e (두 물체 값을 혼합: 보통 $e = \max(e_A, e_B)$ 또는 평균) 결정.
5. 위 공식으로 j 계산.
6. 속도 갱신.
7. 선택: 위치 보정 (penetration correction) 로 침투 제거.

시간 보간(ToI 기반 CCD)을 하지 않으면 빠른 물체가 관통(tunneling) 할 수 있으므로 필요 시 CCD 알고리즘(GJK+TOI 등)과 결합.

## 5. C++ 구현 예시
아래 코드는 2D 질점 충돌(회전/마찰 제외) 처리 예시입니다.

```cpp
struct Vec2 {
	double x, y;
	Vec2 operator+(const Vec2& o) const { return {x+o.x, y+o.y}; }
	Vec2 operator-(const Vec2& o) const { return {x-o.x, y-o.y}; }
	Vec2 operator*(double s) const { return {x*s, y*s}; }
};

inline double Dot(const Vec2& a, const Vec2& b){ return a.x*b.x + a.y*b.y; }
inline Vec2 Normalize(const Vec2& v){ double l = std::sqrt(Dot(v,v)); return (l>0)? v*(1.0/l) : Vec2{0,0}; }

struct Body {
	Vec2 pos;
	Vec2 vel;
	double invMass;      // 0 => 무한 질량(고정체)
	double restitution;  // 0~1 권장
};

struct Contact {
	Body* A;
	Body* B;
	Vec2 normal;      // 단위 벡터 (A->B)
	double penetration; // 양수면 겹침 깊이
};

void ResolveCollision(Contact& c) {
	Body& A = *c.A; Body& B = *c.B;
	if (A.invMass + B.invMass == 0.0) return; // 둘 다 고정체

	// 1. 상대 속도
	Vec2 rv = { B.vel.x - A.vel.x, B.vel.y - A.vel.y };
	double velAlongNormal = Dot(rv, c.normal);

	// 2. 분리 중이면 반환 (B가 A에서 멀어지는 중)
	if (velAlongNormal > 0.0) return;

	// 3. Restitution 혼합 (여기서는 최댓값 사용)
	double e = std::max(A.restitution, B.restitution);

	// 4. Impulse 크기 j 계산
	double invMassSum = A.invMass + B.invMass;
	double j = -(1.0 + e) * velAlongNormal / invMassSum;

	// 5. 속도 갱신
	Vec2 impulse = c.normal * j;
	A.vel.x -= impulse.x * A.invMass;
	A.vel.y -= impulse.y * A.invMass;
	B.vel.x += impulse.x * B.invMass;
	B.vel.y += impulse.y * B.invMass;

	// 6. 위치 보정 (침투 제거, Baumgarte/분리형 간단 버전)
	const double percent = 0.2;   // 보정 비율
	const double slop    = 0.01;  // 허용 여유
	double correctionMag = std::max(c.penetration - slop, 0.0) / invMassSum * percent;
	Vec2 correction = c.normal * correctionMag;
	A.pos.x -= correction.x * A.invMass;
	A.pos.y -= correction.y * A.invMass;
	B.pos.x += correction.x * B.invMass;
	B.pos.y += correction.y * B.invMass;
}
```

### 5.1 1D 전용 단순 함수 (디버깅용)
```cpp
struct Body1D { double v; double invM; double restitution; };

void Collide1D(Body1D& A, Body1D& B) {
	if (A.invM + B.invM == 0.0) return;
	double rel = B.v - A.v; // A->B 방향
	if (rel <= 0.0) return; // 이미 멀어지는 중이면 패스 (법선 정의에 따라 조정 가능)
	double e = std::max(A.restitution, B.restitution);
	double j = -(1.0 + e) * rel / (A.invM + B.invM);
	A.v -= j * A.invM;
	B.v += j * B.invM;
}
```

## 6. 수치 안정성 / 에러 분석
문제와 완화 전략:

| 이슈 | 증상 | 원인 | 대응 |
|------|------|------|------|
| 침투 잔존 | 얇은 객체 관통 | 큰 dt, 빠른 속도 | CCD, 더 작은 dt, 위치 보정 percent ↑ |
| 에너지 증가 | 점프 반복 시 더 높게 | 높은 e + 반올림 오차 | e 상한(<=0.9), 작은 상대 속도에서 e=0 처리 |
| 떨림(Jitter) | 정지해야 할 물체 미세 진동 | 과도한 보정 / 정밀도 손실 | slop 증가, 보정 percent 감소 |
| NaN 속도 | 수치 폭발 | 0 질량 처리 누락, 잘못된 법선 | invMass=0 분기, 법선 재정규화 |
| 스택 붕괴 | 블록 쌓기 불안 | 순차 충돌 순서 편향 | 반복(Iterations) 증가, Warm Starting |

추가 팁:
- 매우 작은 침투는 무시(슬롭)해 불필요한 반응 최소화.
- 상대 속도 절대값이 임계값 이하일 때 e=0 (마찰적인 충돌처럼) 처리하면 안정.
- 작은 질량 대비 큰 질량 충돌에서 극단적 속도 변화 → 최대 임펄스 캡.

## 7. 확장 / 변형 기법
- 마찰(Friction) 추가: 접선 방향 상대 속도 계산 → 탄젠트 벡터 → 쿠롱 마찰 제한(클램프) 임펄스 적용.
- 회전 포함: 각속도, 관성 모멘트, 접촉점 레버 암($r \times n$) 포함하여 유효 질량(Effective Mass) 계산.
- Restitution Mixing 전략: $e = \max$, 평균, 또는 물리 재료 테이블 기반.
- Warm Starting: 이전 프레임 임펄스를 캐시해 수렴 빠르게.
- Split Impulse: 위치 보정이 에너지(속도)에 영향 주지 않도록 별도 통로.
- CCD(Time of Impact): 빠른 탄환 터널링 방지.
- Energy Clamping: 반동 후 에너지가 이전 + 허용 오차 초과하면 축소.

## 8. 참고 자료
- Erin Catto, GDC Physics Tutorials (Iterative Dynamics, Restitution, Warm Starting).
- David H. Eberly, "Game Physics".
- Ian Millington, "Game Physics Engine Development".
- Chris Hecker, "Rigid Body Dynamics" Articles.
- Box2D Lite 소스 코드 (충돌 임펄스 구현 참고).

---
요약: 충돌 반응은 "법선 방향 상대 속도"를 원하는 값($-e$ 배)으로 재설정하는 임펄스 문제로 단순화할 수 있습니다. 1D 공식을 이해하면 2D/3D 는 법선 투영 + 벡터 연산으로 확장됩니다.
