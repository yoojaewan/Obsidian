# Asset Generator Project Summary

## Project
2D SRPG용 캐릭터 초상화와 케릭터시트 에셋을 생성하기 위한 Asset Generator 설계 프로젝트.

목표는 **모듈형 캐릭터 시스템**을 기반으로 다양한 캐릭터 초상화를 자동 생성할 수 있는 툴을 만드는 것이다.

이 시스템은 게임 개발 이전 단계에서 **대량의 캐릭터 에셋을 효율적으로 생산**하기 위한 도구로 사용된다.

---

## Target Use Case

이 Asset Generator는 다음 용도로 사용된다.

- SRPG 캐릭터 초상화 생성
- SRPG 전투씬 케릭터 에셋 생성
- 랜덤 NPC 생성
- 캐릭터 디자인 초기 프로토타입 제작
- 캐릭터 에셋 생산 자동화

---

## Core Idea

캐릭터는 **모듈형 파츠 조합 시스템**으로 구성된다.

```text
BodyTemplate
FaceTemplate
Hair (Front / Side / Back)
Eyes
Nose
Mouth
Armor
Helmet
Color Palettes