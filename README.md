# 🌙 Lunar Realm (루나 렐름)

[🎥 시연 영상 보기](https://www.youtube.com/watch?v=EVAGW5MIVKE&t=289s)

> **"압도적인 물량의 쾌감과 전략적 디펜스의 조화"**
> **팀 행운의 토끼발 (블랙스톰 기업 협업 프로젝트)**

![Project Status](https://img.shields.io/badge/Status-Completed-green)
![Engine](https://img.shields.io/badge/Unreal%20Engine-5.6-black?style=flat-square&logo=unrealengine)
![Language](https://img.shields.io/badge/Language-C++%20%7C%20Blueprint-blue?style=flat-square&logo=cplusplus)
![Platform](https://img.shields.io/badge/Platform-PC%20%7C%20Mobile-lightgrey?style=flat-square)

<br>

## 📖 1. 프로젝트 개요 (Overview)

**Lunar Realm**은 서브컬처 감성의 RPG와 **자동 디펜스(Auto-Defense)** 장르를 결합한 게임입니다. 플레이어는 메인 캐릭터를 직접 조작하거나 자동 전투를 통해 몰려오는 대규모 적 웨이브를 막아내고, 아군 유닛을 소환하여 적의 기지를 파괴해야 합니다.

* **장르**: RPG + 자동 디펜스 (오토 배틀/방치형 요소 포함)
* **시점**: 가로 사이드뷰 (2.5D/3D 리소스 기반)
* **핵심 컨셉**: 다수의 적을 처치하는 **Hack & Slash**의 쾌감 + 아군 멤버/장비 조합을 통한 **전략적 디펜스**
* **레퍼런스**: 냥코 대전쟁, 팔라독, 명일방주, 원신, 트리컬:리바이브

<br>

## ✨ 주요 기능 및 코드 (Key Features)
> 각 기능의 상세 구현 코드는 아래 링크를 통해 확인할 수 있습니다.

* **[플레이어 데이터 파이프라인 및 상태 동기화](https://github.com/jackbongmin/Lunar_Realm/blob/main/Source/Lunar_Realm/Private/Units/Player/LRPlayerCharacter.cpp)**
  * Enhanced Input을 통한 조작 구현 및 애니메이션 블루프린트(ABP) 액션 스테이트 동기화.
  * 시각적 처리(`Character`)와 데이터 논리(`PlayerState`)를 분리하고 다중 서브시스템 연동을 통한 동적 스탯 연산 구축.
* **[동료(Member) 캐릭터 동적 소환 및 풀링 생명주기 관리](https://github.com/jackbongmin/Lunar_Realm/blob/main/Source/Lunar_Realm/Private/Units/Player/Component/LRSummonComponent.cpp)**
  * 자원(에테르) 및 쿨타임 검증을 통한 동적 소환(`ULRSummonComponent`) 및 코어 기반 무작위 스폰 좌표 연산.
  * 글로벌 오브젝트 풀링을 도입하여 동료 캐릭터 파괴/생성 비용을 제거하고, 재활성화 시 AI 컨트롤러 블랙보드를 명시적으로 리셋(`ResetAIController`)하여 로직 꼬임 방지.
* **[FSM 기반 독자적 자동 전투(Auto) 시스템](https://github.com/jackbongmin/Lunar_Realm/blob/main/Source/Lunar_Realm/Private/Units/Player/Component/LRCombatComponent.cpp)**
  * 컨트롤러 빙의 교체 없이 캐릭터에 부착된 FSM(Finite State Machine) 형태의 가벼운 전투 컴포넌트(`ULRCombatComponent`) 설계. Tick 최적화를 위해 일정 주기(0.2초) 타이머 기반 로직 처리.
* **[모바일 맞춤형 카메라 매니저 및 뷰포트 클램핑](https://github.com/jackbongmin/Lunar_Realm/blob/main/Source/Lunar_Realm/Private/Units/Player/LRPlayerCameraManager.cpp)**
  * `FMath::VInterpTo`를 활용한 카메라 래깅(Lagging) 시스템 및 맵 경계선 이탈 방지를 위한 뷰포트 클램핑(`FMath::Clamp`) 로직 구축.
* **[상태 기반 고급 시각 피드백 (데드 스테이트 및 체력바)](https://github.com/jackbongmin/Lunar_Realm/blob/main/Source/Lunar_Realm/Private/UI/InGame/LRHealthWidget.cpp)**
  * 사망 시 RetainerBox 머티리얼 동적 교체를 통한 흑백 UI 연출, 그리고 `FInterpTo` 기반의 잔상(Ghost) 체력바 로직 및 위기 상황 동적 시각화 구현.
* **[데이터 기반 비동기 로딩 시스템 (Async Loading)](https://github.com/jackbongmin/Lunar_Realm/blob/main/Source/Lunar_Realm/Private/Core/Transition/LRTransitionGameMode.cpp)**
  * `StreamableManager`를 활용하여 세이브 데이터와 스테이지 데이터를 교차 검증하고, 런타임에 필요한 에셋만 선별적으로 로드하여 메모리 최적화 구축.

<br>

## 🔄 3. 게임 루프 & 시스템 (Core Loop)

### 🎮 조작 방식 (Controls)
* **Manual (수동)**
    * **이동**: 가상 조이패드 (PC: `WASD`)
    * **공격/스킬**: 액티브 스킬 버튼 터치 (PC: 키보드 단축키)
* **Auto (자동)**
    * **AI 모드**: 오토 토글 버튼을 통해 활성화. 독자적 FSM 컴포넌트가 타겟 우선순위 규칙에 따라 자동으로 이동, 소환, 스킬 사용.
    * **방치형 플레이**: 반복 파밍 시 플레이어의 피로도를 낮추는 "보는 재미" 제공.

### 🔄 핵심 루프 (Core Loop)
1.  **준비**: 장비 장착, 스킬 세팅, 유닛 덱 편성
2.  **전투**: 웨이브 디펜스 & 적 기지 파괴
3.  **보상**: 스테이지 클리어 및 재화 획득
4.  **성장**: 가챠(캐릭터/장비), 강화, 장비 교체, 세트 효과 활성화
5.  **반복**: 다음 스테이지 도전 및 파밍

<br>

## ⚙️ 4. 주요 시스템 (Key Systems)

### ⚔️ 인게임 시스템 (In-Game)
* **캐릭터 구성**: 메인 플레이어블 캐릭터 1명 + 소환 유닛 다수
* **소환 시스템**: GAS Attribute(에테르) 연동 실시간 자원 검증. 아군 코어(`ALRPlayerCore`) 위치 기반 동적 스폰 좌표 연산.
* **배속 및 UI 동기화**: `SetGlobalTimeDilation` 기반 1.5x, 2.0x 배속 순환 제어 및 오토 모드-UI 예외 처리.
* **디펜스 코어 시스템**: 수학적 보간(`FMath::Lerp`)을 활용한 시네마틱 붕괴 연출 및 객체 지향 기반 파괴 흐름 제어.

### 🎒 아웃게임 시스템 (Out-Game)
* **가챠 (Gacha)**: 캐릭터 및 장비 소환.
* **편성 (Deck Building)**: 메인 캐릭터와 시너지를 낼 소환 유닛 선택.
* **성장 UI 파이프라인**: 첫 클리어/패배 등 조건에 따른 경험치 배율 동적 할당 및 `FInterpConstantTo`를 활용한 부드러운 게이지 상승/레벨업 이월 로직.

<br>

## 🏗 프로젝트 구조 및 컨벤션 (Structure & Convention)

### 에셋 관리 방식 (Asset Management)
* **네이밍 컨벤션:** 언리얼 엔진 표준 네이밍 컨벤션 준수 (예: `T_` Texture, `M_` Material, `SK_` SkeletalMesh).
* **데이터 관리:** 하드코딩을 지양하고 `Data Table`을 활용하여 캐릭터, 스킬, 장비, 에너미 ID 등 핵심 에셋을 FName 도메인 기반으로 통합 파싱 및 관리.

### 코드 스타일 및 아키텍처 (Code Style & Architecture)
* **아키텍처:** Gameplay Ability System (GAS) 기반 전투 모듈화 및 글로벌 오브젝트 풀링(`UPoolingSubsystem`)을 통한 라이프사이클 최적화.
* **파라미터 네이밍:** 함수 파라미터는 명시적인 구분을 위해 반드시 `In` 접두사를 사용. (예: `InDamage`)
* **괄호 및 띄어쓰기 규칙:** 괄호 안팎으로는 공백을 두지 않으며, 콤마(,) 뒤에만 띄어쓰기를 적용하여 가독성 통일.
  * ⭕ 올바른 예: `ApplyDamage(float InDamage, float InMultiplier)`
  * ❌ 잘못된 예: `ApplyDamage( float InDamage , float InMultiplier )`
* **안전한 참조 보장:** 오브젝트 풀링 등 동적 해제가 빈번한 환경을 고려하여 액터 참조 시 무분별한 `TObjectPtr` 대신 `TWeakObjectPtr` 활용.

<br>

## 🛠 Tech Stack
### Core Engine
* **Unreal Engine 5.6** (C++ Base)
* **IDE:** Visual Studio 2022 / Rider

<br>

## 👥 Team (행운의 토끼발)
| 이름 (Name) | 포지션 (Role) | 담당 상세 (Responsibilities) |
| :---: | :---: | :--- |
| **백종민** | **Player & System Opt** | • **Player**: Enhanced Input 조작, ABP 동기화 및 `PlayerState` 기반 데이터 파이프라인 구축<br>• **In-Game**: 동료 캐릭터(Member) 동적 소환, 풀링 생명주기 관리 및 AI 컨텍스트 리셋 로직<br>• **Auto Combat**: BT 없는 가벼운 FSM 자동 전투 컴포넌트(`ULRCombatComponent`) 설계<br>• **UI / UX**: 방사형 MID 쿨타임, 흑백 데드 스테이트, 배속 제어 등 고급 시각 연출 연동<br>• **Optimization**: StreamableManager 비동기 로딩, 카메라 클램핑, TWeakObjectPtr 크래시 대응 |
| **박준범** | **Core / Architecture** | • **Game Flow**: `GameMode`/`GameInstance` 기반 게임 루프 및 스테이지 관리<br>• **Level System**: 레벨 스트리밍 및 스테이지 전환 로직 구현 |
| **김우빈** | **AI & Combat** | • **AI Logic**: Behavior Tree/Blackboard 기반 적 NPC 행동 패턴 설계<br>• **Battle System**: 전투 판정 및 상태(State) 관리, 몬스터 스폰 로직 |
| **김하신** | **System & Collection** | • **Equipment**: 장비 데이터 구조 설계 및 장착 스탯 반영<br>• **Subsystems**: 도감(Collection) 및 인게임 매니저 구현 |
| **박용익** | **Economy & Content** | • **Gacha System**: 확률 테이블 기반 소환 알고리즘 및 연출 로직<br>• **Currency**: 재화 획득/소모 로직 및 인벤토리 관리 |

<br>

## 📂 Installation (실행 환경)
본 프로젝트는 포트폴리오 목적으로 별도의 빌드 패키지(.exe)를 배포하지 않습니다. 소스 코드 빌드 및 확인을 위한 환경은 아래와 같습니다.

1. **Prerequisites**
* Unreal Engine 5.6
* Visual Studio 2022
