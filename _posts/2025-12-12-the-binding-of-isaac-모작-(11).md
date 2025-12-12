---
title: "The binding of Isaac 모작 (11)"
date: 2025-12-12 18:57:05 +0900
description: The binding of Isaac의 모작의 버그를 수정해보자 ㅜㅜ
categories: [Portfolio, GunsGermsAndSteel] #[upper, lower]
tags: [unity, c#, addressable] #must be lower case
math: true
mermaid: true
---

## 1. 메모리 누수? 버그? 올바르지 않은 형태의 증가
> ![](https://velog.velcdn.com/images/garage_keeper/post/0f3833b6-6bed-4d5e-856f-d885a823924c/image.png)
><div style="text-align: center; font-style: italic; text-decoration: line-through;  color: gray;">
     WTF
     </div>

위의 사진에 보이듯 엄청난 메모리 할당량이다. 
더욱 충격적인 사실은 아래의 사진을 통해서 확인하자

>![](https://velog.velcdn.com/images/garage_keeper/post/e5519106-7c0a-40ae-acf9-bd2ea9e2fcf2/image.png)

그렇다... 단 40여초 사이에 1.7GB가 증가했다....


## 2. 원인찾기
빌드된 파일에서 이 현상을 처음 발견했다. 게임 자체가 버벅여 작업표시줄을 확인했더니 2D게임에서 메모리를 6GB를 차지하는 놀라운 장면을 목격하고 말았다....
하지만 에디터에서 실행하면 정상적인 메모리 점유를 했다.
<br>

### 1. 프로파일러 살펴보기
![](https://velog.velcdn.com/images/garage_keeper/post/aafbf35e-0ecb-49aa-a365-2496c6da0330/image.png)
<div style="text-align: center; font-style: italic; text-decoration: line-through; color: gray;">
     Unknown은 뭐 어쩌라는 건지....
     </div>
내게는 별 도움이 되지 못했다..
<br>

### 2. 원인이 되는 코드 범위 좁히기
#if UNITY_EDITOR의 영향을 받던 코드들 중 GenerateRoom 함수의 마지막 부분 
```AltSetActive(r.RoomObject, r.RoomType == ERoomType.Start);``` 부분을 주석 처리하니 동일한 현상이 나타났다.

<br>

### 3. 원인 코드
MapManager.cs
```csharp
public void AltSetActive(GameObject room, bool state)
{
   FindChildByName(room.transform, "Obstacle")?.gameObject.SetActive(state);
   FindChildByName(room.transform, "Collider")?.gameObject.SetActive(state);
   FindChildByName(room.transform, "ProjectileCollider")?.gameObject.SetActive(state);
   foreach (Transform child in FindChildByName(room.transform, "Doors"))
   {
       foreach (Transform sprites in child)
           sprites.gameObject.SetActive(state);
   }
   FindChildByName(room.transform, "Pickups")?.gameObject?.SetActive(state);
   FindChildByName(room.transform, "Monster")?.gameObject.SetActive(state);
   FindChildByName(room.transform, "ShopItems")?.gameObject.SetActive(state);
}
```

### 4. 원인 분석
AltSetActive 함수는 내 나름대로의 최적화를 위해서 SetActive대신에 사용한 함수이다.
stage의 형태를 보면서 디버깅을 해야해서 방의 타일맵만 남겨두고 Obstacle, Pickup, Door등을 state 값을 통해서 켜고 끈다.
**AltSetActive(r.RoomObject, r.RoomType == ERoomType.Start);** 가 실행되지 않는 것이 메모리 증가의 원인이다.
이 코드가 실행되지 않으면 모든 방의 모든 하위 오브젝트가 켜진 상태가 된다.

하위 오브젝트를 하나씩 비활성화 해보다 원인을 찾았다.
**Collider 오브젝트와 ProjectileCollider 오브젝트가 둘 다 켜져있을 때** 메모리가 증가한다.
한 스테이지의 방이 여러개라 이런 증가가 배가 되었다.

이게 왜...? 라는 질문에 열심히 구글링을 하던중 [이것](https://www.reddit.com/r/Unity2D/comments/1013gli/my_game_is_lagging_in_my_beefy_computer_ive/?utm_source=chatgpt.com)을 읽게 되었다. 
> **'타일맵 콜라이더 2D를 구성하는 타일이 너무 많다. 겹치는 콜라이더가 많으면 문제가 생길 수 있다. '**

일반적인 충돌과 투사체의 충돌 범위가 '조금' 달라야 해서 별도의 콜라이더 사용, 겹치는 부분 많음 이 두가지가 원인이 되었던 것 같다. 
(물리 매트릭스에서 이미 충돌 체크를 해제했는데 발생하는걸 보니, 내부에서 경계 정보등을 갱신하면서 메모리가 늘어나는 듯)


### 5. 해결
[링크](https://www.reddit.com/r/Unity2D/comments/1013gli/my_game_is_lagging_in_my_beefy_computer_ive/?utm_source=chatgpt.com)에서는 Composite Collider 2d를 사용하는 것도 좋은 방법이라고 한다.
나는 그냥 Box Collider 2D들의 조합으로 해결했다.


### 6. 발견하지 못한 이유
[링크](https://www.reddit.com/r/Unity2D/comments/1013gli/my_game_is_lagging_in_my_beefy_computer_ive/?utm_source=chatgpt.com)의 이슈는 사실 이전에 겪어본 적이 있었다. 그 때는 테스트를 하는데 많은 장애물, 몬스터, 픽업들을 사용했다. 그냥 오브젝트가 많고, 에디터의 인스펙터가 업데이트하는 정보가 많아서 그런 줄 알았다. 그래서 **AltSetActive()**를 사용하게 되면서 잊혀졌다가 지금에서야 발견되었다. 

<br>

## 2. 사운드의 위상이 겹치는 문제
거창하게 소제목을 적었지만, 그냥 동시에 같은 소리를 재생하면 소리가 증폭이 되는 문제다.
Unity에서 제공하는 Mixer의 Compressor를 이용해 해결 하였다.