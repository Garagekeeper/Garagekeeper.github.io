---
title: "Unreal Network - 2"
date: 2025-12-13 00:38:35 +0900
description: ""
categories: [Computer Science, Unreal Network] #[upper, lower]
tags: [unreal, cs, cpp, server] #must be lower
math: true
mermaid: true
---
# 1. Replication
복제, 사본등의 의미를 가지고 있다.
권한을 가진 서버가 연결된 클라이언트들에게 게임의 상태를 전달하는 것 서버의 상태를 복사한다고 볼 수 있다.
# 2. Actor Replication

(일반적으로 말하는 Replication)
### 개념
* 서버의 액터를 클라이언트로 복제
* 원본의 클래스를 토대로 복제품을 만드는 것
  * 복제품의 변수등은 클래스를 만들 때 초기상태
  * 변수 컴포넌트도 복제하고 싶으면 설정 해야
* **서버 -> 클라 방향으로만 진행됨**

### 몇몇 예시
* Replicated Properties
  * ``` UPROPERTY( replicated ) ```
  * 기본적으로 Replication 사용하게 해줌
  * 조건 제한들을 설정 할 수 있음
* Replicated Using Properties
  ```cpp
   UPROPERTY(ReplicatedUsing=OnRep_HealthUpdate)
  ```
 ```cpp 
 void ADerivedActor::OnRep_HealthUpdate()
 {
         UE_LOG(LogTemp, Log, TEXT("OnRep_HealthUpdate"))
 
         // Add custom OnRep logic
 }
  ```
  * 복제가 발생하면 RepNotify를 실행
  * RepNotify 함수를 등록하자
  
* RPC
  * 액터가 다른 머신의 함수를 실행하게 해줌.
  
### Replication의 초기 단계
* 클라이언트가 어떤 액터를 어떤 연결에 복제할지 확인해준다
* 서버는 프로퍼티 업데이트, RPC의 실행 순서를 결정
* 서버가 관련된 정보를 클라에게 전송


### replication feature
* Creation and Destruction
  * 서버에 복제가능한 액터의 authoritative version이 생기면 연결된 모든 클라이언트에 복제본(proxy)를 생성 
  * 서버에서 파괴되면 원격 프록시들도 자동 파괴
* Movement
  * 권한이 있는 액터의 ```Replicate Movement``` 나 ```bReplicateMovement == true``` 이면 위치, 회전, 속도를 자동으로 복제
* Properties
  * 설정된 프로퍼티들은 값이 바뀔 때 자동으로 복제
* Components
	* 액터의 일부분으로 복제
	* 복제 되도록 설계된 컴포넌트는
    * 여기서 호출된 RPC는 일관되게 작동
* Subobjects
  * UObject를 액터에 붙여서 서브오브젝트로 복제 할 수 있다.
  * 디테일한 부분은 문서 참조
* Remote Procedure Calls
  * 원격으로 함수를 호출 하는 느낌
  * 서버만, 클라이언트만, 전부다 처럼 타겟을 정할 수 있음

###  not replicate
이런 것들은 클라이언트마다 다르게 보여야 할 수 있다.
* Skeletal Mesh Component
* Static Mesh Component
* Materials
* Animation Blueprints
* Particle System Component
* Sound Emitters
* Physics Objects

# 3. RPC
### 개념
* Remote Procedure Call
* 함수의 호출은 local에서 실행은 remote에서
* 일시적인 이벤트들에 사용
  * 소리
  * 파티클 소환
  * 애니메이션 실행
* Replicated 나 ReplicatedUsing를 사용하는 복제된 프로퍼티를 보충해줌

### RPC의 타입
* Client
  * Run On Owning Client
* Server
  * on server
* Remote
  * Client + Server - local
* NetMulticast
  * Server + Client


### RPC 생성
```cpp
 UFUNCTION(Client)
      void ClientRPC();

```
* UFUNCTION()에 RPC의 타입을 넣어서생성
* AActor의 생성자에서 replicate 하도록
* RPC함수 구현


### RPC 실행
```cpp
// Call from client to run on server
	ADerivedClientActor* MyDerivedClientActor;
	MyDerivedClientActor->ServerRPC();

	// Call from server to run on client
	ADerivedServerActor* MyDerivedServerActor;
	MyDerivedServerActor->ClientRPC();

	// Call from server to run on server and all relevant clients
	ADerviedServerActor* MyDerivedServerActor;
	MyDerievedServerActor->MulticastRPC();
```
* RPC는 내가 소유한 (제어하는) 액터를 대상으로만 호출 할 수 있다.
  * 동적으로 소환된 대상의 소유권은 서버인데....
  * ex) 맵에 총이 소환되고 플레이어들은 이걸 주워서 쓴다고 하면...?
    * 소유권에 관한 내용으로, 다음 게시물에서 다룸.
 
### RPC의 신뢰성
* 기본적으로 UDP라 신뢰성 보장 X
* Reliable 사용을 통해서 신뢰성 보장