# AI Handoff

작업 단위 완료 시 섹션 추가. 최신이 위로 (top-append).
활성 엔트리 10개 초과 시 `archive/AI_HANDOFF_YYYYMM.md`로 이관.

---

## 2026-04-29 10:58 — Roadmap Step 4·5 통합 진행 + 부트스트랩 템플릿 스크립트 by Claude

**Step:** Roadmap 4·5단계 (통합 진행) — 사용자가 명시적으로 implementation 위임

**배경:** 로드맵상 4단계(레이어 합성)는 "중앙 배치만, 앵커 빼고"로 명세됐지만, 파츠가 크롭된 스프라이트인 점을 고려하면 중앙 배치만으론 시각 검증이 사실상 불가능(13개 사각형이 한 점에 뭉침). 사용자와 합의 후 5단계(앵커 시스템)와 통합 진행.

**변경/생성 파일:**
- (Godot `26b0060`) `asset-generater/scenes/character_view.tscn` — 13개 파츠 슬롯 트리. Head 그룹 회전/오프셋 단위, layer z-order는 `z_index` + `z_as_relative=false`로 강제.
- (Godot `26b0060`) `asset-generater/scripts/character_view.gd` — 캐릭터 JSON 로더 + 앵커 마커 RGB 정수 스캔 + 파츠별 setup. PNG 없으면 셰이더 우회 플레이스홀더 fallback.
- (Godot `32c6526`) `asset-generater/scripts/tools/generate_templates.gd` — 부트스트랩용 EditorScript. `face_template/sharp_01.png`(마커 7종) + `body_template/male_normal.png` 생성. AI 에셋 들어오기 전 통합 검증용 임시 자산.
- (Obsidian `91b8701`) `06_Production/60_Build_Roadmap.md` — 4·5단계 [x] + 통합 진행 표기 + 이슈 로그.

**미해결 질문 / 가정:**

1. **41_Character_Data_Schema에 `skin_color` 필드 부재.** FaceTemplate/BodyTemplate/Nose/Mouth는 24_Color_System상 `skin_color` uniform 사용 대상인데 캐릭터 JSON에 필드 없음. 임시로 `DEFAULT_SKIN_COLOR` 상수(`#E8B894`) 처리. **스키마 보완 필요.**
2. **22_Layer_System의 트리 구조 vs layer 번호 충돌.** Helmet은 Head 그룹 안(회전 동행) + 레이어 13/14(Armor=12 위) 두 요구가 트리만으로 양립 불가. `z_index` + `z_as_relative=false`로 해결. 문서에 이 메커니즘 명시 보완 권장.
3. **Helmet_Back/Helmet_Front 레이어 분리(13/14)는 단일 Helmet 노드(z=14)로 통합 구현.** 6단계(헬멧 룰)에서 분할 필요 시 재검토.
4. **skinMarking / faceOverlay 위치 명세 없음.** 일단 full-canvas 가정, position=(0,0). 실제 에셋 반입 시 재검토.
5. **Eyes offsetX 해석.** 좌우 대칭(양 눈 거리 조절)으로 채택. 평행 이동 vs 대칭 둘 다 가능했으나 미용 도구 관점에서 대칭 채택. 41_Character_Data_Schema에 명시 보완 권장.
6. **부트스트랩 템플릿 마커 위치는 대충 잡은 값** (눈 ±50, 코 +10, 입 +60 등). 실제 에셋 들어오면 무관, 검증용 임시값.

**리뷰 필요:** yes — 트리거: rendering logic 추가, 13개 파츠 합성 로직, 스키마 가정 6건. Codex 리뷰 권장.

**다음 담당:** User — 부트스트랩 스크립트 실행(Ctrl+Shift+X) → `character_view.tscn` 씬 실행 → 검증 포인트(눈코입이 마커 위치에 붙는지) 확인. 통과 시 6단계 진행. 위 미해결 1·5는 스키마 문서 보완 필요(Claude 또는 Codex).

---

## 2026-04-29 09:27 — Roadmap Step 2·3 검증 보완 + 체크박스 갱신 by Claude

**Step:** Roadmap 2단계 보완 + 2·3단계 [x] 처리

**배경:** 60_Build_Roadmap이 1단계만 [x]였는데 실제 코드엔 2~3단계 작업물이 들어가 있는 갭 발견. 2단계 검토 중 `skin_marking`, `face_overlay` 폴더에 파츠 메타데이터 JSON 누락 확인. 3단계(셰이더)는 사용자 합의로 테스트 목적 최소 구현 통과.

**변경/생성 파일:**
- (Godot `65db09b`) `asset-generater/assets/parts/skin_marking/skin_marking_scar_01.json`, `skin_marking_tattoo_01.json`
- (Godot `65db09b`) `asset-generater/assets/parts/face_overlay/face_overlay_glasses_01.json`, `face_overlay_warpaint_01.json`
- 스키마: 다른 부수 파츠와 동일하게 `{id, beauty, age}`
- (Obsidian `2e5878b`) `06_Production/60_Build_Roadmap.md` — 2·3단계 [x] + 진행 로그(완료일, 커밋 해시, 이슈)

**미해결 질문:** 없음

**리뷰 필요:** no — JSON 4개는 기존 패턴 답습, 스키마 변경 없음.

**다음 담당:** Claude — 4·5단계 진행 (이 엔트리 후속으로 같은 세션에서 진행됨, 다음 핸드오프 엔트리 참조)

---

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