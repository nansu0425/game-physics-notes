## 1. 개요
탄성 충돌은 두 물체가 충돌한 후에도 운동 에너지가 보존되는 충돌 유형입니다. 이 문서에서는 탄성 충돌의 정의, 수학적 모델, 그리고 게임 물리에서의 적용을 설명합니다.

## 2. 수식 / 핵심 개념
탄성 충돌의 기본 수식은 다음과 같습니다:

1. 운동량 보존 법칙:
   \[
   m_1 v_{1i} + m_2 v_{2i} = m_1 v_{1f} + m_2 v_{2f}
   \]

2. 에너지 보존 법칙:
   \[
   \frac{1}{2} m_1 v_{1i}^2 + \frac{1}{2} m_2 v_{2i}^2 = \frac{1}{2} m_1 v_{1f}^2 + \frac{1}{2} m_2 v_{2f}^2
   \]

여기서 \(m\)은 질량, \(v_i\)는 초기 속도, \(v_f\)는 최종 속도를 나타냅니다.

## 3. 게임 적용 포인트
탄성 충돌은 게임에서 물체 간의 상호작용을 모델링하는 데 필수적입니다. 예를 들어, 공이 벽에 부딪히거나 두 공이 충돌할 때 탄성 충돌 모델을 사용하여 물체의 반응을 계산할 수 있습니다. 이를 통해 현실감 있는 물리적 상호작용을 구현할 수 있습니다.

## 4. 예시 / 의사코드
다음은 탄성 충돌을 처리하는 간단한 의사코드 예시입니다:

```
function handleElasticCollision(body1, body2):
    // 운동량 보존 계산
    v1f = (body1.mass - body2.mass) / (body1.mass + body2.mass) * body1.velocity + (2 * body2.mass) / (body1.mass + body2.mass) * body2.velocity
    v2f = (2 * body1.mass) / (body1.mass + body2.mass) * body1.velocity + (body2.mass - body1.mass) / (body1.mass + body2.mass) * body2.velocity

    body1.velocity = v1f
    body2.velocity = v2f
```

## 5. 튜닝 & Pitfall
탄성 충돌을 구현할 때 주의해야 할 점은 다음과 같습니다:
- 물체의 질량과 초기 속도를 정확히 설정해야 합니다.
- 충돌 후 속도가 비현실적으로 증가하지 않도록 주의해야 합니다.

## 6. 추가 확장
- 비탄성 충돌과의 비교
- 다양한 물체 간의 충돌 시뮬레이션
- 충돌 후 물체의 회전 운동 고려

## 7. 참고
- "Physics for Game Developers" by David M. Bourg
- 관련 논문 및 자료는 docs/references/ 폴더에 추가 예정입니다.