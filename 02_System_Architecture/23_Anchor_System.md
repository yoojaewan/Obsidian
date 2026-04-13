# Anchor System

## 원칙
- 좌표는 파츠가 아니라 FaceTemplate이 가진다
- FaceTemplate 이미지에 **약속된 색상으로 1픽셀 마커를 직접 찍는다**
- 눈/코/입 파츠가 덮으므로 렌더링에 노출되지 않는다
- FaceTemplate 이외의 파츠(Hair, Armor, Helmet 등)는 캔버스 중앙 기준 배치

## 핀 좌표 추출 방식
- 툴이 FaceTemplate 이미지 스캔 시 마커 색상을 감지해 핀 좌표를 자동 추출한다

## 마커 색상 규칙
- #FF0000 (빨강) = Eye_L
- #00FF00 (초록) = Eye_R
- #0000FF (파랑) = Nose
- #FFFF00 (노랑) = Mouth
- #FF00FF (자홍) = Hair_Front (Y만 사용)
- #00FFFF (청록) = Hair_Side/Back (Y만 사용)
- #FF8000 (주황) = Helmet (Y만 사용)

## 주요 Anchor
- Eye_L / Eye_R — X, Y 모두 사용
- Nose / Mouth — X, Y 모두 사용
- Hair / Helmet — Y 좌표만 사용, X는 캔버스 중앙 고정

## 규칙
- FaceTemplate 제작 시 마커 색상은 실제 그림에 절대 사용 금지
- Hair, Armor, Helmet은 캔버스 중앙 기준으로 배치
