# 코드 구조

`Assets/Scripts` 134개 · `Assets/Editor` 32개. 전부 C#, 외부 라이브러리 없음.

게임 이름과 조작은 저장소 루트의 [`README.md`](../../README.md) 를 보세요.

---

## 씬은 넷

| 씬 | 무엇 | 짓는 방식 |
|---|---|---|
| `Scenes/Lobby.unity` | 중앙홀 — 카트 고르기 · 이야기 · 타이틀 | 구워서 저장 |
| `Scenes/Track.unity` | 레이스 | **실행할 때 지어진다** |
| `Scenes/Gallery.unity` | 전시실 | 구워서 저장 |
| `Scenes/Campus.unity` | 걸어다니는 캠퍼스 | 구워서 저장 |

트랙만 `Awake` 에서 짓고 나머지 셋은 에디터에서 미리 지어 저장합니다.
이야기 장면과 타이틀 화면은 **별도 씬이 아니라 중앙홀 안에서** 돕니다.

---

## 진행 상태는 두 개뿐

저장되는 값은 이 둘이고(`PlayerPrefs`), 나머지는 전부 여기서 파생됩니다.

| | 어디 |
|---|---|
| 모은 증거 수 | `CollectionState.Count` |
| 결승 클리어 여부 | `GrandFinal.Cleared` |

```
CollectionState.Count + GrandFinal.Cleared
   ├─ MissionManager.CurrentGoal   이번 판에 할 일
   ├─ AiRaceGate                   상대 카트를 내보낼지
   ├─ DebrisGate · CargoGate       그 판 전용 장애물 · 화물
   ├─ CampusMood · CampusBoarding  폐과 딱지 · 못질한 판자
   ├─ CampusFestival · CampusVictory  축제 · 광고판과 동상 철거
   └─ GalleryMood · GalleryLights  먼지 덮개 · 전시실 점등
```

게이트들이 컴포넌트 값을 읽지 않고 **수집 기록을 직접 보기 때문에**,
오래 전에 구워 둔 씬에서도 규칙이 바르게 돕니다.

---

## 주요 파일

### 주행

| 파일 | 하는 일 |
|---|---|
| `KartController` | 레이캐스트 서스펜션 · 구동 · 드리프트 · 충돌 |
| `KartInput` | 입력 (Input System 전용) |
| `KartCamera` | 3인칭 백뷰 · 속도에 따른 시야각과 거리 |
| `KartAi` | 코스 경로를 따라가는 상대 카트 |
| `KartSkin` · `KartSpec` | 캐릭터별 모델과 성능 |

### 코스와 세계

| 파일 | 하는 일 |
|---|---|
| `TrackBuilder` | 코스 535 m · 노면 · 담장 · 발판 · 결승 아치 |
| `CampusBuilder` | 건물 13동 · 내부 · 소품 · 담장 |
| `HanokRoof` · `HanokDoor` | 한옥 지붕과 문 (세 씬이 공유) |
| `Surfaces` · `FlatMaterial` | 재질 7종. **색이 마감을 정한다** |

### 진행과 이야기

| 파일 | 하는 일 |
|---|---|
| `MissionManager` | 아홉 판의 판정 |
| `ExhibitCatalogue` | 증거 8종. **이 순서가 곧 임무 순서** |
| `StoryScript` | **대사 전체.** 장면 16개 · 370줄 |
| `DialogueRunner` · `DialogueHUD` | 재생과 그리기 |
| `StoryAlbum` | 곰 기억 앨범 (`B`) |

### 미니게임

| 파일 | 하는 일 |
|---|---|
| `MinigameFlow` | 준비 · 카운트다운 · 일시정지 · 등급 (셋이 공유) |
| `Canteen*` | 오늘의 급식 (완성) |
| `SafetyDrill*` | 안전 점검 훈련 |
| `Tracing*` | 따라 그리기 |

### 화면

`Hud` 가 팔레트 · 글꼴 · 나무 판을 들고 있고, 각 HUD 는 배치만 합니다.
좌표는 전부 **가상 1280×720** 이며 `Hud.Begin` 이 화면 크기에 맞춰 배율을 겁니다.

---

## 손댈 때 주의할 것

- **입력은 Input System 전용.** `Input.GetKey` 는 컴파일은 되지만 실행 중에 예외가 납니다.
  `Keyboard.current` 를 쓰거나 `KartInput` · `Keys` 를 거치세요.
- **카트와 캐릭터에 `MeshCollider` 를 붙이지 마세요.** 충돌은 루트의 `BoxCollider` 가 맡습니다.
  (도로와 담장의 `MeshCollider` 는 정상입니다.)
- **움직일 물건에 `isStatic` 을 달지 마세요.** 정적 배칭에 묶이면 좌표를 옮겨도 화면이 안 바뀝니다.
- **회전한 오브젝트를 월드 AABB 로 재지 마세요.** 로컬 좌표로 내려서 재야 맞습니다.
- 구워서 저장하는 씬(중앙홀 · 전시실 · 캠퍼스)은 빌더를 다시 돌리면 덮어써집니다.
  손으로 놓은 물건은 `Editor/MyProps.cs` 가 되살립니다.

---

## 개발용 키

`F7~F12` 등 디버그 키는 `Dev.Enabled` 뒤에 있고, 이 값은 `#if UNITY_EDITOR` 라
**배포 빌드에서는 상수 `false`** 입니다. 해당 코드는 컴파일 단계에서 사라지므로
배포본에는 들어가지 않습니다.
