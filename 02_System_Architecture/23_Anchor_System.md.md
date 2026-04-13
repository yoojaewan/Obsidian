# Anchor System

## 원칙
- 좌표는 파츠가 아니라 FaceTemplate이 가진다
- 파츠는 Anchor + Offset 방식으로 배치한다
- 좌표는 절대값보다 비율 기반 저장을 우선 고려한다

## 주요 Anchor
- Eye_L
- Eye_R
- Nose
- Mouth
- Hair_Front
- Hair_Side
- Hair_Back
- Helmet

## 규칙
- Head size 변화 시 Eye/Nose/Mouth anchor도 함께 이동
- Hair는 기본적으로 scale 고정, 필요시 anchor 위치만 보정