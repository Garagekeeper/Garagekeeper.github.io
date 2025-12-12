---
title: "CPU Cache and CPU Pipeline"
date: 2025-12-12 23:10:35 +0900
description: "CPU에 대해서 알아보자"
categories: [Computer Science, Computer Architecture] #[upper, lower]
tags: [cs] #must be lower
math: true
mermaid: true
---

# CPU Cache
## 개념
> CPU가 Main Memory의 data를 접근하는데 드는 비용(시간, 에너지)를 줄이기 위해서 사용되는 hardware cache 이다.[(출처)](https://en.wikipedia.org/wiki/CPU_cache)
>![](https://velog.velcdn.com/images/garage_keeper/post/f301dd37-9158-42c7-9dcf-936d861303e3/image.png)

* Main Memory보다 속도 빠른 SRAM 사용
* L1, L2, L3 의 계층을 가짐 숫자가 작을수록 core에 가까움
* cache hit
  * 캐시에 원하는 데이터가 있는 상태
* cache miss
  * 캐시에 원하는 데이터가 없는 상태
* 평군 기억장치 액세스 시간 ($T_a$)
  * $T_a = H \times T_c + (1-H) \times T_m$
  * $T_c, T_m$ 는 각각 캐시 액세스 시간, 메인메모리 액세스 시간.
* 캐시의 구조
  * 블록을 담는 라인(슬롯)들로 구성
  * 각 라인은 태그를 통해서 저장된 블록을 구분
  
## 지역성 (locality)
* CPU가 짧은 시간 동안 동일한 위치의 메모리를 참조하는 경향
* 시간적 (temporal locality)
  * 최근에 사용한 데이터를 다시 사용할 가능성이 높다.
* 공간적 (spatial locality)
  * 인접해서 저장된 데이터들이 연속적으로 액세스 될 가능성이 높다. (vector, 배열)
  
## 사상 (Mapping)
* Direct mapping
  * 메모리를 블록들로 나누고 각 블록을 정해진 위치에 매핑
* Fully associative mapping
  * 빈 공간 아무곳에나 매핑
* Set associate mapping
  * 위 2가지를 합친 방식
  * 구역을 정해놓고 그 구역의 빈 공간 아무곳에나
  배치
  
## 교체 알고리즘
* LRU
  * 사용되지 않은 채로 가장 오래된 블록을 교체
* FIFO
  * 먼저 들어온 걸 교체
* LFU
  * 가장 적게 사용된 걸 교체
  
  
## 쓰기 정책
* Write through
  * 캐시와 메인 메모리를 동시에 수정
  * 비교적 느림
* Write back
  * 캐시만 수정
  * 비교적 빠름
  * 메인 메모리의 값과 달라서 교체될 때 갱신해야함. (상태 비트)
  
## 캐시 일관성 프로토콜
>![](https://velog.velcdn.com/images/garage_keeper/post/7e2189ef-8e36-413e-ae46-43b6a86f2786/image.png)

위 그림처럼 각 캐시와 메인 메모리간의 데이터 불일치가 발생
이를 해결하기 위해서 캐시 일관성 프로토콜 사용

* [MESI 프로토콜](https://en.wikipedia.org/wiki/MESI_protocol)(스누피 프로토콜)
  * 메모리를 4가지 상태로 표현
  * Modified(수정) 상태 : 데이터가 수정된 상태
  * Exclusive(배타) 상태 : 유일한 복사본이며, 주기억장치의 내용과 동일한 상태
  * Shared(공유) 상태 : 데이터가 두 개 이상의 프로세서 캐시에 적재되어 있는 상태
  * Invalid(무효) 상태 : 데이터가 다른 프로세스에 의해 수정되어 무효화된 상태
  
* 스누피 프로토콜
  * 버스 스누핑
    * 코어가 버스를 통해서 메모리 접근 감시
  * 그에따라 snoopy protocol(MSI, MESI, MOESI 등) 사용
* 디렉토리 프로토콜
  * 어떤 캐시가 어떤 블록을 가지고 있는지 기록
  * 메모에도 기록되어서 이를 기반으로 관리

# CPU Pipleline(명령어 Pipeline)
명령어를 싱글 프로세서에서 명령어 수준 병렬을 구현하는 기법이다.
* 명령어 수준 병렬
  * 동시에 여려 명령어를 처리 하는 것
* 명령어를 여러 단계로 나눠서 동시에 여러 명령어를 처리하는 기법
>![](https://velog.velcdn.com/images/garage_keeper/post/c49ce1ce-7749-42b5-a4b4-ec7e68dd9b48/image.png)
> * 인출
> * 해독
> * 실행
> * 메모리 접근
> * 레지스터에 쓰기



* 모든 명령어가 모든 단계를 거치지 않는다
* 파이프라인을 거치지 않음
* 조건 분기를 만나면 실행하던거 무효화될 수 있음
  * 분기 예측
  * 목적지 선인출
  * 루프 버퍼
  * 지연 분기