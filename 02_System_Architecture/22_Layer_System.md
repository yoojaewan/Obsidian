# Layer System

## 기본 레이어 순서 (z_index 절대값, 작을수록 뒤)

| z | 레이어 | 비고 |
|---|---|---|
| 0 | Hair_Back | 가장 뒤 (Body, Face 뒤로) — 자연스러운 뒷머리 |
| 1 | BodyTemplate | 어깨~허리 |
| 2 | FaceTemplate | 머리/얼굴 베이스 |
| 3 | Nose | |
| 4 | Mouth | |
| 5 | SkinMarking | 주근깨/흉터/점/다크서클. Nose/Mouth 위, Eyes/Beard 아래. 단일 또는 null |
| 6 | Eyes | 눈썹 포함, 눈+눈썹 세트로 하나의 파츠 |
| 7 | Beard | |
| 9 | FaceOverlay | 안대/마스크/외눈안경. Hair_Back 위, Hair_Side/Hair_Front 아래. 단일 또는 null |
| 10 | Hair_Side | |
| 11 | Hair_Front | 앞머리 |
| 12 | Armor | 갑옷 |
| 13 | Helmet_Back | |
| 14 | Helmet_Front | |

(z=8 의도적으로 비워둠 — Hair_Back을 0으로 옮긴 이력. 향후 추가 레이어 슬롯 여유)

## 컨텍스트 구분: 편집 시 트리 vs 합성 순서

**중요:** 이 문서가 정의하는 두 정보는 *서로 다른 관점*이다.
- **합성 순서 (z_index)** = 게임 내 출력 시 그려지는 순서. 위 표.
- **편집 시 Godot 씬 트리** = 에셋 생성기 단계의 노드 부모/자식 관계. 아래 그림. 머리/몸을 별도 조정하기 위해 합성 순서와 무관하게 결정.

## Godot 씬 그룹 구조 (편집 시)

```
Root (CharacterView)
├── Hair_Back       ← Head 그룹 밖 (중력, 회전 없음). z_index=0
├── BodyTemplate    ← Head 그룹 밖 (에셋 생성기에서 머리/몸 별도 조정). z_index=1
│
├── Head (Node2D)   ← head.offset/scale/rotationZ 적용 대상
│   ├── FaceTemplate
│   ├── Nose
│   ├── Mouth
│   ├── SkinMarking ← 선택 (null 허용)
│   ├── Eyes (Node2D)
│   │   ├── EyeL (flip_h=true)
│   │   └── EyeR
│   ├── Beard
│   └── Helmet      ← 머리와 함께 움직이므로 그룹 안
│
├── FaceOverlay     ← Head 그룹 밖. 선택 (null 허용)
├── Hair_Side       ← Head 그룹 밖
├── Hair_Front      ← Head 그룹 밖
└── Armor           ← 몸통, 그룹 밖
```

각 노드는 `z_as_relative=false` 로 z_index 절대값 적용 (트리 위치와 무관하게 합성 순서 결정).

## 원칙
- Hair는 반드시 Front / Side / Back 분리 (Hair_Back은 Body 뒤, Front/Side는 얼굴 위)
- Hair는 중력 표현을 위해 Head 그룹 밖 — 회전/오프셋 영향 없음. 단 정수리 마커는 face_template 안 → 코드에서 head.offset/scale 보정해서 정렬 (회전 제외)
- Helmet은 Head 그룹 안에 포함 — 머리와 함께 회전
- BodyTemplate / Armor는 몸통이므로 Head 그룹 밖
- Face 파츠 (Nose/Mouth/Eyes/SkinMarking 등)는 FaceTemplate anchor를 기준으로 배치
- Beard는 Mouth anchor Y 공유, X 중앙 고정, 선택 파츠 (null 허용)
- SkinMarking / FaceOverlay는 단일 또는 null. 다중 동시 착용 미지원
- FaceOverlay는 `side` 필드로 좌/우 결정 (eye 앵커 재활용), 또는 중앙 (마스크 등)
- ⚠️ FaceOverlay는 Head 그룹 밖 → 머리 회전 따라가지 않음. 회전 대응 필요 시 z_index 오버라이드로 Head 그룹 안에 두는 방식 향후 검토
