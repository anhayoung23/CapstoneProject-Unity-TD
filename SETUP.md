# CDP2 Layer Playground — 처음부터 만드는 과정

> 이 문서는 프로젝트를 처음부터 새로 구성할 때 참고하는 단계별 가이드입니다.

---

## 목차

1. [사전 준비](#1-사전-준비)
2. [Unity 프로젝트 초기 설정](#2-unity-프로젝트-초기-설정)
3. [씬 자동 생성 (ProtoSceneBuilder)](#3-씬-자동-생성-protoscenebuilder)
4. [영상 클립 연결](#4-영상-클립-연결)
5. [레이어 설정](#5-레이어-설정)
6. [레이어 카메라 6대 설정](#6-레이어-카메라-6대-설정)
7. [RenderTexture 생성 및 연결](#7-rendertexture-생성-및-연결)
8. [Syphon Server 설정](#8-syphon-server-설정)
9. [맵핑 씬 설정](#9-맵핑-씬-설정)
10. [TouchDesigner 설정](#10-touchdesigner-설정)
11. [맵핑 작업](#11-맵핑-작업)
12. [Arduino 연결](#12-arduino-연결)
13. [전시 실행 체크리스트](#13-전시-실행-체크리스트)

---

## 1. 사전 준비

### 필수 설치

| 소프트웨어 | 버전 | 용도 |
|-----------|------|------|
| Unity | 2022.3 LTS | 메인 개발 |
| TouchDesigner | 최신 버전 | 프로젝션 맵핑 |
| KlakSyphon | Package Manager에서 설치 | Unity → Syphon 송출 |

### KlakSyphon 설치

1. Unity → `Window → Package Manager`
2. 왼쪽 상단 `+` → `Add package from git URL`
3. 아래 URL 입력:
   ```
   https://github.com/keijiro/KlakSyphon.git
   ```
4. Add 클릭 → 설치 완료

### 프로젝트 폴더 구조 준비

Unity 프로젝트 `Assets` 폴더 안에 아래 폴더를 미리 만들어둡니다.

```
Assets/
├── Scripts/
│   └── Editor/
├── Materials/
│   └── Panels/
├── RenderTextures/
├── Scenes/
└── Videos/
```

---

## 2. Unity 프로젝트 초기 설정

### 스크립트 추가

아래 스크립트 파일을 해당 경로에 복사합니다.

| 파일 | 경로 |
|------|------|
| ExhibitionManager.cs | Assets/Scripts/ |
| PanelPlayer.cs | Assets/Scripts/ |
| FlashlightBeam.cs | Assets/Scripts/ |
| ArduinoManager.cs | Assets/Scripts/ |
| MappingPreview.cs | Assets/Scripts/ |
| ProtoSceneBuilder.cs | Assets/Scripts/Editor/ |
| SyphonSenderSetup.cs | Assets/Scripts/Editor/ |

스크립트를 모두 추가하면 Unity가 컴파일합니다. 오류 없이 완료될 때까지 기다립니다.

---

## 3. 씬 자동 생성 (ProtoSceneBuilder)

ProtoSceneBuilder가 패널 14개, Flashlight 8개, Manager 오브젝트를 자동으로 생성합니다.

1. Unity 상단 메뉴 → `Playgroumd → Build Scene` 클릭
2. 완료 팝업 확인
3. Hierarchy에 아래 구조가 생성됩니다:

```
Hierarchy
├── Main Camera
├── Directional Light
├── Panels_1F_과보호
│   ├── Panel_1F_0_영아기
│   ├── Panel_1F_1_전환①
│   ├── Panel_1F_2_아동기
│   ├── Panel_1F_3_전환②
│   ├── Panel_1F_4_청소년기
│   ├── Panel_1F_5_전환③
│   ├── Panel_1F_6_성인기
│   ├── Flashlight_1F_0
│   ├── Flashlight_1F_2
│   ├── Flashlight_1F_4
│   └── Flashlight_1F_6
├── Panels_2F_자립
│   └── (동일 구조)
└── Managers
    ├── ExhibitionManager
    └── ArduinoManager
```

---

## 4. 영상 클립 연결

각 패널에 영상 클립을 연결합니다.

1. `Assets/Videos/` 폴더에 영상 파일(.mp4) 추가
2. Hierarchy에서 패널 선택 (예: `Panel_1F_0_영아기`)
3. Inspector → `PanelPlayer` 컴포넌트 → `Main Clip` 항목
4. Project 창에서 영상 파일을 `Main Clip` 칸으로 드래그

**인터랙션 구간 설정 (생애주기 패널만)**

| 항목 | 설명 | 기본값 |
|------|------|--------|
| Interaction Start | 인터랙션 시작 시간(초) | 19 |
| Loop At Interaction | 구간 루프 여부 | false |
| Interaction Loop End | 루프 끝 시간(초) | 24 |

레버 대기 중 영상을 반복하려면 `Loop At Interaction`을 체크하고 시작/끝 초를 영상에 맞게 조정합니다.

**영상 교체 방법 (기존 파일 덮어쓰기)**

파일명이 동일하면 Inspector 연결이 유지됩니다.

1. Hierarchy에서 패널 선택 → Inspector에서 기존 영상 우클릭 → `Show in Finder`
2. Finder에서 새 영상으로 파일 교체 (파일명 동일하게)
3. Unity로 돌아오면 자동 재임포트

---

## 5. 레이어 설정

### 레이어 생성

1. `Edit → Project Settings → Tags and Layers`
2. User Layer 빈 슬롯에 3개 추가:
   - `StoryPanels`
   - `TransitionPanels`
   - `Flashlights`

### 레이어 자동 지정

1. Unity 메뉴 → `Tools → Assign Layers (Story · Trans · Flash)`
2. 완료 팝업에서 숫자 확인:
   - StoryPanels: 8개
   - TransitionPanels: 6개
   - Flashlights: 8개

> **FlashlightVideoPanel이 있는 경우**: 별도로 `Flashlights` 레이어를 수동 지정해야 합니다.
> 해당 오브젝트 선택 → Inspector 오른쪽 위 `Layer` → `Flashlights`

---

## 6. 레이어 카메라 6대 설정

Hierarchy에서 빈 GameObject를 추가하고 Camera 컴포넌트를 붙여 6대를 만듭니다.

**카메라 이름 및 Culling Mask**

| 카메라 이름 | Culling Mask |
|-------------|--------------|
| Camera_Story_1F | StoryPanels |
| Camera_Trans_1F | TransitionPanels |
| Camera_Flash_1F | Flashlights |
| Camera_Story_2F | StoryPanels |
| Camera_Trans_2F | TransitionPanels |
| Camera_Flash_2F | Flashlights |

**각 카메라 Inspector 설정값**

```
Clear Flags       : Solid Color
Background        : 검정 (R:0 G:0 B:0 A:0)  ← Alpha 반드시 0
Culling Mask      : 해당 레이어만 단독 체크
                   (Nothing 선택 후 해당 레이어만 켜기)
Projection        : Orthographic
Size              : 270
Position          : Main Camera와 동일하게 (X:0 Y:0 Z:-10)
Target Texture    : (7단계에서 연결)
```

> **Alpha = 0 이 중요한 이유**
> TD에서 Over 합성 시 투명 영역이 아래 레이어를 가리지 않아야 합니다.
> Alpha = 1이면 검은 배경이 불투명하게 렌더링되어 합성이 깨집니다.

---

## 7. RenderTexture 생성 및 연결

각 카메라가 개별 RT에 렌더링해야 Syphon이 카메라별 출력을 독립적으로 잡습니다.

### RT 생성

1. Project 창 `Assets/RenderTextures/` 폴더 선택
2. 우클릭 → `Create → Render Texture`
3. 총 6개 생성:

| RT 이름 | 해상도 | 용도 |
|---------|--------|------|
| RT_Story_1F | 1920×1080 | 스토리 1면 |
| RT_Trans_1F | 1080×1620 | 전환 1면 |
| RT_Flash_1F | 1920×1080 | 라이트 1면 |
| RT_Story_2F | 1920×1080 | 스토리 2면 |
| RT_Trans_2F | 1080×1620 | 전환 2면 |
| RT_Flash_2F | 1920×1080 | 라이트 2면 |

RT 선택 후 Inspector에서 Width/Height를 위 값으로 설정합니다.

### 카메라에 RT 연결

각 카메라 선택 → Inspector → `Target Texture` → 해당 RT 드래그

### 전환 패널 Scale + RT 자동 변경

`Tools → Resize Transition Panels (2:3)` 실행 시 전환 패널 6개의 Quad Scale과 RT 해상도를 한 번에 변경합니다.

---

## 8. Syphon Server 설정

카메라 6대 각각에 SyphonServer 컴포넌트를 추가합니다.

**Camera_Story_1F 기준 (나머지도 동일)**

1. Camera_Story_1F 선택
2. Inspector → `Add Component → Syphon Server`
3. Source → `Texture`
4. Source Texture → `RT_Story_1F` 연결
5. Server Name → `Story_1F` 입력

| 카메라 | Source Texture | Server Name |
|--------|---------------|-------------|
| Camera_Story_1F | RT_Story_1F | Story_1F |
| Camera_Trans_1F | RT_Trans_1F | Trans_1F |
| Camera_Flash_1F | RT_Flash_1F | Flash_1F |
| Camera_Story_2F | RT_Story_2F | Story_2F |
| Camera_Trans_2F | RT_Trans_2F | Trans_2F |
| Camera_Flash_2F | RT_Flash_2F | Flash_2F |

---

## 9. 맵핑 씬 설정

본 전시 씬과 별도로 맵핑 작업 전용 씬을 구성합니다.

1. `File → Save As` → `MappingScene`으로 저장
2. `Managers` 오브젝트 비활성화 (Inspector 체크 해제)
3. Hierarchy에서 빈 GameObject 생성 → 이름 `MappingPreview`
4. `MappingPreview` 컴포넌트 추가

Play 누르면 0.5초 후 모든 패널의 첫 프레임이 표시되고 사라지지 않습니다. 이 상태에서 TD 맵핑 작업을 진행합니다.

---

## 10. TouchDesigner 설정

### Syphon 스트림 수신

1. Unity를 **Play 모드**로 전환
2. TD에서 빈 공간 더블클릭 → TOP → `Syphon Spout In` 6개 생성
3. 각 노드 이름과 Sender Name 설정:

| 노드 이름 | Sender Name |
|-----------|-------------|
| Story_1F | Unity:Story_1F |
| Trans_1F | Unity:Trans_1F |
| Flash_1F | Unity:Flash_1F |
| Story_2F | Unity:Story_2F |
| Trans_2F | Unity:Trans_2F |
| Flash_2F | Unity:Flash_2F |

### Composite 설정

1면과 2면 각각 Composite TOP 생성:

1. 빈 공간 더블클릭 → TOP → `Composite`
2. Operation → `Over`
3. 입력 연결 (드래그):
   - Input 0: Story_1F (또는 Story_2F)
   - Input 1: Trans_1F (또는 Trans_2F)
   - Input 2: Flash_1F (또는 Flash_2F)

### Kantan Mapper 연결

1. Palette에서 `kantanMapper` COMP 추가 × 2
2. 각 kantanMapper 선택 → `Open Kantan Window`
3. Kantan Window 내 우클릭 → `Add Shape → Rectangle`
4. Rectangle → Texture → `comp1` (또는 `comp2`) 경로 입력

---

## 11. 맵핑 작업

프로젝터를 연결하고 물리 패널에 영상을 맞추는 작업입니다.

**준비**
- 프로젝터 2대 연결 및 전원 ON
- Unity MappingScene Play 모드 실행
- TD에서 Syphon 스트림 수신 확인 (노란 경고 삼각형 사라짐)

**맵핑 조정**

1. Kantan Window에서 Rectangle 선택
2. 4개 코너 핸들을 드래그해 실제 물리 스크린 경계에 맞춤
3. 패널 경계가 정확히 맞으면 `Save Project` 클릭

**출력 설정**

1. TD에서 kantanMapper 출력 → `Window COMP` 연결
2. Window COMP → Monitor → 해당 프로젝터 디스플레이 선택
3. `Open` 클릭 → 전체화면 투사 시작

---

## 12. Arduino 연결

레버 인터랙션 감지를 위한 Arduino 설정입니다.

1. Hierarchy → `Managers` → `ArduinoManager` 선택
2. Inspector:
   - `Use Serial` 체크
   - `Port Name` → Arduino가 연결된 포트 입력 (예: `/dev/cu.usbmodem14101`)
   - `Baud Rate` → `9600`
3. Play 모드에서 레버 회전 시 Console에 감지 로그 확인

> 테스트 시 `Use Serial` 해제 + Spacebar로 레버 감지 시뮬레이션 가능

---

## 13. 전시 실행 체크리스트

전시 당일 실행 전 아래 항목을 순서대로 확인합니다.

**하드웨어**
- [ ] 프로젝터 2대 전원 ON 및 연결 확인
- [ ] Arduino 레버 USB 연결 확인
- [ ] 컴퓨터 절전 모드 OFF

**Unity**
- [ ] `SampleScene` 열기 (MappingScene 아님)
- [ ] `Managers` 오브젝트 활성화 확인
- [ ] 각 패널 `Main Clip` 연결 확인
- [ ] ArduinoManager `Use Serial` 체크 + 포트 확인
- [ ] Play 버튼 클릭

**TouchDesigner**
- [ ] Unity Play 후 Syphon 스트림 수신 확인 (6개 노드 정상)
- [ ] Kantan Mapper 저장된 맵핑 로드 확인 (`Load Project`)
- [ ] 프로젝터 출력 창 열기 (`Open`)
- [ ] 투사 위치 최종 확인

**이상 시 빠른 대처**
- Syphon 스트림 안 보임 → Unity Play 모드 재시작
- 맵핑 어긋남 → Kantan Window에서 코너 재조정 후 Save
- 패널 영상 안 나옴 → 해당 패널 Inspector에서 Main Clip 재확인
