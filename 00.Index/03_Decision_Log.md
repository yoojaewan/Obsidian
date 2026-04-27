# Decision Log

이 문서는 프로젝트 진행 중 **확정된 설계 결정**과  
**논의 후 제외된 아이디어**를 기록한다.

목적:
- 설계 방향 유지
- 동일 논의 반복 방지
- AI 설계 제안 가이드

---

# Confirmed Decisions

## 2026-04-27 — 얼굴 액세서리 슬롯 도입 (skinMarking / faceOverlay)

- **추가 슬롯:**
  - `skinMarking` — 주근깨/흉터/점/다크서클 등 피부 위 표시. 레이어: **Nose/Mouth 위, Eyes/Beard 아래**
  - `faceOverlay` — 안대/마스크/외눈안경 등 눈 위 부착물. 레이어: **Hair_Back 위, Hair_Side/Hair_Front 아래** (모든 얼굴 파츠보다 위, 앞/옆머리에 가려짐)
- **레이어 재배치:** 기존 Eyes(3)→Nose(4)→Mouth(5)→Beard(6) 순서를 Nose(3)→Mouth(4)→SkinMarking(5)→Eyes(6)→FaceOverlay(7)→Beard(8)로 변경. 이유: 코/입 외곽선 위에 주근깨가 찍히는 게 자연스러움 (피부 디테일은 얼굴 구조보다 위)
- **다중 착용 미지원:** 각 슬롯은 단일 객체 또는 null (배열 아님). 추후 필요 시 확장 검토
- **기존 슬롯 패턴 유지:** Hair_Front/Side/Back처럼 슬롯이 곧 레이어 위치를 결정. 파츠 메타데이터에 layer 필드 두지 않음
- **FaceOverlay `side` 필드:** `"left"` / `"right"` / `"center"` — eye 앵커 재활용, 마스크는 중앙
- **신규 폴더:** `assets/parts/skin_marking/`, `assets/parts/face_overlay/`
- **반영 문서:** [[22_Layer_System]], [[41_Character_Data_Schema]]

---

## 2026-04-27 — 파츠 캔버스 512 → 500 변경

- **변경:** 파츠 원본 PNG 크기 `512×512` → `500×500`
- **이유:** remove.bg 무료 등급 출력 상한이 500×500. 23장 + 향후 추가 파츠 모두 동일 도구로 일괄 배경 제거하기 위해 통일.
- **영향:**
  - UI 미리보기: 384×384 → 375×375 (원본의 75% 유지)
  - 썸네일: 192×192 → 187×187 (원본의 37.5% 유지)
  - Godot 4 2D는 non-power-of-2 텍스처 성능 영향 사실상 없음
- **반영 문서:** [[27_UI_Layout]]
- **이관:** Work_Log 2026-04-15의 "512×512 통일" 결정은 역사적 기록으로 남김

---

## System Scope

- 본 프로젝트는 **2D SRPG 캐릭터 초상화 생성용 Asset Generator**이다.
- 시스템 목적은 **대량 캐릭터 에셋 생산 자동화**이다.
- 생성 대상은 **캐릭터 Portrait 에셋**이다.

---

## Engine Scope

다음 요소는 **프로젝트 범위에 포함되지 않는다.**

- Live2D
- Animation System
- 3D Character System

---

## Equipment Scope

다음 장비는 **현재 범위에서 제외한다.**

- Shoulder Armor
- Cloaks / Capes
- Complex Accessories

이유:
- 레이어 충돌 복잡도 증가
- 아트 리소스 폭발

---

## Character System Structure

캐릭터는 **모듈형 파츠 시스템**으로 구성한다.

구성 요소:

BodyTemplate  
FaceTemplate  
Hair (Front / Side / Back)  
Eyes  
Nose  
Mouth  
Beard  
Armor  
Helmet  

---

## Layer System

기본 렌더 레이어 구조는 다음과 같다.

BodyTemplate  
FaceTemplate  
Eyes  
Nose  
Mouth  
Beard  
Hair_Back  
Hair_Side  
Hair_Front  
Armor  
Helmet_Back  
Helmet_Front  

---

## Head Group System

초상화 각도는 **정면 고정**. 각도 바리에이션은 다음 두 파라미터로 확보한다.

- `head.offset_y` — 머리 Y 위치 조정 (목 길이 / 거북목 표현)
- `head.rotation_z` — Z축 미세 회전 (고개 기울기)

**Godot 씬 그룹:**
- Head Node2D 하위에 BodyTemplate, FaceTemplate, Eyes, Nose, Mouth, Beard, Helmet 포함
- Hair는 중력 표현을 위해 Head 그룹 밖 — 회전/오프셋 영향 없음
- Armor는 몸통이므로 Head 그룹 밖

---

## Anchor System

- 파츠 좌표는 **FaceTemplate이 소유한다**
- FaceTemplate 이미지에 **약속된 색상으로 1픽셀 마커를 직접 찍는다**
- 눈/코/입 파츠가 덮으므로 렌더링에 노출되지 않는다
- 마커 색상: Eye_L=#FF0000, Eye_R=#00FF00, Nose=#0000FF, Mouth=#FFFF00
- Hair/Helmet 앵커는 Y 좌표만 사용, X는 캔버스 중앙 고정
- Hair, Armor, Helmet 등 나머지 파츠는 **캔버스 중앙 기준 배치**

---

## Hair Structure

헤어는 반드시 **3분할 구조**로 제작한다.

Hair_Front  
Hair_Side  
Hair_Back  

이유:

- 투구 대응
- 레이어 충돌 최소화
- 재사용성 증가

---

## Helmet Handling

투구는 단순 이미지가 아니라 **Rule Object**로 처리한다.

투구 장착 시 다음 처리가 가능하다.

- Hair Layer Hide / Swap
- Face 파츠 Hide

---

## Helmet Types

투구는 다음 4가지 타입으로 분류한다.

| 타입 | 헤어 규칙 | 얼굴 파츠 hide | 예시 |
|---|---|---|---|
| TIARA | NONE | 없음 | 티아라, 왕관, 머리띠 |
| HAT | SHOW | 없음 | 모자, 두건 |
| MASK | SHOW | nose, mouth, beard | 마스크, 복면 |
| HELM | HIDE / SWAP | nose, mouth, beard | 갑옷 투구 |

---

## Color System

- 파츠 에셋은 **그레이스케일 베이스**로 제작한다
- 색상은 **그라디언트 매핑** 방식으로 런타임 적용한다
- 유저는 **주 컬러 1개**만 지정, 하이라이트/그림자는 자동 계산
- 명암 구조(위치)는 그레이스케일 베이스 에셋이 결정한다
- 색상값은 캐릭터 데이터가 소유한다
- 적용 대상: Hair, Eyes, Beard, Armor, Helmet

---

## FaceTemplate

- **2개로 시작**, 이후 확장 구조 유지
- 귀 포함 (얼굴 실루엣의 일부)
- **초상화 전용** — 필드 스프라이트에 영향 없음
- Template A: 넓고 둥근 얼굴 (친근한 인상)
- Template B: 좁고 긴 얼굴 (날카로운 인상)
- 앵커 좌표가 크게 다른 두 형태로 시스템 확장성 검증

---

## Field Sprite 구조

- 공통 뼈대 사용 — 모든 캐릭터 동일
- 헤어 / 갑옷 / 투구로 시각적 구분
- 개성 표현: **기본자세(pose) 바리에이션**
- 공격 모션은 무기 타입 기반 (별도 설계)

---

## 성별 에셋 분리 정책

| 파츠 | 분리 여부 | 이유 |
|---|---|---|
| BodyTemplate | 분리 | 체형 자체가 다름 |
| FaceTemplate | 분리 | 얼굴 윤곽/비율이 다름 |
| Hair | 분리 | 스타일 자체가 다름 |
| Eyes | **공용** | 타입 선택 + scale/position으로 인상 조절 |
| Nose | **공용** | 성별 신호 약함 |
| Mouth | **공용** | 1자입 스타일 — 성별 무관 |
| Beard | 남성 전용 | 어차피 남성 전용 |
| Armor / Helmet | **공용** | 대부분 공유 가능 |

---

## 초상화 구도

- 허리까지 표시 (머리 ~ 허리/벨트)
- 대화창: 허리 구도에서 얼굴 영역 크롭해서 사용 (별도 에셋 없음)
- 감정 표현: 얼굴 크롭 + 눈/입 파츠 교체
- 이벤트 씬 등 용도별 크롭은 Godot에서 텍스처 영역 지정으로 처리

---

## Armor 시스템

- **통짜 스프라이트** — 오버레이 없음 (추후 필요시 검토)
- 베이스: 바디타입 3 × 갑옷타입 3 = **9개**
  - 갑옷 타입: 판금 / 가죽 / 천
  - 바디 타입: 덩치 / 기본 / 여성형
- 등급 차이: **색상으로만 처리** (초급=낡고 어두움, 고급=밝고 선명)

---

## Asset Pipeline

- 아트 에셋은 **AI 이미지 생성**으로 제작, 필요시 **Krita**로 편집
- 파츠 에셋은 **Portrait(512×640)** 과 **Field Sprite(80×96)** 두 버전으로 제작
- 인게임에서 Field Sprite는 2× 스케일로 표시 (160×192)

---

## Scale System

- 파츠 에셋 자체는 항상 **스케일 1**로 제작
- 스케일 값은 **캐릭터 데이터가 소유**하며 캐릭터 생성 시 적용
- 스케일은 Anchor 기준으로 확대/축소

---

## Random Generation

랜덤 캐릭터 생성 시 다음 요소를 사용한다.

BodyTemplate  
FaceTemplate  
Hair Style  
Hair Color  
Eye Style  
Eye Color  
Armor  
Helmet  

---

## Parts Metadata — 파츠 수치 시스템

각 파츠는 다음 두 가지 수치를 메타데이터로 가진다.

- `beauty` : 미수치 (-10 ~ +10). 높을수록 아름다운 인상, 낮을수록 거칠고 투박한 인상
- `age` : 연령감 (-10 ~ +10). 높을수록 나이들어 보임, 낮을수록 어려 보임

캐릭터 생성 시 장착 파츠 전체 합산 → 캐릭터 총합 수치 보유.
유사 수치 파츠끼리 조합하면 자연스러운 인상 확보 (예: beauty 낮은 파츠끼리 → 거친 전사형).

---

## Asset Generator 목적

- 인게임 에셋 조합이 올바르게 렌더링되는지 **확인용**
- 좋은 조합을 **고유 캐릭터로 등록**하기 위한 도구
- 편집기 기능은 범위에 포함되지 않는다

---

# Rejected Concepts

다음 아이디어는 논의 후 **채택하지 않기로 결정하였다.**

---

## Full Pose System

팔 분리 기반 포즈 시스템은 채택하지 않음.

이유:

- 아트 리소스 증가
- 장비 조합 복잡도 증가

---

## Fully Dynamic RGB Color

완전 랜덤 RGB 색상 생성은 사용하지 않음.

이유:

- 색 조합 품질 문제
- 캐릭터 디자인 일관성 저하

대신 **그라디언트 매핑 기반 색상 시스템** 사용.

---

## Universal Helmet Compatibility

모든 헤어 × 모든 투구 조합 대응 방식은 사용하지 않음.

이유:

- 리소스 폭발
- 유지보수 불가능

대신 **Rule 기반 처리 시스템** 사용.

---

## 2026-04-22 — Face/Body 네이밍 3박자 통일 (Template 계열)

기존에 레이어 이름, 데이터 필드명, 폴더 구조 간 용어가 혼재해 파츠 구조 설계 시 혼란이 있었다.

**결정 1 — FaceBase → FaceTemplate 단일화**
- 하나의 PNG가 두 역할 수행: 앵커 마커 소스 + 피부색 렌더링 레이어
- 마커 픽셀은 눈/코/입 파츠에 가려 렌더링에 노출되지 않으므로 한 파일로 충분

**결정 2 — BodyBase / BodyType → BodyTemplate 단일화**
- `BodyBase`(레이어명) + `bodyType`(JSON 필드) + `body_base/`(폴더)로 세 용어가 분산되어 있던 것을 `BodyTemplate` 계열로 통일
- FaceTemplate과 동일한 네이밍 패턴: 레이어=`BodyTemplate`, 필드=`bodyTemplate`, 폴더=`body_template/`

**통일 패턴 (전 파츠 공통):**
- 레이어명(PascalCase) ↔ JSON 필드명(camelCase) ↔ 폴더명(snake_case) 삼박자 일치

**영향 문서:** [[22_Layer_System]], [[24_Color_System]], [[41_Character_Data_Schema]], [[01_Project_Summary]], [[03_Decision_Log]]

---

# Pending Decisions

다음 항목은 **Godot 프로토타입 구현 시 결정**한다.

- 랜덤 생성 가중치 정책
- Face scale 범위 (테스트 후 결정)

## 2026-04-27 — AI 협업 프로토콜 v1 확정

- Claude Code와 Codex는 `05_AI_Workspace/AI_DIALOG.md`를 비동기 논의 공간으로 사용한다.
- 작업 완료 후 인수인계는 `05_AI_Workspace/AI_HANDOFF.md`에 최신 항목을 위로 추가한다.
- 확정 결정은 이 Decision Log에 기록하고, 작업 단위/상태는 `06_Production/60_Build_Roadmap.md`를 기준으로 한다.
- 기본 레인은 Claude=기획/문서/설계 추론, Codex=구현/리팩터/구현 디테일이며 예외는 사용자가 명시한다.
- 교차 리뷰는 매 라운드 강제가 아니라 스키마 변경, 렌더링 로직, 저장/내보내기, 대량 파일 생성, 기획-구현 불일치 위험이 있을 때 필수로 한다.
- `AI_HANDOFF.md` 활성 엔트리가 10개를 넘으면 `05_AI_Workspace/archive/AI_HANDOFF_YYYYMM.md`로 이관한다.

영향 파일: `05_AI_Workspace/AI_DIALOG.md`, `05_AI_Workspace/AI_HANDOFF.md`, `05_AI_Workspace/archive/.gitkeep`.