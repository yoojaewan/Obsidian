# Character Data Schema

## 원칙
- 파츠 에셋 자체는 항상 스케일 1, 그레이스케일 베이스로 제작
- 스케일과 색상은 캐릭터 데이터가 소유하며 런타임에 적용된다
- 색상은 그라디언트 매핑으로 적용 (명암/하이라이트 구조 유지)
- 모든 JSON 필드명은 camelCase (3박자 네이밍 규칙)
- beard / skinMarking / faceOverlay는 선택 파츠 — null이면 미착용
- 좌/우 비대칭 눈은 미지원 (양쪽 동일, 한 PNG를 미러링)
- 비대칭 표현은 faceOverlay로 처리 (안대 등)

## 스키마

```json
{
  "nameKey": "character.char_knight_01.name",
  "bodyTemplate": "male_normal",
  "faceTemplate": "sharp_01",
  "head": {
    "offsetY": 0,
    "rotationZ": 0.0
  },
  "hair": {
    "front": "hair_front_03",
    "side": "hair_side_02",
    "back": "hair_back_04",
    "scale": 1.0,
    "color": "#8B4513"
  },
  "eyes": {
    "id": "eyes_02",
    "scale": 1.1,
    "color": "#2E8B57"
  },
  "skinMarking": null,
  "nose": {
    "id": "nose_01",
    "scale": 1.0
  },
  "faceOverlay": null,
  "mouth": {
    "id": "mouth_03",
    "scale": 1.0
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
