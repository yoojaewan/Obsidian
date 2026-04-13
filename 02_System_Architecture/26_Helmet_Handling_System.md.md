# Helmet Handling System

## Helmet Types

| 타입 | 헤어 규칙 | 얼굴 파츠 hide | 예시 |
|---|---|---|---|
| TIARA | NONE | 없음 | 티아라, 왕관, 머리띠 |
| HAT | SHOW | 없음 | 모자, 두건 |
| MASK | SHOW | nose, mouth, beard | 마스크, 복면 |
| HELM | HIDE / SWAP | nose, mouth, beard | 갑옷 투구 |

## 처리 원칙
- Helmet은 단순 이미지가 아니라 룰 오브젝트다
- 타입에 따라 Hair 레이어와 Face 파츠를 제어한다
- 모든 Hair × Helmet 개별 대응은 금지
- Hair category/tag 기반 규칙 처리 우선

## Hair 규칙 상세

NONE (TIARA)
- Hair 레이어 변경 없음

SHOW (HAT)
- Hair 유지

HIDE / SWAP (HELM)
- Hair_Back 숨김
- Hair_Side 일부 숨김
- 필요시 Hair_Front helmetfit 버전으로 swap

## Face 파츠 규칙 상세

TIARA / HAT
- 모든 얼굴 파츠 표시 유지

MASK / HELM
- nose 숨김
- mouth 숨김
- beard 숨김 (있을 경우)
