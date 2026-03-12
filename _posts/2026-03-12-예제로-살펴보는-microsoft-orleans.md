---
title: "Microsoft Orleans 살펴보기"
date: 2026-03-12 15:18:44 +0900
description: ""
categories: [Orleans] #[upper, lower]
tags: [C#] #must be lower
math: true
mermaid: true
---

## [Microsoft Orleans](https://learn.microsoft.com/en-us/dotnet/orleans/overview?pivots=orleans-10-0)
공식 문서에서는 다음과 같이 설명한다.

> * 간단하게 분산앱을 만들기 위해서 고안된 크로스 플랫폼 프레임 워크입니다. 
> * 싱글 서버 부터 수천개의 클라우드 기반 앱까지 규모에 상관 없이 분산스시템의 복잡성을 도와주는 도구를 제공해 줍니다. 
> * 기존의 C#의 개념을 멀티 서버 환경에 확장시켜 개발자가 앱의 로직에 집중할 수 있도록 해줍니다.

<br>
이 설명 만으로는 잘 와닿지 않는다. 조금 더 자세한 설명을 살펴보자

<br>

>* It's designed to scale elastically. Add or remove servers, and Orleans adjusts accordingly to maintain fault tolerance and scalability.
>* It simplifies distributed app development with a common set of patterns and APIs, making it accessible even for those new to distributed systems.
>* It's cloud-native and runs on platforms where .NET is supported—Linux, Windows, macOS, and more.
>* It supports modern deployment options like Kubernetes, Azure App Service, and Azure Container Apps.

요약하면 탄력적인 스케일링, 간소화된 분산앱 개발, .NET이 지원 되는 환경에서 실행되는 클라우드 기반 환경, 최선 배포 옵션 지원이다. 

기존 웹서버를 생각해보면 클라이언트가 요청을 보내면 알맞은 서버를 선택해 로직을 수행하고 이 과정에서 필요한 데이터는 DB에서 가져온다.
Orleans는 이런 과정을 간소화 하고 여러 도구들을 제공한다는데 어떤 것인지 알아보자. 

## Actor model
기존의 객체 처럼 상태, 상태와 관련한 행위를 캡슐화한 것이지만, 여기에 경량, 불변성, 동시성이 추가된 객체이다.
Orleans에서는 이러한 액터가 영구적으로 존재하는 '가상 액터 추상화'를 발명했다고 한다.

## Grain
Grain은 Orleans의 구성 단위로 위에서 말한 가상 액터에 해당되며 Identity, Behavior, State로 구성된다.
![](https://learn.microsoft.com/en-us/dotnet/orleans/media/grain-formulation.svg)

grain에서의 identity는 '사용자 정의 키'로 grain을 항상 호출할 수 있도록 만들어 준다. 다음의 인터페이스 중 하나를 상속 받아 구현 (gui, 정수, 복합키)
>* IGrainWithGuidKey: Marker interface for grains with Guid keys.
>* IGrainWithIntegerKey: Marker interface for grains with Int64 keys.
>* IGrainWithStringKey: Marker interface for grains with string keys.
>* IGrainWithGuidCompoundKey: Marker interface for grains with compound keys.
>* IGrainWithIntegerCompoundKey: Marker interface for grains with compound keys.

<br>

```cshap
GetGrain<IUserGrain>(userId)
```
이런 식으로 키와 그레인의 타입을 통해서 원하는 그레인을 호출 가능하다. 여기까지만 살펴보고 그래서 이게 뭔데 라는 생각을 했지만 다음 그림을 살펴보고 어느정도 이해가 되었다.

<br>

![](https://learn.microsoft.com/en-us/dotnet/orleans/media/grain-lifecycle.svg)

Orleans 런타임은 요청이 들어오면 grain을 자동으로 인스턴스화해서 메모리에 올린다. grain은 state를 들고 있을 수 있기 때문에 매 요청마다 DB에 접근할 필요 없이 메모리에 있는 grain에 접근해서 빠르게 원하는 데이터를 받아 올 수 있다.

한동안 사용되지 않은 grain은 메모리에서 내려간다. 그렇다면 메모리에 올라가지 않은 grain을 호출하면 어떻게 될까 

클라이언트의 입장에서는 해당 그레인의  아이덴티티(타입과 키)만 알면 되고 그 이외의 요소는 알 필요가 없다. 가상 액터 모델에서 말한대로 그레인이 있다고 생각하고 요청을 보낸다. 실제 해당 그레인이 없다면 런타임이 만들어서 메모리에 올릴거고, 메모리에 있으면 해당 그래인의 참조를 건네줄 것이다.

## Silo
Silo는 Orleans의 또 다른 구성 단위로 하나 이상의 grain을 호스팅한다.

![](https://learn.microsoft.com/en-us/dotnet/orleans/media/cluster-silo-grain-relationship.svg)

여러 사일로들은 클러스터로 동작하며 사일로들은 클러스터 내에서 분산작업, 오류 감지 및 복구에 협조한다.

런타임이 클러스터에 호스팅된 grain들을 마치 하나의 프로세스에 있는것처럼 상호작용하게 해준다.

타이머, 리마인더, 지속성, 트랜잭션 stream 등의 기능을 추가로 제공하기도 한다.

## 정리
Orleans는 기존의 stateless 방식과는 다르게 서버 메모리에 grain을 올리고 이를 통해서 분산앱을 구현한다.
원하는 정보를 grain이라는 객체로 들고 있고 클라이언트에서도 grain을 로컬 객체처럼 사용하는게 장점인 것 같다.
런타임에서 기본 적인 분산처리, 생명주기 관리를 해주고 커스텀 설정도 가능해 보인다.
C#을 사용하는 클라이언트에서 사용하면 효과적일 것 같다.