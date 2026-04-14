# Decision Log

이 문서는 프로젝트 진행 중 **확정된 설계 결정**과  
**논의 후 제외된 아이디어**를 기록한다.

목적:
- 설계 방향 유지
- 동일 논의 반복 방지
- AI 설계 제안 가이드

---

# Confirmed Decisions

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

BodyType  
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

BodyBase  
FaceBase  
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
- Head Node2D 하위에 BodyBase, FaceBase, Eyes, Nose, Mouth, Beard, Helmet 포함
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

BodyType  
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

# Pending Decisions

다음 항목은 아직 확정되지 않았다.

- FaceTemplate 수량
- Armor 바리에이션 방식
- 랜덤 생성 가중치 정책
- Face scale 범위
