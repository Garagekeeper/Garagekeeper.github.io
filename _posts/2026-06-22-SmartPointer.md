---
title: "Smart Pointer"
date: 2026-06-22 09:02:46 +0900
description: ""
categories: [Unreal Engine, GC] #[upper, lower]
tags: [cpp, unreal, memory_management ,unreal_gc] #must be lower
math: true
mermaid: true
---

<br>

# 동적 할당

<br>

## c style
```cpp
// 100byte의 메모리를 동적 할당하고 이를 int*로 캐스팅
int* p = (int*)malloc(100);
```
* void* 타입을 반환하기 때문에 캐스팅해서 사용
* 실패시 NULL 리턴
* 필요한 사이즈를 명시해야함
* heap에 할당

<br>

## c++ style
```cpp
int* p = new int();
```
* 요청한 타입의 포인터를 반환
* 실패시 기본적으로 예외(Bad_alloc)를 Throw
* 컴파일러가 자동으로 사이즈를 판단해줌
* 연산자 오버로딩 가능	
* 생성자 소멸자 사용가능

* new expression vs operator new
  * new expression : 우리가 아는 new
  * operator new , operator new[] : 할당 함수, 메모리를 할당해서 넘겨주는 함수
    * 재정의 가능
  * new expression 과정 Myclass* myc = new Myclass();
    * operator new를 실행해 새로운 메모리를 할당
	  * void* rawMemory = operator new(sizeof(MyClass));
	* placement new를 통해서 위의 메모리를 초기화
	  * myc = new (rawMemory) MyClass();
	  * 반드시 delete를 쓰지 말고, 소멸자를 명시적으로 호출할 것.
	  * 메모리 풀에서 메모리를 받아 placement new를 통해서 객체를 생성했다면 절대 delete를 사용하지마, 반드시 소멸자를 명시적으로 호출해. (delete는 메모리를 해제하려고 하는데, 풀의 일부분을 할당 해제하는 것은 안된다! 해제해도 풀 전체를 해제해야함.)

<br>
<br>


# 발생할 수 있는 문제들

* memory leak : 해제를 하지 않아서 메모리가 계속 점유되는 상태
* double free : 해제한 메모리를 다시 해제하려는 경우
* use after free : 해제된 메모리를 사용하려는 경우

<br>

# 문제 해결 방안
## ref count
```cpp
// sudo class
class RefCountedObject
{
private:
	atomic<int32> refCount;

public:
	RefCountedObject() : refCount(1) {}
	Add()
	{
		refCount++;
	}
	Release()
	{
		refCount--;
		if (refCount == 0)
			delete this;
	}
}

class MyClass : public RefCountedObject
{

}

int main()
{
	Myclass* myc = new Myclass();
	Myclass* myc2 = myc;

	myc2->Add();

	myc2->Release();
	myc2 = nullptr;

	myc->Release();

	// 사실 복사 대입 연산자, 복사 생성자, 소멸자 등에서 처리를 하면 되지만
	// 전체적인 흐름을 보기 위해서 수동으로 진행
}
```
* 위와 같은 클래스를 상속 받아서 레프카운트를 관리
* 처음 만들 때만 new를 사용하고 그 이후에는 임의로 new delete를 사용하지 않는다.

<br>

* 단점 
  * 상속을 받는 것 자체가 단점
  * atomic으로 변수를 선언하면 오버헤드가 있음
  * 그렇다고 atomic을 사용하지 않으면 멀티쓰레드에서 문제가 발생함

<br>

## 스마트 포인터
* Ref Count를 자동으로 관리하는 래퍼 클래스를 만들자.
```cpp
// 처음 생성만 new, 나머지는 TRefCounterPtr클래스가 알아서 관리
// sp를 일반 포인터처럼 사용
TRefCounterPtr<x> sp(new x());
```

### Shared_ptr

![](https://learn.microsoft.com/en-us/cpp/cpp/media/shared_ptr.png?view=msvc-160)
* 오브젝트에 관여하는 포인터
* 구조와 원리
  * 구조
     * 오브젝트 포인터 (개체를 가리키는 포인터)
     * 제어 블록 포인터
		* Strong Count : 오브젝트 관리에 영향을 주는 Count
		* Weak Count   : 제어블록 관리에 영향을 주는 Count
  * shared_ptr을 생성하면 제어블록이 새로 할당되고, 여기에 Ref Count를 기록
    * strong = 1, weak = 1
	* strong = 0이 되면 원본 객체를 delete
  * -> 연산자를 오버라이딩해 일반 포인터처럼 사용가능

* 단점 
  * 순환참조(해제되지 않음)
    * ex : 플레이어와 인벤이 서로 shared_ptr로 참조하고 있는 경우 서로의 Strong Count가 1에서 줄지 않는다.
	
* 참고사항
  * 절대 shared_ptr을 만들 때 this로 만들지마!
    * 이 경우 제어블록이 새로 생성되 의도한 용도로 쓸 수 없다.
	  * 정확히 말하면 shared_ptr을 원시 포인터로 만들면 새로운 제어블록을 생성한다.
	  * this는 자기 자신을 가리키는 포인터이기 때문에 새로운 제어블록을 생성하게 된다.
    * 전용 함수가 따로 있음
	   * c++ : `std::enable_shared_from_this` 상속받은 뒤, `shared_from_this()` 사용
	   * Unreal : `TSharedFromThis<T>` 상속받은 뒤, `AsShared()` 사용
	     * 표쥰과는 다르게 참조를 반환
	   * 상속을 받은 클래스 내부에는 weak_ptr 멤버 변수가 있고 이를 통해서 이미 생성된건지 구분한다
	   * shared_ptr이 최초로 생성되면 (weak_ptr이 비어있으면) 제어블록 만들고 weak_ptr 멤버 변수 채움
	   * weak_ptr 멤버 변수가 있으면 해당 제어블록의 shared_ptr 생성

   <br>
   
	* shared_ptr를 생성할 때 make_shared를 사용하자
	  * shared_ptr<T>(new T)
	    * 이 경우 T를 동적할당 하고, 제어 블록도 동적할당.
      * make_shared<T>()
	    * T와 제어블록을 사용할 메모리를 한번에 할당

### weak_ptr

```
	[ SharedPtr ]                [ Control Block ]          [ MyClass Object ]
+----------------+          +-------------------+      +------------------+
| Ptr to Object  |--------->| Strong Count: 1   |      |                  |
+----------------+          | Weak Count: 1     |      |    MyClass       |
| Ptr to CB      |--+       +-------------------+      |                  |
+----------------+  |       | Ptr to Object     |----->| (데이터 및 기능)   |
                    |       +-------------------+      +------------------+
                    |                 ^
                    |                 |
[ WeakPtr ]         |                 |
+----------------+  |                 |
| Ptr to CB      |--+-----------------+
+----------------+
```

* 오브젝트에 관여하는 것이 아니라, 제어 블록에 관여하는 포인터
  * weak count만 증가시킴

* 구조
  * 제어 블록 포인터
  * 오브젝트 포인터 (소유권을 넘겨받는 개념이 아니라 그냥 참고용)

* 왜 오브젝트 포인터를 들고 있을까?
  * lock(native)/pin(unreal) 기능을 통해서 새로운 Shared_ptr을 만들어야 하니까

* 플레이어 인벤 관계에서
  * 플레이어는 인벤은 Shared_ptr로
  * 인벤은 플레이어를 weak_ptr로 들고 있으면 순환참조가 해결
  * 하지만 lock/pin을 이용해서 변환하고 이것이 유효한지 확인해서 써야한다. (조금 번거로움)



