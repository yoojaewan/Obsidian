# Layer System

## 기본 레이어 순서
1. BodyBase
2. FaceBase
3. Eyes
4. Nose
5. Mouth
6. Beard
7. Hair_Back
8. Hair_Side
9. Hair_Front
10. Armor
11. Helmet_Back
12. Helmet_Front

## 원칙
- Hair는 반드시 Front / Side / Back 분리
- Helmet은 Front / Back 분리 가능 구조
- Face 파츠는 FaceTemplate anchor를 기준으로 배치
- Beard는 Mouth anchor Y 공유, X 중앙 고정, 선택 파츠(null 허용)
- 액세서리 레이어는 현재 범위 제외
