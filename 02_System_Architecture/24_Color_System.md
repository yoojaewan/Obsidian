# Color System

## 원칙

- 파츠 에셋은 **그레이스케일**로 제작한다
- 색상은 런타임에 **그라디언트 매핑**으로 적용한다
- 유저는 **주 컬러 1개**만 지정하면 된다

---

## 그라디언트 매핑 동작 원리

그레이스케일 이미지의 각 픽셀은 밝기값(0~255)을 가진다.
이 밝기값이 **어디가 하이라이트고 어디가 그림자인지**를 결정한다.

| 픽셀 밝기 | 의미 | 적용 색 |
|---|---|---|
| 255 (흰색) | 빛이 가장 많이 닿는 곳 | 하이라이트 색 |
| 128 (회색) | 중간톤 | 주 컬러 그대로 |
| 0 (검정) | 가장 어두운 그림자 | 그림자 색 |

주 컬러 하나에서 하이라이트/그림자 색을 자동 계산:
- 하이라이트: 주 컬러 → 밝기 증가 + 채도 감소
- 그림자: 주 컬러 → 밝기 감소 + 채도 증가

---

## 역할 분담

| 역할 | 담당 |
|---|---|
| 하이라이트/그림자 위치 결정 | AI가 그레이스케일로 생성할 때 |
| 색상 적용 | 프로그램이 주 컬러 기준으로 자동 계산 |

프로그램은 색을 입힐 뿐, 명암 구조는 만들어내지 않는다.
**그레이스케일 베이스의 명암 품질이 최종 결과물 품질을 결정한다.**

---

## AI 에셋 생성 프롬프트 가이드

AI 이미지 생성 시 다음 키워드를 포함한다:

```
grayscale, monochrome, [파츠명] asset, game sprite,
white background, strong highlights, deep shadows,
no color, concept art
```

흑백 사진처럼 명암이 뚜렷하게 나오면 정상이다.

---

## Krita에서 테스트하는 법

- 레이어에 곱하기(Multiply) 블렌드 모드로 색 올리는 방식은 **하이라이트가 적용 안 됨** → 사용 금지
- 올바른 방법: **필터 > 지도 > 그라디언트 맵**
  - 왼쪽 끝 (어두움) → 그림자 색
  - 중간 → 주 컬러
  - 오른쪽 끝 (밝음) → 하이라이트 색

---

## Krita 파츠 후처리 워크플로우

AI 생성 후 Krita에서 다음 작업이 필요하다:

1. 배경 제거 → 투명 PNG로 저장
2. 경계선 정리
3. 그라디언트 맵으로 색상 결과 검증 후 저장

---

## 적용 대상

- Hair (Front / Side / Back)
- Eyes
- Beard
- Armor
- Helmet

---

## Godot 구현

셰이더에서 주 컬러 1개를 uniform으로 받아 하이라이트/그림자 색을 자동 계산 후 그레이스케일 텍스처에 매핑한다.

파츠별로 `main_color` uniform만 다르게 넘겨주면 각각 다른 색 실시간 적용 가능.

```glsl
shader_type canvas_item;

uniform vec4 main_color : source_color;

void fragment() {
    float brightness = texture(TEXTURE, UV).r; // 그레이스케일 밝기값

    vec4 shadow    = vec4(main_color.rgb * 0.4, 1.0);                      // 어두움
    vec4 highlight = vec4(min(main_color.rgb * 1.8 + 0.2, 1.0), 1.0);     // 밝음

    COLOR = mix(mix(shadow, main_color, smoothstep(0.0, 0.5, brightness)),
                highlight,
                smoothstep(0.5, 1.0, brightness));
}
```

---

## 주의

- 그레이스케일 베이스의 명암 품질이 최종 결과물 품질을 결정한다
- 프로그램은 색을 입힐 뿐, 명암 구조는 만들어내지 않는다
- Krita에서 Multiply 블렌드 모드는 이 용도에 사용 금지
