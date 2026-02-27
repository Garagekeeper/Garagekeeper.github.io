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

위 사진에서 주목한 요소는 Update.SriptRunBehaviourUpdate이다. 코드에서 실행되는 Update들이 모여있는 항목으로 각 Update들의 사용량을 볼 수 있다. 이를 통해서 FirearmController.Update()에서 시작한 콜스택이 전체 사용량의 25%에 육박한다. 콜스택을 따라가보면 사격처리에 관한 로직임을 알 수 있고, ShowDamageText 함수가 7.8%임을 보인다. 해당 함수는 다음과 같다.

```cs
public void ShowDamageText(DamageInfo info)
{
    var go = HeadManager.Resource.Instantiate(EObjectID.DMGFX, info.hitPoint,HeadManager.ObjManager.DmgRoot);
    go.GetComponent<DamageText>().Init(info);
}
```

