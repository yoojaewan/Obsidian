# AI Handoff

작업 단위 완료 시 섹션 추가. 최신이 위로 (top-append).
활성 엔트리 10개 초과 시 `archive/AI_HANDOFF_YYYYMM.md`로 이관.

---

## 2026-04-27 10:30 — Roadmap Step 2 Codex 리뷰 반영 by Claude

**작업 내용:** Codex 리뷰 4건 처리

1. **[High] 워크트리→메인 프로젝트 복사 완료**
   - `F:\godot\Asset_Generater\asset-generater\` 하위에 `data/`, `assets/` 반영됨

2. **[Low] camelCase 통일**
   - `name_key` → `nameKey`, `offset_y` → `offsetY`, `rotation_z` → `rotationZ`
   - 영향 파일: `char_knight_01.json`, `armor_*.json`, `helmet_*.json`

3. **[Medium] .gitignore에 `.claude/` 추가**

4. **[Medium] `data/strings/strings_ko.json` 플레이스홀더 생성**
   - 5개 키 (character 1 + armor 2 + helmet 2)

**메인 프로젝트 루트:** `F:\godot\Asset_Generater\asset-generater\`

**미해결 (이전 엔트리에서 이관):**
- helmet.type 위치 (character JSON vs helmet metadata) — 사용자 결정 필요
- schema 문서(41_Character_Data_Schema)에 `nameKey` 필드 명시 보완 필요

---

## 2026-04-27 09:50 — Roadmap Step 2 샘플 JSON 생성 by Claude

**Step:** Roadmap 2단계 (샘플 데이터 작성)

**변경/생성 파일:** (총 23개)
**워크트리 루트:** `F:\godot\Asset_Generater\asset-generater\.claude\worktrees\cool-gauss-413d41\asset-generater\`
- `data/characters/char_knight_01.json` — 풀 스택 샘플 캐릭터 (남성, 기사, 헬멧 HELM)
- `assets/parts/body_template/{male_normal, female_slim}.json`
- `assets/parts/face_template/{round_01, sharp_01}.json` — Decision Log Template A/B 매칭
- `assets/parts/eyes/{eyes_round_01, eyes_sharp_01}.json`
- `assets/parts/nose/{nose_straight_01, nose_blunt_01}.json`
- `assets/parts/mouth/{mouth_smile_01, mouth_thin_01}.json`
- `assets/parts/beard/{beard_light_01, beard_heavy_01}.json`
- `assets/parts/hair_{front,side,back}/..._short_01.json, ..._long_01.json` (6개)
- `assets/parts/armor/{armor_knight_01, armor_leather_01}.json` — texture metal/leather
- `assets/parts/helmet/{helmet_closed_01, helmet_open_01}.json` — texture metal/leather

**미해결 질문 / 스키마 이슈:**

1. **helmet.type 위치 모호.** [[41_Character_Data_Schema]] 예시에선 character JSON 안 helmet 객체에 `"type": "HELM"` 있음. 하지만 논리적으로 helmet의 type은 helmet asset 자체의 intrinsic 속성이라 helmet metadata에 있어야 자연스러움. **현재 스키마 따라 character에만 둠 — 사용자/Codex 결정 필요.**

2. **character JSON에 `name_key` 추가함.** [[41_Character_Data_Schema]]에는 명시 없지만 [[43_i18n_Policy]] 적용 대상에 "캐릭터 이름" 포함되어 추가. 스키마 문서 보완 필요할 듯.

3. **armor/helmet 메타에만 `name_key` 포함.** 다른 파츠(eyes/nose/mouth 등)는 유저 노출 텍스트 없다고 보고 미포함.

4. **scale 정책.** schema 예시는 hair에 단일 scale, front/side/back 별도 scale 없음. 그대로 따랐음. 향후 비대칭 요구 시 분리 필요.

**리뷰 필요:** yes — 대량 파일 생성 + 스키마 해석 모호점 3건 (트리거 조건 충족)

**다음 담당:** Codex — JSON 파싱 가능 여부 확인 + 위 4개 이슈 검토 + 네이밍 3박자 일관성 재확인. 또는 User가 직접 검토.

---