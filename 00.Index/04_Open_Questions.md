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
Armor  
Helmet  
Color Palette  

---

## Layer System

기본 렌더 레이어 구조는 다음과 같다.

BodyBase  
FaceBase  
Eyes  
Nose  
Mouth  
Hair_Back  
Hair_Side  
Hair_Front  
Armor  
Helmet_Back  
Helmet_Front  

---

## Anchor System

- 파츠 좌표는 **FaceTemplate이 소유한다**
- 파츠는 **Anchor + Offset 방식으로 배치한다**
- 좌표는 **비율 기반 구조 사용을 고려한다**

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

- Hair Layer Hide
- Hair Layer Swap
- Helmet Layer Add

---

## Helmet Types

투구는 다음 타입으로 분류한다.

OPEN  
HALF  
CLOSED  

각 타입은 Hair Layer 처리 규칙을 가진다.

---

## Color System

헤어와 눈 색상은 **캐릭터 데이터가 소유한다.**

색상 시스템은 다음 방식 사용:

Palette Selection  
+  
Small Color Variation  

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

대신 **Palette 기반 색상 시스템** 사용.

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