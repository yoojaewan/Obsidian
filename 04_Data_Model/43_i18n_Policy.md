# i18n (다국어) 정책

## 원칙

유저에게 보이는 텍스트는 전부 언어 파일로 분리한다.
데이터 JSON에는 텍스트 대신 `name_key`를 저장하고, 실제 텍스트는 언어 파일에서 관리한다.

---

## 적용 대상

- 장비 이름
- 캐릭터 이름
- 클래스 이름 (기사, 마법사 등)
- 스킬/아이템 이름
- UI 텍스트 (버튼, 라벨 등)

---

## 구조

```json
// armor_knight_01.json
{
  "id": "armor_knight_01",
  "name_key": "item.armor_knight_01.name"
}

// strings_ko.json
{
  "item.armor_knight_01.name": "철기사 흉갑",
  "class.knight.name": "기사",
  "ui.save": "저장"
}

// strings_en.json
{
  "item.armor_knight_01.name": "Iron Knight Breastplate",
  "class.knight.name": "Knight",
  "ui.save": "Save"
}
```

---

## name_key 네이밍 규칙

| 카테고리 | 접두사 | 예시 |
|---|---|---|
| 장비 | `item.` | `item.armor_knight_01.name` |
| 캐릭터 | `character.` | `character.001.name` |
| 클래스 | `class.` | `class.knight.name` |
| 스킬 | `skill.` | `skill.slash_01.name` |
| UI | `ui.` | `ui.save` |

---

## 언어 추가 방법

새 언어 파일(`strings_jp.json` 등)을 추가하기만 하면 된다.
기존 데이터 JSON은 수정 불필요.
