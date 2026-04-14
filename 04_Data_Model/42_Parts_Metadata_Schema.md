# Parts Metadata Schema

## 원칙
- 각 파츠 이미지 파일 옆에 동일한 이름의 JSON 메타데이터 파일을 둔다
- 메타데이터는 파츠의 시각적 인상 수치를 담는다
- 수치는 캐릭터 생성 시 조합 제어 및 캐릭터 총합 계산에 사용된다

## 수치 정의

| 수치 | 설명 | 범위 |
|---|---|---|
| `beauty` | 미수치. 높을수록 아름다운 인상, 낮을수록 거칠고 투박한 인상 | -10 ~ +10 |
| `age` | 연령감. 높을수록 나이들어 보임, 낮을수록 어려 보임 | -10 ~ +10 |

## 파일 구조

```
/parts/eyes/
  eyes_02.png
  eyes_02.json   ← 메타데이터
```

## 스키마 예시

```json
// eyes_02.json — 크고 또렷한 눈
{
  "id": "eyes_02",
  "beauty": 7,
  "age": -3
}

// nose_blunt_01.json — 뭉툭한 코
{
  "id": "nose_blunt_01",
  "beauty": -4,
  "age": 2
}

// scar_face_01.json — 얼굴 흉터
{
  "id": "scar_face_01",
  "beauty": -8,
  "age": 5
}

// beard_heavy_01.json — 덥수룩한 수염
{
  "id": "beard_heavy_01",
  "beauty": -5,
  "age": 6
}
```

## 캐릭터 총합 계산

캐릭터 생성 시 장착된 모든 파츠의 수치를 합산한다.

```
eyes_02        beauty +7  age -3
nose_blunt_01  beauty -4  age +2
beard_heavy_01 beauty -5  age +6
───────────────────────────────
캐릭터 총합    beauty -2  age +5
```

## 활용

- 랜덤 생성 시 목표 beauty/age 범위를 설정해 일관된 인상의 캐릭터 생성
- 비슷한 수치의 파츠끼리 조합하면 자연스러운 외형 확보
- 인게임 이벤트/대사 분기 조건으로 활용 가능
