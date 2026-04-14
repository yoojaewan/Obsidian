# Character Data Schema

## 원칙
- 파츠 에셋 자체는 항상 스케일 1, 그레이스케일 베이스로 제작
- 스케일과 색상은 캐릭터 데이터가 소유하며 런타임에 적용된다
- 색상은 그라디언트 매핑으로 적용 (명암/하이라이트 구조 유지)
- beard는 선택 파츠 — null이면 미착용

## 스키마

```json
{
  "bodyType": "male_normal",
  "faceTemplate": "sharp_01",
  "head": {
    "offset_y": 0,
    "rotation_z": 0.0
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
  "nose": {
    "id": "nose_01",
    "scale": 1.0
  },
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
