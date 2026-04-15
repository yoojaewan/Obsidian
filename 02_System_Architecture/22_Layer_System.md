# Layer System

## 기본 레이어 순서
1. BodyBase
2. FaceBase
3. Eyes  <!-- 눈썹 포함, 눈+눈썹 세트로 하나의 파츠 -->
4. Nose
5. Mouth
6. Beard
7. Hair_Back
8. Hair_Side
9. Hair_Front
10. Armor
11. Helmet_Back
12. Helmet_Front

## Godot 씬 그룹 구조

```
Root
├── Hair_Back      ← Head 그룹 밖 (중력, 회전 없음)
├── Hair_Side      ← Head 그룹 밖
├── Hair_Front     ← Head 그룹 밖
│
├── Head (Node2D)  ← rotation_z, offset_y 적용 대상
│   ├── BodyBase
│   ├── FaceBase
│   ├── Eyes
│   ├── Nose
│   ├── Mouth
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
- 액세서리 레이어는 현재 범위 제외
