---
title: "Unreal Network - 3"
date: 2025-12-13 00:39:13 +0900
description: 
categories: [Computer Science, Unreal Network] #[upper, lower]
tags: [unreal, cs, cpp, server] #must be lower
math: true
mermaid: true
---

## 1. 소유권
* RPC는 자신이 제어하는 대상으로만 유효
  * 특히 서버에서 실행되는 RPC에서 중요
  * ex) 클라에서 serverRPC 호출
    * 같은 액터가 3개 있고 하나만 내가 조종 상황
      * is server false 체크 
      * serverRPC 호출
    * 총 6번 실행 되어야 할 것 같지만 2번 실행
    * 각 클라이언트에서 내가 소유한 액터만 실행
      * IsLocallyControlled 체크랑 동일한 상황
* 동적으로 생성된 액터들의 소유권
  * 기본적으로 서버가 가짐
  * 서버가 소유권 주는것의 주체
  * SetOwner 또는 액터를 생성할 때 설정 가능
    * 폰, 플레이어 컨트롤러 설정

## 2. 액터 채널
* 클라와 서버는 연결되면  UNetConnection 생성
  * 여기에서 RPC, Replication System을 위한 ActorChannel 생성
  * 조금 더 들어가서 Relevancy가 없으면 채널 삭제
* Relevancy
   * 직역하면 관련성
   * 위치 등 여러 요소들을 종합해서 판단
     * 자세한 내용은 [여기](https://dev.epicgames.com/documentation/en-us/unreal-engine/actor-relevancy-in-unreal-engine)
   * 관련성이 있어야 채널이 생기고 복제가 됨
   * 관련성 없으면 
     * ex) 너무 멀리 있으면 
     * 보이지도 않는데 그냥 복제 하지 말자
   * 이걸 잘 설정하지 않으면 위험
     * Relevancy를 왔다갔다 하는 패킷을 보내서 서버를 셧다운 시킬 수도 있다고...

## 3. 자신이 어떤 머신인지 확인 하는 법
* enum ENetMode
  * NM_StandardAlone
  * NM_DedicatedServer
  * NM_ListenServer 
  * NM_Client
* HasAuthority()
  * 간혹가다 예외 있음
  * 클라이언트에서만 소환 하는 액터들에서 예외
* NetRole
  * Authority (서버)
  * proxy (클라이언트)
    * Autonomous (내가 조종하는 액터)
    * Simulated  (시뮬레이션 되는 액터)
  * GetLocalRole() : 현재 머신에서 이 액터의 Role
  * GetRemoteRole() : 나의 반대에서 이 액터의 Role
