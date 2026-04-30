# Anchor System

## 원칙
- 좌표는 파츠가 아니라 FaceTemplate이 가진다
- FaceTemplate 이미지에 **약속된 색상으로 1픽셀 마커를 직접 찍는다**
- 눈/코/입 파츠가 덮으므로 렌더링에 노출되지 않는다
- FaceTemplate 이외의 파츠(Hair, Armor, Helmet 등)는 캔버스 중앙 또는 별도 컨벤션 기준 배치

## FaceTemplate 캔버스 사이즈

**FaceTemplate = 200×200 고정** (2026-04-30 확정).

- 머리 일러스트가 캔버스 거의 전체를 차지
- body_template (500×500) 과 합성 시 머리:어깨 ≈ 1:2.5 사실 비율
- 캔버스 사이즈 변경 금지 — 마커 좌표가 절대 픽셀이라 사이즈 바뀌면 합성 깨짐

## 핀 좌표 추출 방식
- 툴이 FaceTemplate 이미지 스캔 시 마커 색상을 감지해 핀 좌표를 자동 추출
- 코드: 좌상→우하 픽셀 스캔, 각 마커 색에 대해 매칭되는 첫 픽셀 사용 (1픽셀 마커 사양 전제)

## 마커 색상 규칙 (5종 — 2026-04-29 통합)

| 마커 | RGB | HEX | 용도 |
|---|---|---|---|
| eye_l | (255, 0, 0) | `#FF0000` 빨강 | 왼쪽 눈 |
| eye_r | (0, 255, 0) | `#00FF00` 초록 | 오른쪽 눈 |
| nose | (0, 0, 255) | `#0000FF` 파랑 | 코 |
| mouth | (255, 255, 0) | `#FFFF00` 노랑 | 입 |
| headwear | (255, 0, 255) | `#FF00FF` 마젠타 | 정수리 — Hair(F/S/B) + Helmet 공용 baseline |

이전에 hair_front/hair_side·back/helmet 별로 3종 마커가 있었으나 모두 Y만 사용하고 캐릭터별 fine-tune이 들어가므로 **단일 정수리 마커(headwear)** 로 통합 (4-29).

## 마커 의미 (2026-04-30 명시)

마커 = **부위 PNG 캔버스 중심이 와야 할 위치**. 해부학적 중심 (눈동자/콧대/입술) 이 아니라 *정렬 기준점*. 코드는 Sprite2D `centered=true` 라 PNG 캔버스 중심을 마커 좌표에 둘 뿐. 사용자가 face_template에 마커 박을 때 *"이 위치에 부위 PNG 통째로 와야 한다"* 로 판단.

## 200×200 face_template 마커 좌표 (좌상단 = 0,0 기준)

`asset-generater/assets/parts/face_template/sharp_01.png` 측정값 (참고/시작점):

| 마커 | 절대 좌표 (x, y) | 캔버스 중심 기준 (x, y) |
|---|---|---|
| eye_l | (73, 87) | (-27, -13) |
| eye_r | (126, 87) | (+26, -13) |
| nose | (100, 110) | (0, +10) |
| mouth | (100, 136) | (0, +36) |
| headwear | (101, 9) | (+1, -91) |

이 좌표는 sharp_01 디자인 기준. 다른 face_template 만들 때는 그 일러스트의 실제 정수리/눈/코/입 위치에 맞춰 박을 것.

## 주요 Anchor
- eye_l / eye_r — X, Y 모두 사용. 좌우 대칭(양 눈 거리 조절)으로 처리됨 (캐릭터 JSON `eyes.offsetX > 0` → 양쪽 멀어짐)
- nose / mouth — X, Y 모두 사용
- headwear — Y는 모든 hair / helmet 의 baseline. X는 캔버스 중앙 고정 또는 hair.offsetX 적용

## Hair 좌표계 (편집 트리 보정)

**Hair는 Head 그룹 밖** (회전 영향 없음 — 22_Layer_System 중력 표현). 그러나 정수리 마커(headwear)는 face_template 안에 있어 Head 좌표계 종속.

→ 코드는 hair 위치 계산 시 Head 의 `offsetY` / `scale` 만 부분 보정 (회전 제외). 회전은 머리만 갸웃하고 머리카락은 중력 따름:

```
hair_world_y = head.offsetY + (face local headwear_y) * head.scale + hair.offsetY
hair_world_x = hair.offsetX
```

## hair PNG 정렬 컨벤션 (2026-04-30 추가)

다른 부위와 달리 hair는 **캔버스 상단(Y=0) = 정수리** 정렬. 대부분 디자인이 정수리부터 아래로 흘러내리므로 디폴트 시작점이 직관적.

코드: `Sprite2D.offset.y = height/2` 로 텍스처 상단을 sprite position(=정수리 마커)에 정렬. centered=true 유지 → scale 기준점도 정수리.

위로 솟구치는 디자인(산다라박류)은 캐릭터 JSON `hair.offsetY` 음수 보정 필요. 디폴트 정렬에서는 캔버스 위쪽이 잘림.

## 규칙
- FaceTemplate 제작 시 마커 색상은 실제 그림에 절대 사용 금지 (셰이더가 마커 픽셀도 외곽선/색칠 대상으로 처리할 위험)
- 마커는 **1px 정확** + **alpha 1.0** + **anti-aliasing OFF** + **opacity 100%** + **정확한 RGB 값** (245나 250 같은 근사값 X)
- Hair는 캔버스 상단=정수리 컨벤션, 다른 부위는 캔버스 중심=마커 컨벤션
- Armor는 캔버스 중앙 기준 배치 (body_template 500×500 기준)
