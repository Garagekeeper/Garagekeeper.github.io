---
title: "Compaer And Swap"
date: 2025-12-13 00:24:34 +0900
description: "Compaer And Swap에 대해서 알아보자"
categories: [Computer Science, OS] #[upper, lower]
tags: [cs] #must be lower
math: true
mermaid: true
---

# CAS (Compaer And Swap)
## 1. CAS란?
CAS는 Compare And Swap의 약자로 [위키] (https://en.wikipedia.org/wiki/Compare-and-swap)의 설명에 따르면,
'**멀티 스레딩에서 동기화 달성을 위해 사용되는 원자적 명령어(atomic instruction)**'이다. 
이 문장을 이해하기 위해서는 몇가지 배경 지식이 필요하다.


## 2. CAS 탐구 
다시 CAS의 의미를 살펴보면 Compare-And-Swap을 원자적으로 수행하는 것 인데. 어떤 경우에 사용할까?

멀티스레드 환경에서는 c++에서 제공되는 mutex와 lock_guard를 사용하는 예제가 많았다. 하지만 이 경우 스레드간의 문맥전환이 많아지고 성능저하의 원인이 되는 상황이 있다고 한다. 
물론 스레드간의 문맥전환은 프로세스간의 문맥전환 보다는 가벼운 작업일 것이다. (CPU와 컴파일러가 계속 발전해가는데 아직도 눈에 띄는 성능저하가 있는지는 개인적인 의문이다.)

그래서 lock을 사용하지 않고 CAS를 사용해 동기화를 보장해준다.

```cpp
//In window
bool compare_exchange_strong(_TVal& _Expected, const _TVal _Desired,
    const memory_order _Order = memory_order_seq_cst)
```
위의 함수는 visual studio에서 제공하는 C++ CAS의 기본 함수이다.
* _Expected : atomic 오브젝트가 가지길 기대하는 값
* _Desired : 기대하는 값을 가졌을 때 대체하길 원하는 값
* _Order : CAS를 실패 하거나 성공했을 때 적용할 memory_order
  * 성공/실패 시 적용할 memory_order를 따로 정할 수 있다.

의사 코드로 만들어 보자면 다음과 같다
```cpp
if (objValue == expected)
{
	objValue = desired;
	return true;
}
else
{
	expected = objValue;
	return false;
}
```
* compare_exchange_strong : 도중에 다른 스레드에 의해 인터럽트가 나면 끝까지 시도
* compare_exchange_weak : 도중에 다른 스레드에 의해 인터럽트가 나면 실패 (spurious fake) (loop와 함께 사용)

이를 기반으로 Lock-free 자료구조를 만들 수 있다.

## 4. CAS의 문제 (ABA Problem)
간단히 말하면, 객체의 변경을 감지하지 못하는 문제이다.
(메모리 주소의 재사용에 관한 문제)
* Stack 
  * A-> B -> C
  * head : A

이 상황에서 아래의 과정을 진행. 

* expected = &A
* desired = B
* head.CAS(expected, desired)
* pop

하지만 멀티 스레드 환경에서 2번과 3번을 실행하기 전에 stack 이 다음과 같은 과정을 겪는다고 가정해보자

* Stack
  * A-> B -> C
  * head : A
  
* Stack
  * B -> C
  * head : B
  
* Stack
  * C
  * head : C
  
* Stack
  * A-> C
  * head : A

다시 원래의 스레드로 돌아오면 여전히 head == expected == A 이므로 head = desired를 실행하지만, B는 해제 되었을 수 있고, 다른 변수가 사용중일 수 있다.

#### 해결 방법
[Double compare-and-swap](https://en.wikipedia.org/wiki/Double_compare-and-swap) 말그대로 2개를 비교한다. 
기존 시스템의 2배의 길이로 비교하며, count 값과 포인터를 비교한다. 
포인터를 사용하면 count값을 증가 시킨다. 향후에 CAS를 할 때 포인터는 물론 count값 까지 동일하면 온전한 상태!

Visual Studo에서는 _InterlockedCompareExchange128를 지원하므로 이를 사용해보자

## 5. 여담
학부시절 운영체제 강의에서 배웠던 내용인데 오랜만에 다시 살펴 보았는데, 의외로 살펴볼 내용이 많아서 놀랐다. Lock-Free 자료구조를 만드는 것은 물론이고 신경 쓸 부분이 너무 많다. 정말로 Lock-Free하게 동작하는지도 검증하는 것이 어려운 점인 듯하다.
듣기로는 이 분야에 특허도 많다고 하는데 왜 그런지 알 것 같다. 
