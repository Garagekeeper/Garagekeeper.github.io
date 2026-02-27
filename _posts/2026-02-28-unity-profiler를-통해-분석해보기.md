---
title: "[Unity] Profiler를 통해 분석해보기"
date: 2026-02-28 01:30:22 +0900
description: 
categories: [] #[upper, lower]
tags: [] #must be lower
math: true
mermaid: true
---

# Profiler
[프로파일러]()는 게임의 성능에 관한 정보를 보여주는 툴이다. 이를 통해서 CPU, GPU, 랜더링, 메모리등의 수치를 확인해 볼 수 있다.

에디터의 플레이모드에서 프로파일러를 실행하면 에디터 루프가 추가되어 실제 게임만의 데이터를 보기 쉽지 않으니 Development Build를 통해서 프로파일러에 연결해 사용하자 (Script Debugging을 사용하면 IDE에서 디버깅을 할 수 있다)

<br>

![ProfilerExample](/assets/img/ProfilerExample.png)
<p align="center">
  <sub>빌드된 게임의 프로파일링을 하는 장면</sub>
</p>
<br>

![ProfilerExample](/assets/img/ProfilerExample2.png)
<p align="center">
  <sub>프로파일러 화면, 어떤 요소가 CPU를 많이 점유하는지 확인할 수 있다</sub>
</p>

<br>

# Profiler 분석하기

## 살펴보기
빌드된 게임을 플레이하면서 원하는 지표를 살펴본다. 나의 경우는 프레임이 떨어지는 구간이 있었기에 CPU 사용량을 살펴보며 진행했다.

그러다가 지표가 갑자기 튀는 구간이 있으면 그곳을 확인해본다.

![ProfilerExample](/assets/img/BeforeOpt.png)

위 사진에서 주목한 요소는 Update.SriptRunBehaviourUpdate이다. 코드에서 실행되는 Update들이 모여있는 항목으로 각 Update들의 사용량을 볼 수 있다. 이를 통해서 FirearmController.Update()에서 시작한 콜스택을 따라가보면 사격처리에 관한 로직임을 알 수 있고, ShowDamageText 함수가 7.8%(3.23ms)임을 보인다. 데미지를 표시하는 오브젝트는 풀링을 사용하고 있었기 때문에 이상했다.

계속 추적한 결과 풀매니저에서 풀을 새로 만들고 오브젝트를 Instantiate하는 과정이 문제인 것 같았다. 오브젝트의 잦은 Instantiate를 피하기위한 풀링인데 잘못 사용하고 있었다. (정확히 말하면 처음 적을 맞힌 상황에서 처음으로 풀을 생성해서 발생한 문제다. 풀의 오브젝트가 모자라서 새로 생성한 것과는 다른 문제이다.)

<br>

## 분석하기
기존의 흐름은 다음과 같다
* 리소스 매니저에 오브젝트 생성요청
* 풀 매니저에 오브젝트 요청
* 해당 오브젝트의 풀이 있는지 확인
  * 없으면 새로운 풀 생산
  * 있으면 풀에서 하나 가져옴
* 풀 매니저가 리소스 매니저에 오브젝트 반환
* 리소스 매니저가 오브젝트 반환

여기서 '없으면 새로운 풀 생산' 부분 때문에 Object.Instantiate가 실행되어 사용량이 많았다고 추측되었다. 기존에 사용했던 풀에서 새로운 풀 방식으로 넘어오면서 사용할 풀을 미리 생성하는 과정이 누락되어 발생한 일이었다.

<br>

## 수정하기
게임씬으로 넘어올 때 사용할 풀을 모두 생성하는 코드를 추가했다.


```cs
public override void Init()
{
    base.Init();
    HeadManager.Game.IsPlayerDead = false;
    PreWarmPool();
    HeadManager.UI.ShowSceneUI<UIGameScene>();
}

// 사용할 풀을 미리 생성하는 함수
private void PreWarmPool()
{
    // Scriptable Object의 정보를 담은 Dict
    var soDict = HeadManager.Resource.SourceCatalog.SoDict;
    Transform rootTransform = null;
    
    // 풀링 오브젝트별로 root 지정
    foreach (var soPair in soDict)
    {
        switch (soPair.Key)
        {
            case Defines.EObjectID.Enemy:
                rootTransform = HeadManager.ObjManager.EnemiesRoot;
                break;
            case Defines.EObjectID.ExpGemNormal:
            case Defines.EObjectID.ExpGemRare:
            case Defines.EObjectID.ExpGemEpic:
                rootTransform = HeadManager.ObjManager.ExpGemsRoot;
                break;
            case Defines.EObjectID.SFX:
                rootTransform = HeadManager.ObjManager.SoundRoot;
                break;
            case Defines.EObjectID.DMGFX:
                rootTransform = HeadManager.ObjManager.DmgRoot;
                break;
            default:
                break;
        }
        
        HeadManager.Pool.CreatePoolExternal(soPair.Value,  rootTransform);
    }
}
```

<br>

## 결과 확인하기

완벽히 똑같은 환경은 아니지만 최대한 비슷한 환경(처음으로 적을 맞힌 상황)에서 전과 후를 비교해보자

![ProfilerExample](/assets/img/BeforeOpt.png)
![ProfilerExample](/assets/img/AfterOpt.png)


이전의 ShowDamageText()는 3.23ms, 이후의 ShowDamageText()는 0.79ms로 의도한 방식으로 풀링이 동작하는것을 확인할 수 있었다.

코드를 작성하면서 발견하지 못한 실수나 구조의 결함등을 찾을 수 있는 방법중에 하나인 것 같다. 제대로 활용하는 법을 배우면 유용할 것 같다.

