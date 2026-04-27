# Layer System

## 기본 레이어 순서
1. BodyTemplate
2. FaceTemplate
3. Nose
4. Mouth
5. SkinMarking  <!-- 주근깨/흉터/점/다크서클. Nose/Mouth 위, Eyes/Beard 아래. 단일 또는 null -->
6. Eyes  <!-- 눈썹 포함, 눈+눈썹 세트로 하나의 파츠 -->
7. Beard
8. Hair_Back
9. FaceOverlay  <!-- 안대/마스크/외눈안경. Hair_Back 위, Hair_Side/Hair_Front 아래. 단일 또는 null -->
10. Hair_Side
11. Hair_Front
12. Armor
13. Helmet_Back
14. Helmet_Front

## Godot 씬 그룹 구조

```
Root
├── Hair_Back      ← Head 그룹 밖 (중력, 회전 없음)
├── FaceOverlay    ← Head 그룹 밖 (Hair_Back 위, Hair_Side 아래). 선택 (null 허용)
├── Hair_Side      ← Head 그룹 밖
├── Hair_Front     ← Head 그룹 밖
│
├── Head (Node2D)  ← rotation_z, offset_y 적용 대상
│   ├── BodyTemplate
│   ├── FaceTemplate
│   ├── Nose
│   ├── Mouth
│   ├── SkinMarking ← 선택 (null 허용). Nose/Mouth 위, Eyes/Beard 아래
│   ├── Eyes
│   ├── Beard
│   └── Helmet     ← 머리와 함께 움직이므로 그룹 안
│
└── Armor          ← 몸통, 그룹 밖
```

## 원칙
- Hair는 반드시 Front / Side / Back 분리
- Hair는 중력 표현을 위해 Head 그룹에서 제외 — 회전/오프셋 영향 없음
- Helmet은 Head 그룹 안에 포함 — 머리와 함께 회전
- Armor는 몸통이므로 Head 그룹 밖
- Face 파츠는 FaceTemplate anchor를 기준으로 배치
- Beard는 Mouth anchor Y 공유, X 중앙 고정, 선택 파츠(null 허용)
- SkinMarking / FaceOverlay는 단일 또는 null. 다중 동시 착용 미지원 (1개만)
- FaceOverlay는 `side` 필드로 좌/우 결정 (eye 앵커 재활용), 또는 중앙 (마스크 등)
- ⚠️ FaceOverlay는 레이어 순서상 Head 그룹 밖에 위치 → 머리 회전(`rotationZ`)에 따라가지 않음. 회전 대응 필요 시 z_index 오버라이드로 Head 그룹 안에 두는 방식 향후 검토
