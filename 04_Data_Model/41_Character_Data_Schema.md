# Character Data Schema

## 원칙
- 파츠 에셋 자체는 항상 스케일 1, 그레이스케일 베이스로 제작
- 스케일과 색상은 캐릭터 데이터가 소유하며 런타임에 적용된다
- 색상은 그라디언트 매핑으로 적용 (명암/하이라이트 구조 유지)
- 모든 JSON 필드명은 camelCase (3박자 네이밍 규칙)
- beard / skinMarking / faceOverlay는 선택 파츠 — null이면 미착용
- 좌/우 비대칭 눈은 미지원 (양쪽 동일, 한 PNG를 미러링)
- 비대칭 표현은 faceOverlay로 처리 (안대 등)
- 얼굴 파츠는 FaceTemplate 앵커 좌표를 기본으로 하되, `offsetX`/`offsetY`(픽셀)로 미세 조정 가능
- beard는 X축이 캔버스 중앙 고정이므로 `offsetX` 없음, `offsetY`만 지원
- 모든 파츠에 `scale` 필드 — 일관성. bodyTemplate / faceTemplate 도 객체로 (id + scale)
- `head.scale` 미세 범위 0.9~1.1 권장 (외곽선 ≥2px 전제 안전 마진). SD/귀여운 캐릭터는 별도 face_template asset (큰 캔버스로 머리 크게) 사용
- hair 는 3종(front/side/back) 같이 이동하는 공통 `offsetX`/`offsetY` 보유 (산다라박류 위로 솟는 디자인 보정)
- `blackThreshold` / `whiteThreshold`는 파츠 메타 JSON 소유 (에셋 고유값). 캐릭터 JSON에 두지 않음. 기본값: black=0.04, white=0.8

## 스키마

```json
{
  "nameKey": "character.char_knight_01.name",
  "bodyTemplate": {
    "id": "male_normal",
    "scale": 1.0
  },
  "faceTemplate": {
    "id": "sharp_01",
    "scale": 1.0
  },
  "head": {
    "offsetX": 0,
    "offsetY": -230,
    "scale": 1.0,
    "rotationZ": 0.0
  },
  "hair": {
    "front": "hair_front_03",
    "side": "hair_side_02",
    "back": "hair_back_04",
    "scale": 1.0,
    "offsetX": 0,
    "offsetY": 0,
    "color": "#8B4513"
  },
  "eyes": {
    "id": "eyes_02",
    "scale": 1.1,
    "offsetX": 0,
    "offsetY": 0,
    "color": "#2E8B57"
  },
  "skinMarking": null,
  "nose": {
    "id": "nose_01",
    "scale": 1.0,
    "offsetX": 0,
    "offsetY": 0
  },
  "faceOverlay": null,
  "mouth": {
    "id": "mouth_03",
    "scale": 1.0,
    "offsetX": 0,
    "offsetY": 0
  },
  "beard": null,
  "armor": {
    "id": "armor_01",
    "scale": 1.0,
    "color": "#708090"
  },
  "helmet": {
    "id": "helmet_closed_01",
    "type": "HELM",
    "scale": 1.0,
    "color": "#708090"
  }
}
```

## 필드 상세

### head
- `offsetX` / `offsetY` (정수, 픽셀): 머리(Head 그룹) 의 X/Y 위치. body 캔버스(500×500) 위에 머리 위치 결정. 일반적으로 offsetY 음수 (-230 정도) 로 머리를 body 위쪽에 정렬
- `scale` (float, 0.9~1.1): 머리 크기 미세 조정. 외곽선 ≥2px 전제 시 안전 범위
- `rotationZ` (float, 라디안): 머리 갸웃 회전. Hair는 따라가지 않음 (중력)

### hair
- `front` / `side` / `back` (string): 각각 hair_front/, hair_side/, hair_back/ 폴더의 파츠 id
- `scale` (float): 3종 모두 같이 적용. head.scale 과 곱셈 합성
- `offsetX` / `offsetY` (정수): 3종 모두 같이 이동. 산다라박류 위로 솟는 디자인 보정 등에 사용 (default 0)
- `color` (#RRGGBB): 머리카락 주 컬러

### eyes
- `offsetX` 는 좌우 대칭 동작 — 양수면 양 눈 멀어짐, 음수면 가까워짐 (사이 거리 조절)
- `offsetY` 는 양쪽 같이 위/아래

### bodyTemplate / faceTemplate
- 4-29 결정: 단순 문자열에서 `{id, scale}` 객체로 승격 — 다른 파츠와 구조 일관

### skinMarking / faceOverlay
- 단일 또는 null. scale 필드 보유

### skin_color (미해결 — 추후 추가 예정)
현재 코드는 `DEFAULT_SKIN_COLOR = #E8B894` 상수 사용. FaceTemplate / BodyTemplate / Nose / Mouth는 24_Color_System 상 `skin_color` uniform 사용 대상이지만 캐릭터 JSON 필드 부재. 향후 `head.skinColor` 또는 root `skinColor` 필드로 추가 검토.

## 선택 슬롯 사용 예

### skinMarking (피부 위 표시 — 주근깨/흉터/점/다크서클)
```json
"skinMarking": {
  "id": "freckles_01",
  "color": "#8B4513"
}
```
레이어 위치: FaceTemplate 위, Eyes 아래.

### faceOverlay (눈 위 부착물 — 안대/마스크/외눈안경)
```json
"faceOverlay": {
  "id": "eyepatch_leather_01",
  "side": "left",
  "color": "#2A1810"
}
```
- `side`: `"left"` / `"right"` / `"center"` (마스크 등)
- 레이어 위치: Eyes 위, Nose 아래.
