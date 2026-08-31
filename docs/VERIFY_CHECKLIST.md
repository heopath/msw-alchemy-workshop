# 검증 체크리스트

> T3~T7을 만드는 동안 확인 항목을 누적하는 문서. 사람이 마지막에 이 목록 하나로 전체를 검증한다.
> 작성 규칙은 [BUILD_BRIEF.md](BUILD_BRIEF.md) 0절 참고.

## 검증하는 법

1. 메이커에서 **확인할 맵을 먼저 연다** (재생 버튼은 편집기에 열려 있는 맵을 실행한다)
2. 재생하고 각 항목의 조작을 해본다
3. 기대대로면 `[x]`, 아니면 그대로 두고 아래 **어긋난 것** 표에 적는다
4. 전부 훑은 뒤 한 번에 수정을 요청한다

재생을 끄고 나서 수정을 요청해야 한다. 켜둔 채로 시키면 파일 갱신이 막힌다.

---

## T1 — 헤네시스 맵

- [x] **맵이 열리고 좌우 이동이 된다** — 확인 완료 2026-08-31
- [ ] **양끝 벽에서 막힌다**
  어디서: MapHenesys
  조작: 좌우 끝까지 걸어간다
  기대: 화면 밖으로 나가지 않는다
  실패하면: 맵의 foothold 데이터와 벽 배치

## T2 — 거대 버섯 보스

- [ ] **두 패턴이 번갈아 나온다**
  어디서: MapHenesys, 스폰 지점에서 오른쪽 (보스는 x=5)
  조작: **때리지 말고** 10초간 지켜본다
  기대: 내려찍기(빨갛게 0.5초 → 착지) → 3초 → 포자 3발 부채꼴 → 5초 → 반복
  실패하면: `GiantMushroomAI.mlua`의 패턴 스케줄링
- [ ] **피해가 들어가고 체력이 준다**
  조작: 보스를 공격한다
  기대: 데미지 숫자가 뜨고 콘솔에 `Hp=.../200`
  실패하면: `GiantMushroom.model`의 HitComponent, 충돌 그룹
- [ ] **2페이즈로 넘어간다**
  조작: 체력을 100 이하로 만든다
  기대: `GiantMushroom entered phase 2` 로그, 내려찍기 간격 3초 → 2초
  실패하면: `GiantMushroomAI.mlua`의 페이즈 판정
- [ ] **처치 신호가 나간다**
  조작: 체력을 0으로 만든다
  기대: 사망 클립 후 소멸, `BossDefeatedEvent sent: region=Henesys` 로그
  실패하면: `BossDefeatedEvent.mlua`
- [ ] **난이도가 적당한가** (수치 판단)
  참고: 체력 200, 플레이어 공격 50 → 네 대면 죽는다. 짧다고 느끼면 체력을 올린다.

---

## 어긋난 것

검증하다 기대와 다른 것을 여기에 적는다. 전부 훑은 뒤 한 번에 고친다.

| 항목 | 실제로 일어난 일 | 비고 |
|---|---|---|
| | | |

---

## T3 — 포자 발판 능력화

설치 키는 **F**입니다 (기본 조작인 방향키·Alt·Space와 겹치지 않게 골랐습니다).

- [ ] **능력 획득 전에는 설치되지 않는다**
  어디서: MapHenesys
  조작: 보스를 잡기 전에 F를 누른다
  기대: 아무것도 생기지 않고 콘솔에 `place refused, ability 'SporePad' not owned yet`
  실패하면: `Ability/AbilityLogic.mlua`의 `HasAbility` / `OwnedAbilities` 동기화
- [ ] **보스를 잡으면 능력을 얻는다**
  조작: 거대 버섯을 처치한다
  기대: 콘솔에 `AbilityLogic granted ability 'SporePad' for clearing region 'Henesys'`
  실패하면: `GiantMushroomAI.AnnounceDefeat` → `AbilityLogic.ReportBossDefeated`, `Data/AbilityTable.csv`의 `regionId` 열
- [ ] **획득 후 F로 발밑에 설치되고 밟으면 튀어오른다**
  조작: 보스를 잡은 뒤 F를 누르고 그 자리를 밟는다
  기대: 발판이 생기고 위로 튕긴다. 콘솔에 `SporePad placed` → `SporePad bounced entity`
  실패하면: `SporePad/SporePadComponent.mlua`, `Models/MapObjects/SporePad.model`의 TriggerComponent
- [ ] **공중에서는 설치되지 않는다**
  조작: 점프 중에 F를 누른다
  기대: 설치되지 않고 `place refused, player is not on the ground`
  실패하면: `AbilityLogic.RequestPlaceAbility`의 `IsOnGround` 판정
- [ ] **동시에 2개까지만 남는다**
  조작: 위치를 옮겨가며 F를 세 번 누른다 (쿨다운 3초를 기다리면서)
  기대: 세 번째를 놓는 순간 가장 먼저 놓은 것이 사라진다. `pad cap 2 reached, removing ...`
  실패하면: `AbilityLogic.EnforcePadCap`
- [ ] **쿨다운 3초가 걸린다**
  조작: F를 연타한다
  기대: 3초에 한 번만 설치된다. `place refused, 'SporePad' on cooldown`
  실패하면: `AbilityLogic.RequestPlaceAbility`의 쿨다운, `AbilityTable.csv`의 `cooldown` 열
- [ ] **20초 뒤 저절로 사라진다**
  조작: 설치하고 20초 기다린다
  기대: 발판이 사라지고 `SporePad expired`
  실패하면: `SporePadComponent.Expire`, `AbilityTable.csv`의 `duration` 열

## T4 — 페리온 맵

`MapHenesys`를 복제했으므로 지형·벽 조건이 T1과 완전히 같습니다. T1의 벽 항목이 실패하면 여기도 같이 실패합니다.

- [ ] **맵이 열리고 좌우 이동이 된다**
  어디서: MapPerion
  조작: 좌우로 걷는다
  기대: 지형 위를 걸어다닌다
  실패하면: `Global/SectorConfig.config`의 `map://mapperion` 등록, `map/MapPerion.map`
- [ ] **양끝 벽에서 막힌다**
  조작: 좌우 끝(x ≈ -8.9 / +8.0)까지 걸어간다
  기대: 화면 밖으로 나가지 않는다
  실패하면: T1과 동일 — 지형 타일을 메이커에서 칠해야 합니다

## T5 — 바위 전사 보스

- [ ] **좌우로 돌진한다**
  어디서: MapPerion, 스폰 지점에서 오른쪽 (보스는 x=5)
  조작: 때리지 말고 지켜본다
  기대: 2.5초마다 `주황색으로 물듦 0.6초 → 직선 돌진`. 콘솔에 `telegraphing charge` → `charging`
  실패하면: `Boss/RockWarriorAI.mlua`의 상태 기계
- [ ] **벽에 부딪히면 1.5초 경직된다**
  조작: 돌진이 맵 끝까지 가게 둔다
  기대: 파랗게 물들고 멈춘다. `RockWarrior staggered by wall for 1.5s`
  실패하면: `RockWarriorAI.TickCharge`의 정지 감지(`BlockedEpsilon`). **T1/T4의 벽이 없으면 이 항목은 반드시 실패합니다** — 대신 `charge timed out with no wall hit`이 찍힙니다
- [ ] **경직 중 피해가 3배로 들어간다**
  조작: 경직된 동안 때린다
  기대: 데미지 숫자가 평소(50)의 3배로 체력이 줄어든다. 콘솔 `Hp=.../300` 감소폭 비교
  실패하면: `Monster.mlua`의 `DamageMultiplier`, `RockWarriorAI.SetDamageMultiplier`
- [ ] **공중에 뜨면 돌진에 맞지 않는다**
  조작: 돌진이 오는 순간 점프한다
  기대: 피해를 입지 않는다
  실패하면: `RockWarriorAttack.IsAttackTarget`의 `IsOnGround` 판정
- [ ] **2페이즈로 넘어간다**
  조작: 체력을 150 이하로 만든다
  기대: `RockWarrior entered phase 2`, 돌진 주기 2.5초 → 1.8초
  실패하면: `RockWarriorAI.CheckPhase`
- [ ] **처치되면 페리온 능력 신호가 나간다**
  조작: 체력을 0으로 만든다
  기대: `BossDefeatedEvent sent: region=Perion`
  실패하면: `RockWarriorAI.AnnounceDefeat`
  참고: `AbilityTable.csv`에 Perion 행이 없으므로 `no AbilityTable row for regionId=Perion` 경고가 같이 납니다. **의도한 상태입니다** — 페리온 능력(돌진 방패)은 프로토타입 범위 밖이고, 지역 클리어 표시는 정상 동작합니다

## T6 — 약점 상성 (이 프로토타입의 핵심)

**이 항목이 기획 전체의 판단 근거입니다.** 나머지가 다 되고 이것만 안 되면 T6부터 다시 봅니다.

- [ ] **돌진 경로에 놓은 발판이 보스를 튕긴다**
  어디서: MapPerion (헤네시스를 먼저 깨서 포자 발판을 얻은 상태여야 합니다)
  조작: 보스가 예고 동작(주황색)을 할 때 돌진 경로 위에 F로 발판을 놓고 비킨다
  기대: 보스가 발판을 밟고 공중으로 튀어오른다. `RockWarrior hit a spore pad` → `launched by spore pad, waiting for landing`
  실패하면: `SporePad/SporePadComponent.mlua`의 `SporePadBounceEvent` 발신, `RockWarriorAI.HandleSporePadBounceEvent`
- [ ] **착지 후 5초간 경직된다**
  조작: 위 항목에 이어서 본다
  기대: 착지하는 순간 파랗게 물들고 `RockWarrior staggered by spore pad for 5.0s`
  실패하면: `RockWarriorAI.TickLaunched`의 착지 판정(`LaunchGrace` / `MaxAirTime`)
- [ ] **그 5초 동안 피해가 3배로 들어간다**
  조작: 경직 중에 계속 때린다
  기대: 벽 경직(1.5초)보다 훨씬 많이 깎인다
  실패하면: T5의 3배 항목과 동일
- [ ] **능력 없이도 잡을 수 있다** (설계 검증)
  조작: 발판을 쓰지 않고 벽 경직(1.5초)만 노려서 잡아본다
  기대: 잡히긴 하지만 훨씬 오래 걸린다
  실패하면: 못 잡으면 벽 경직이 너무 짧고, 쉽게 잡히면 발판의 보람이 없습니다. 수치 조정 대상
- [ ] **재미있는가** (사람의 판단 — 이 프로토타입의 목적)
  헤네시스를 먼저 깨고 얻은 포자 발판으로 페리온을 공략하는 것이 재미있는가?
  재미없으면 지역을 늘리기 전에 수치와 약점 설계를 먼저 손봅니다

## T7 — 지역 선택

시작하면 지역 선택 창이 떠 있습니다. **M 키**로 열고 닫습니다.

- [ ] **지역 선택 창이 뜬다**
  어디서: 아무 맵
  조작: 재생한다
  기대: 화면 가운데에 `지역 선택` 창과 `헤네시스` / `페리온` 버튼 두 개
  실패하면: `ui/RegionSelect.ui`, `Region/RegionSelectUI.mlua`의 바인딩 UUID
- [ ] **두 지역을 오갈 수 있다**
  조작: `페리온` 버튼을 누른다 → M을 눌러 다시 열고 `헤네시스`를 누른다
  기대: 각각 해당 맵으로 이동한다. 콘솔에 `RegionLogic moved ... to region 'Perion' (map MapPerion)`
  실패하면: `Region/RegionLogic.mlua`의 `_TeleportService:TeleportToMapPosition`, `Data/RegionTable.csv`의 `mapName` 열
- [ ] **M 키로 열고 닫힌다**
  조작: M을 두 번 누른다
  기대: 창이 사라졌다 나타난다
  실패하면: `RegionSelectUI.OnKeyDown`
- [ ] **이미 깬 지역이 표시된다**
  조작: 헤네시스 보스를 잡은 뒤 M으로 창을 연다
  기대: 버튼이 `헤네시스  (클리어)`로 바뀐다
  실패하면: `AbilityLogic.ClearedRegions`의 클라이언트 동기화, `RegionSelectUI.BuildLabel`

---

## 아직 확인 못 한 것 (만든 사람이 아는 위험)

재생을 못 해서 아래는 전부 미확인입니다. 검증하다 이상하면 여기부터 보세요.

### 1. 빌드 경고 LWA-4012 두 건 (남아 있음)

```
model://sporeshot   | Damage        | SporeShot
model://rockwarrior | ContactDamage | RockWarriorAttack
```

스크립트 `property`와 `.model` 저장값의 타입이 어긋난다는 경고입니다. `integer` → `number`로 바꾸고
`.model`에서 해당 값을 아예 빼는 것까지 했는데도 로그에서 사라지지 않았습니다 (거대 버섯 쪽 같은 경고는
같은 처방으로 사라졌습니다). 갱신 후 빌드 로그가 다시 생성되지 않는 것일 수도 있습니다.

**실제로 문제가 되는지 확인하는 법:** 포자에 맞았을 때 8, 바위 전사에게 닿았을 때 20이 아니라 **1**이
뜨면 값이 안 붙은 것입니다. 그러면 `SporeShot.mlua` / `RockWarriorAttack.mlua`의 property 기본값을
직접 고치면 됩니다 (지금도 기본값은 각각 8, 20으로 맞춰져 있습니다).

### 2. 벽이 아직 없습니다 (T1에서 넘어온 문제)

T1의 벽 항목이 미확인 상태입니다. 벽이 없으면 **T5의 벽 경직**과 **T6의 능력 없이 공략** 항목이
같이 실패합니다. 바위 전사는 `charge timed out with no wall hit`만 반복합니다.

### 3. 페리온 능력은 없습니다

`AbilityTable.csv`에 Perion 행이 없습니다. 페리온을 깨면 지역 클리어는 표시되지만 능력은 안 나오고
경고가 뜹니다. 프로토타입 범위(상성 한 쌍 검증)에는 필요 없어서 의도적으로 비웠습니다.

### 4. 스킬 프레임워크를 통째로 만들지 않았습니다

`maplestory-skill-maker` 스킬은 `SkillCatalogLogic` + 공격 스킬 DataSet + 핫바 UI로 이어지는 큰
틀을 요구합니다. 포자 발판은 피해가 없는 설치형이라 그 스키마(damage/hitDelay/projectileSpeed…)에
맞지 않고, 틀을 세우는 것이 T3~T7 범위를 훨씬 넘어섭니다.

지시서의 실제 요구인 **"스킬 테이블을 스크립트에 박아넣지 않는다"** 는
`Data/AbilityTable.userdataset` + `.csv`로 지켰습니다 (튕김 세기·지속·동시 개수·쿨다운·지역 매핑이
전부 이 표에 있고 `_DataService`로 읽습니다). 틀 전체가 필요하면 말씀해주세요.
