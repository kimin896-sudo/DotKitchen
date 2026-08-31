# 07. 기술 구현 우선순위

## 초기 구현 우선순위

| 순위 | 시스템 | 설명 | 선행 의존 |
|------|--------|------|-----------|
| **1** | Grid/Tilemap 동선 | 손님·플레이어 이동, 테이블·주방 기구 배치 | — |
| **2** | 손님 AI (FSM) | 입장→대기→주문→음식대기→식사→결산→퇴장/이탈 | Grid |
| **3** | 데이터 (SO/CSV) | 레시피, 손님 유형, 재료, 해금 테이블 | — |
| **4** | 메뉴판 & 조리 | 4스탯 기반 조리 파이프라인 | 데이터, Grid |
| **5** | Day 루프 & 결산 | 준비→영업→결산 UI/로직 | FSM, 조리 |
| **6** | 메타 성장 | 특성 트리, 레시피·장비 해금 | Day 루프, 데이터 |
| **7** | 세트 시너지 | 카테고리 조합 버프 계산 | 메뉴판, 데이터 |

---

## MVP 로드맵

### Phase 1 — 프로토타입

- [ ] Grid/Tilemap 기반 맵 + 플레이어 이동
- [ ] 손님 FSM (입장 → 주문 → 대기 → 퇴장/이탈)
- [ ] 기본 1~2종 요리 조리·서빙
- [ ] 대기 시간 게이지 UI

### Phase 2 — Day 루프

- [ ] 준비 단계: 메뉴 슬롯 지정
- [ ] 영업 단계: 손님 스폰 → 주문 → 조리 → 제공
- [ ] 결산 단계: 일일 매출·만족도·이탈 손님 집계
- [ ] Day 시작/종료 전환

### Phase 3 — 데이터 & 콘텐츠

- [ ] RecipeData ScriptableObject 스키마
- [ ] CustomerData ScriptableObject 스키마
- [ ] 카테고리 5종 + 기본 메뉴 등록
- [ ] 재료·기구 데이터

### Phase 4 — 메타 성장

- [ ] 30일 종료 → Rank 산정
- [ ] 명예 포인트 → 레시피 해금 1종
- [ ] 특성 트리 UI (1카테고리)
- [ ] 회차 간 저장/로드

### Phase 5 — 확장

- [ ] 세트 시너지 시스템
- [ ] 하이브리드 빌드 지원
- [ ] 알바생 AI
- [ ] 인테리어/테이블 배치
- [ ] 전체 특성 트리

---

## 핵심 데이터 스키마 (초안)

### RecipeData

```csharp
[CreateAssetMenu(fileName = "Recipe", menuName = "DotKitchen/Recipe")]
public class RecipeData : ScriptableObject
{
    public string id;
    public string displayName;
    public FoodCategory category;

    public float prepTime;      // 손질/전처리 (초)
    public float cookTime;      // 조리/불질 (초)
    public float serveTime;     // 포장/서빙 (초)
    public int price;
    public bool canPreMake;     // 사전 준비 가능 여부

    public IngredientCost[] ingredients;
    public string[] requiredEquipment;
    public int unlockCost;      // 명예 포인트
}
```

### CustomerData

```csharp
[CreateAssetMenu(fileName = "Customer", menuName = "DotKitchen/Customer")]
public class CustomerData : ScriptableObject
{
    public string id;
    public float patienceDuration;  // 대기 게이지 총량 (초)
    public float tipChance;
    public FoodCategory[] preferredCategories;
}
```

### TraitData

```csharp
[CreateAssetMenu(fileName = "Trait", menuName = "DotKitchen/Trait")]
public class TraitData : ScriptableObject
{
    public string id;
    public string displayName;
    public TraitTree tree;        // Korean, Western, Chinese, FastFood, Common
    public int unlockCost;
    public TraitEffect[] effects;
}
```

---

## 손님 FSM 상태 전이 (구현 참고)

```
State: Entering
  → OnReachSeat: Seated (if table available)
  → OnNoTable: Waiting

State: Waiting
  → OnTableAvailable: Seated
  → OnPatienceExpired: Leaving (이탈)

State: Seated
  → OnOrderPlaced: WaitingForFood

State: WaitingForFood
  → OnFoodServed: Eating
  → OnPatienceExpired: Leaving (이탈)

State: Eating
  → OnEatComplete: Paying

State: Paying
  → OnPaymentComplete: Leaving

State: Leaving
  → OnExit: (destroy / pool return)
```

---

## 참고 문서

- [01-overview.md](./01-overview.md) — 게임 루프
- [02-core-systems.md](./02-core-systems.md) — 손님 메커니즘, 재화
- [04-recipes-and-ingredients.md](./04-recipes-and-ingredients.md) — 4스탯, 재료표
