---
title: "Unreal Network - 1"
date: 2025-12-13 00:36:57 +0900
description: 
categories: [Computer Science, Unreal Network] #[upper, lower]
tags: [unreal, cs, cpp, server] #must be lower
math: true
mermaid: true
---

언리얼 엔진에서는 자체 네트워킹 프레임 워크를 지원한다.
소규모 게임 혹은 FPS등에 특화되어 빠르게 게임을 만들 수 있다고 한다.

# 1.Unreal Engine Networking Architecture
언리얼 엔진은 네트워크에 연결된 멀티플레이어 게임에 클라이언트-서버 구조를 사용한다.

## 네트워크에 연결된 멀티플레이어 게임
>![](https://velog.velcdn.com/images/garage_keeper/post/8b4fe5ac-c16e-4f7b-bc96-9bb75e66d81e/image.png)
> 서버는 게임 플레이(게임 로직, 판정 등)을 진행하고 클라이언트는 게임을 랜더링한다. 

* 별도의 머신에 있는 플레이어(클라이언트)들이 네트워크를 통해서 중앙 머신에 연결됨.
* 중앙 머신은 클라이언트가 연결된 멀티플레이 게임을 관리한다.
* 싱글 혹은 로컬 멀티플레이 게임과는 다르게 여러 문제가 발생
  * 불안정한 네트워크
  * 서로 다른 네트워크 속도
  * => 서로 다른 게임 상태 유발

 
## 클라이언트-서버 모델
> ![](https://velog.velcdn.com/images/garage_keeper/post/eaabfc59-5281-4765-b876-1f847c9e3816/image.png)

* 위의 문제를 해결하기 위해 서버가 모든 권한을 가짐
  * 서버가 게임의 유일한 상태를 가짐
  * 서버의 상태를 각 클라이언트에 복제해서 보여줌
 * 실제 게임은 서버에서 실행
   * 클라이언트는 서버에서 자신이 소유한 remote pawn을 제어
   * local pawn에서 remote procedure calls-> server pawn -> 실제 액션 -> 서버에서 게임 상태 replicates -> 클라이언트(액터가 있는)에게 전달 -> 이 정보를 기반으로 서버의 상태를 근사 시뮬레이션
   * 어떤 정보를 복제하고 이를 누구에게 보낼지 선택하는게 중요

# 2. 기초적인 네트워킹 개념
## 네트워크 모드
* Standalone
  * 네트워킹이 없는 모드
* Dedicated Server
  * local player가 없는 서버
  * 입력 랜더링 필요 없이 판정만
  * 보안, 정확성, 대규모 서버에 필요
* Listen Server
  * local player가 있는 서버
    * local player : 서버를 호스팅하는 사람
  * 경쟁 없는캐쥬얼 협동게임
* Client
  * 원격 서버에 연결된 클라이언트
  * 서버 코드를 실행하지 않음
  
 
게임 인스턴스의 네트워크 모드를 알아내려면 GetNetMode 사용