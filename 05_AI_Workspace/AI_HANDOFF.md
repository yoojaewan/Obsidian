# AI Handoff

작업 단위 완료 시 섹션 추가. 최신이 위로 (top-append).
활성 엔트리 10개 초과 시 `archive/AI_HANDOFF_YYYYMM.md`로 이관.

---

## 2026-04-30 — 파츠 PNG 캔버스 컨벤션 확정 + face=200×200 baseline 의사결정 by Claude

**Step:** Roadmap 4·5 후속 (사이즈 컨벤션 확정), 8단계 베이크 패턴 사전 설계

**배경:** 2026-04-29 풀사이즈 eye 사고("얼굴이 눈보다 큰 괴물")의 근본 원인 = 파츠 PNG 캔버스 컨벤션 미명시. 사용자 검토 통해 **실사이즈 캔버스 + 캔버스 중심=부위 중심** 컨벤션으로 정정. face_template/body_template만 고정 사이즈.

**확정 결정사항:**

1. **파츠 PNG 캔버스 — 실사이즈 (500×500 통일 폐기)**
   - 캔버스 = 그 파츠의 바운딩 박스 = "범위 템플릿"
   - 필수: 캔버스 중심 = 부위 정렬 기준점 (Sprite2D `centered=true` 와 결합)
   - 외곽선 두께 **≥2px** (linear 보간 안전 마진. 1px는 미세 스케일도 위험)

2. **고정 사이즈:**
   - **face_template = 200×200** (사실 비율 baseline, 머리:어깨 ≈ 1:2.5)
   - **body_template = 500×500** (어깨~허리)
   - 사용자 의도: 초상화는 사실 비율, SD/귀여움은 별도 face_template asset로 처리
   - 캐릭터별 미세 비율 조정은 head.scale 0.9~1.1 (외곽선 2px 전제)

3. **부위별 권장 사이즈 (face 200 기준):**
   - eye(각) **40×40 정사각형** (눈+눈썹 통합 — 22_Layer_System 9행)
   - nose 16×32, mouth 40×16, beard 72×48
   - **마커 의미 재정의:** 마커 = 부위 PNG 캔버스 중심의 정렬 기준점 (해부학적 중심 X). 코드는 `centered=true` 라 PNG 캔버스 중심을 마커 좌표에 둘 뿐. 사용자가 마커 박을 때 "이 위치에 부위 PNG 통째로 와야 한다" 로 판단
   - skin_marking ~50×25, face_overlay ~60×24
   - hair_front ~104×80, hair_side ~112×112, hair_back ~128×128, helmet ~112×104
   - armor 500×500

4. **face_template 200×200 마커 좌표 (좌상단=0,0):**
   - eye_l(73,87) #FF0000  /  eye_r(126,87) #00FF00
   - nose(100,110) #0000FF  /  mouth(100,136) #FFFF00
   - headwear(101,9) #FF00FF
   - ⚠️ 위는 sharp_01 0.4배 시작점. 새 일러스트의 실제 부위 위치에 맞춰 박을 것

5. **AI 생성 워크플로우 (200 함정 회피):**
   - 512×512 큰 캔버스로 AI 생성 → Krita 200으로 다운샘플 → Threshold 필터로 외곽선 강제 검정화 → 마커 1px 박기

6. **8단계 베이크 패턴 사전 설계 (구현은 8단계):**
   - 셰이더 외곽선 제약을 근본 우회
   - 패턴: SubViewport per part → 셰이더 적용 결과를 ImageTexture로 베이크 → Sprite2D에 할당
   - 색 변경 → 해당 파츠 재베이크 / 스케일·회전 → 변환만 갱신 (베이크 그대로)
   - Head 그룹의 다같이 늘어나고 회전은 Godot 부모-자식 트랜스폼 상속으로 자동 보장
   - face_template 베이크 시 마커 픽셀 마스킹 필요 (`_scan_anchors` 후)
   - head.scale ≥1.5 SD 자유 표현 가능해짐

**변경/생성 파일:** 없음 (의사결정 + 메모리 갱신만)

**기획 문서 갱신 필요 (사용자 직접 — "문서가 진실"):**
- ⚠️ [42_Parts_Metadata_Schema](F:\opsidian\git_obsidian\04_Data_Model\42_Parts_Metadata_Schema.md) — 캔버스 컨벤션 + 부위별 권장 사이즈 표 추가
- ⚠️ [24_Color_System](F:\opsidian\git_obsidian\02_System_Architecture\24_Color_System.md) — 외곽선 두께 ≥2px 컨벤션 명시, AI 생성 워크플로우(다운샘플+Threshold) 추가
- ⚠️ [23_Anchor_System](F:\opsidian\git_obsidian\02_System_Architecture\23_Anchor_System.md) — face_template 사이즈 200×200 + 마커 절대 좌표 명시
- 권장: `00.Index/03_Decision_Log.md` 항목 `2026-04-30 — face=200×200 baseline + 외곽선 2px + 8단계 베이크 패턴 결정`

**미해결 질문:** 없음

**리뷰 필요:** yes — 트리거: schema/컨벤션 변경 + 큰 아키텍처 결정(베이크 패턴). Codex가 8단계 진입 시 베이크 패턴 설계 검토 필요.

**다음 담당:** User — 위 작업 순서대로 이미지 재제작:
1. eyes_sharp_01.png 삭제/백업 (풀사이즈 잘못된 에셋)
2. face_template/sharp_01.png 200×200 재제작 + 마커 5개
3. eyes/eyes_sharp_01.png 32×20 재제작
4. Godot character_view.tscn 시각 검증
5. 통과 후 나머지 부위 PNG 점진 제작
검증 통과 시 Claude가 `character_view.gd` PART_IMAGE_SIZE 상수/플레이스홀더 정리 + 6단계(헬멧 룰) 진행.

---

## 2026-04-29 15:00 — generate_templates.gd 제거 (footgun) by Claude

**Step:** Roadmap 4·5 후속 정리

**배경:** 부트스트랩 PNG 생성 스크립트(`generate_templates.gd`)가 사용자가 만든 진짜 face/body PNG를 실수로 덮어쓰는 사고 발생. AI가 이전 메시지에서 *"스킵해도 됨"* 경고와 함께 *"Ctrl+Shift+X로 실행"* 지시를 같이 넣어 사용자 혼동 → 실행 → 진짜 에셋 손실. 사용자가 직접 PNG 다시 복원함.

**조치:**
- (Godot `1e30742`, master ff-merge 완료) `asset-generater/scripts/tools/generate_templates.gd` 삭제 + 빈 `tools/` 폴더 제거
- 향후 부트스트랩 필요 시 git history(32c6526)에서 복원 가능

**미해결 질문:** 없음

**리뷰 필요:** no — 단순 제거. 진행 중인 작업에 영향 없음 (실제 에셋 사용 중).

**다음 담당:** User — character_view.tscn 시각 검증 재시도 (4·5 ✋ 검증). 통과 시 6단계.

---

## 2026-04-29 14:05 — 실제 face/body 에셋 검증 + 스캐너 단순화 유지 by Claude

**Step:** Roadmap 4·5 검증 후속

**배경:** 사용자가 face_template/sharp_01.png + body_template/male_normal.png 실제 에셋 추가 (AI 생성 → Krita 마커 박음). 사용자가 2px 클러스터로 마커 찍어 스캐너 동작 확인. centroid 방식을 잠깐 시도했으나(`2b0e7d2`) 사용자 판단으로 **오버스펙으로 되돌림(`fc058d7`)** — 1픽셀 마커가 사양/계약이고 브러시 정확도는 사용자 책임.

**검증 결과 (사용자 sharp_01.png):**
- 5개 마커 모두 인식 (현 top-left 방식: eye_l(-68,-33), eye_r(+65,-33) 1px shift, nose(-1,+24), mouth(-1,+89), headwear(+2,-228))
- 비율 자연스러움 (정수리→눈 195px, 눈→코 57px, 코→입 65px)
- 파일 사이즈: face 70KB, body 137KB (body는 tinypng 선택사항)

**변경/생성 파일:**
- (Godot `fc058d7`, master ff-merge 완료) `asset-generater/scripts/character_view.gd` — `_scan_anchors` 그대로 좌상→우하 첫 매칭 (중간에 centroid 시도했으나 revert)
- 자산 (사용자가 main 프로젝트에 직접 추가): `assets/parts/face_template/sharp_01.png`, `assets/parts/body_template/male_normal.png`

**미해결 질문:** 없음

**리뷰 필요:** no — 코드 변화는 결과적으로 없음 (try → revert).

**다음 담당:** User — Godot 에디터에서 character_view.tscn 씬 실행해서 실물 합성 확인. 4·5단계 검증 포인트 ✋ 충족 여부 (눈/코/입 마커 위치에 플레이스홀더가 붙는지) 보고. 통과 시 6단계 진행.

---

## 2026-04-29 12:10 — 캐릭터 JSON 스키마 일관화: scale 전 파츠 + bodyTemplate/faceTemplate 객체 승격 by Claude

**Step:** Roadmap N/A (스키마 변경)

**배경:** 사용자 검토에서 파츠 간 scale 일관성 문제 발견. 4개 파츠(bodyTemplate/faceTemplate/skinMarking/faceOverlay)가 scale 누락. 추가로 bodyTemplate/faceTemplate은 객체가 아닌 단순 id 문자열이라 다른 파츠와 구조 불일치. 캐릭터별 비율 다양화는 head.scale + 파츠별 scale 곱셈 합성으로 처리.

**스키마 변경 (캐릭터 JSON):**
```json
"bodyTemplate": "male_normal"
↓
"bodyTemplate": { "id": "male_normal", "scale": 1.0 }

"faceTemplate": "sharp_01"
↓
"faceTemplate": { "id": "sharp_01", "scale": 1.0 }

"head": { "offsetY": 0, "rotationZ": 0.0 }
↓
"head": { "offsetX": 0, "offsetY": 0, "scale": 1.0, "rotationZ": 0.0 }

"skinMarking": { "id": "...", "color": "..." }
↓
"skinMarking": { "id": "...", "scale": 1.0, "color": "..." }

"faceOverlay": { "id": "...", "side": "...", "color": "..." }
↓
"faceOverlay": { "id": "...", "side": "...", "scale": 1.0, "color": "..." }
```

**변경/생성 파일:**
- (Godot `45219e7`, master에 ff-merge 완료) `asset-generater/data/characters/char_knight_01.json` — 신스키마 적용
- (Godot `45219e7`) `asset-generater/scripts/character_view.gd` — `_get_obj()` 헬퍼 추가, faceTemplate/bodyTemplate 객체 접근, head.scale + offsetX 적용, faceOverlay scale 반영

**기획 문서 갱신 (사용자 직접):**
- ⚠️ `04_Data_Model/41_Character_Data_Schema.md` — 스키마 예시 갱신 (위 5개 변경 반영)
- 권장: `00.Index/03_Decision_Log.md`에 `2026-04-29 — 모든 파츠 scale 일관화 + 템플릿 객체 승격` 항목

**미해결 질문:** 없음

**리뷰 필요:** yes — 트리거: schema 변경 (캐릭터 JSON 구조). Codex가 다른 캐릭터 JSON 추가될 때 신스키마 따르는지 + 41 문서 동기화 검증.

**다음 담당:** User — 41 스키마 문서 직접 갱신. 추가 캐릭터 JSON 만들 때 신스키마 사용. 그 후 Codex 리뷰.

---

## 2026-04-29 11:30 — 앵커 마커 단순화 (hair/helmet 통합 → headwear) by Claude

**Step:** Roadmap N/A (스키마/설계 변경)

**배경:** 사용자 결정 — 기존 3개 Y-only 마커(hair_front #FF00FF, hair_side/back #00FFFF, helmet #FF8000)는 모두 Y만 쓰고 캐릭터별 offset/scale fine-tune이 들어가므로 단일 정수리 마커로 충분. 마커 7→5로 줄어 face_template 제작 부담 감소.

**새 마커 사양:**
- `headwear` = #FF00FF (마젠타) at 정수리 1픽셀
- 용도: Hair_Front, Hair_Side, Hair_Back, Helmet 모두 공용 baseline Y

**변경/생성 파일:**
- (Godot `5589d96`, master에 ff-merge 완료) `asset-generater/scripts/character_view.gd` — `ANCHOR_MARKERS` 7→5, `_setup_hair` 3개 모두 `headwear` Y 사용, `_setup_helmet` 도 동일
- (Godot `5589d96`) `asset-generater/scripts/tools/generate_templates.gd` — 부트스트랩 마커 5개로, headwear offset (0, -180)

**기획 문서 갱신 (사용자 직접 — "문서가 진실" 원칙):**
- ⚠️ `02_System_Architecture/22_Layer_System.md` — "Anchor System" 섹션 마커 표 5개로 축소
- ⚠️ `02_System_Architecture/23_Anchor_System.md` — "마커 색상 규칙" 섹션 5개로, "주요 Anchor"에서 Hair/Helmet 항목 통합
- 권장: `00.Index/03_Decision_Log.md`에 `2026-04-29 — 앵커 마커 hair/helmet 통합 결정` 항목 추가

**미해결 질문:** 없음

**리뷰 필요:** yes — 트리거: schema 변경 (앵커 시스템 마커 정의). Codex가 char_knight_01.json 등 캐릭터 데이터에 영향 없는지(없음, 마커는 face_template asset에만 박힘) + 기획 문서 동기화 검증.

**다음 담당:** User — 위 ⚠️ 기획 문서 2건 직접 수정. 그 후 Codex가 리뷰. 진행 중인 face_template/sharp_01.png Krita 작업도 마커 5개만 박으면 됨 (기존 3개 hair/helmet 마커 위치엔 박지 말 것).

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