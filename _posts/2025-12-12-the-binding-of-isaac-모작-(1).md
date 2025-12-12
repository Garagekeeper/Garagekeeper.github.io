---
title: "The binding of Isaac 모작 (1)"
date: 2025-12-12 18:20:59 +0900
description: The binding of Isaac의 움직임을 구현해보자
categories: [Portfolio, GunsGermsAndSteel] #[upper, lower]
tags: [unity, c#, addressable] #must be lower case
math: true
mermaid: true
---

요즘 재미있게 하고있는 게임인 The Binding of isaac의 모작을 하기로 했다.
아이작은 게임자체에서 리소스 추출 툴을 제공해 주기도 하고, 시드 기반 랜덤 맵 등 흥미로운 부분이 많아서 진행해보기로 했다.
<br>

>### 들어가기 전에!
>아이작은 리소스 추출툴을 지원해서 편리하게 모든 리소스를 추출할 수 있다.
하지만 필요한 리소스만 사용하고 싶다면 아래의 링크도 참고해보자
>[아이작 Sprite](https://www.spriters-resource.com/pc_computer/bindingofisaacrebirth/)

무얼 해야할지 모를때는 게임 플레이에서 가장 처음부분을 생각해보자
![](https://velog.velcdn.com/images/garage_keeper/post/6445cca0-f0f3-420d-ae3c-3c7d47acb306/image.png)
  <figcaption style="text-align:center; font-size:15px; color:#808080">
    작고 귀여운(?) 게임 아이작
  </figcaption>
  <br>
위의 사진을 유심히 살펴보면 각종 기능 및 컨텐츠를 알 수 있는데 굵직한 것들을 살펴보자면

* 이동
* 공격
* 폭탄
* 아이템
* 맵
* 미니맵

이 기능들을 구현해보고, 나머지 나타나지 않은 기능들도 구현해보자

## 1.이동
위의 사진에서 찾아 볼 수 있는 아이작 조작의 특징은 다음과 같다
>1. WASD
><br>2. 캐릭터가 멈출 때 미끄러진다.
><br>3. 이동할 때 캐릭터가 해당 방향을 바라본다.
><br>4. 캐릭터가 바라보는 방향은 수평보다 수직에 우선순위가 있다.
><br>

### 1. WASD
### 2. 캐릭터가 멈출 때 미끄러진다.
>```cs
 Vector2 vel = Rigidbody.velocity;
 vel.x = Input.GetAxis("Horizontal") * Speed;
 vel.y = Input.GetAxis("Vertical") * Speed;
>
> if (Input.GetKey(KeyCode.A) && Input.GetKey(KeyCode.D))
>     vel.x = 0;
> if (Input.GetKey(KeyCode.W) && Input.GetKey(KeyCode.S))
>     vel.y = 0;
>
> Rigidbody.velocity = vel;
>```
> <figcaption style="text-align:center; font-size:15px; color:#808080">
    PlayerController.cs 의 일부분</figcaption>
	
위의 코드는 1과 2를 만족한다.
**Input.GetAxis()** 함수를 통해서 수직 수평 입력을 받는다. (InputManager를 통해서 키설정 등을 할 수 있다)
**Input.GetAxisRaw()** 와 다르게 -1 ~ 1의 값을 float 형태로 전달하기 때문에 뚝뚝 끊기지 않고 미끄러지는 느낌을 준다.
물론 AddForce() 함수등 다양한 방법이 있지만 갈 길이 멀기에 가장 먼저 떠오른 방법을 사용했고 움직임의 디테일은 추후에
신경을 쓰기로 했다.
<br>
### 3.이동할 때 캐릭터가 해당 방향을 바라본다.
### 4.캐릭터가 바라보는 방향은 수평보다 수직에 우선순위가 있다.
여기가 사실 지금까지도 고민이 되는 부분이다.
아이작 Sprite를 살펴보면 움직임과 관련된 sprite는 상하체가 분리되어있다...
(상하체를 통으로 스프라이트를 만들면 리소스 파일이 어마어마하게 커질 것 같긴하다)
![](https://velog.velcdn.com/images/garage_keeper/post/90014d79-c60f-47c6-b59e-5df7f0f2abe4/image.png)
  <figcaption style="text-align:center; font-size:15px; color:#808080">
    원래 이런 게임이긴 하지만 분리되어 있어서 깜짝 놀랐다.
  </figcaption>

이렇다 보니, 상태 기반으로 동작을 관리하려고 했는데 고민이 생겼었다.
Player는 크게보면 **Attack, Idle, Move**의 상태를 가지는 것 처럼 보인다. 하지만 이 게임은 **움직이면서 공격**을 할 수 있기에 
조금 곤란한 상황이었다.<br>
그래서 상하체가 각자 다른 상태를 가지고 있도록 해봤다. 각 상태는 Enum으로 정의했다.
하체는 크게보면 **Idle, Move**의 상태를 가지고
상체는 **Attack, Idle**의 상태를 가진다. 또한 어느방향을 바라보느냐를 알려주는 상태를 따로 가진다.
>```cs
protected EPlayerBottomState _bottomState = EPlayerBottomState.Idle;
protected EPlayerHeadState _headState = EPlayerHeadState.Idle;
protected EPlayerHeadDirState _headDirState = EPlayerHeadDirState.None;
public EPlayerBottomState BottomState
{
    get { return _bottomState; }
    set
    {
        if (_bottomState != value)
        {
            _bottomState = value;
            UpdateBottomAnimation();
        }
    }
}
>
public EPlayerHeadState HeadState
{
    get { return _headState; }
    set
    {
        if (_headState != value)
        {
            _headState = value;
        }
    }
}
>
public EPlayerHeadDirState HeadDirState
{
    get { return _headDirState; }
    set
    {
        if (_headDirState != value)
        {
            _headDirState = value;
            UpdateFacing();
        }
    }
}
>```

Update() 에서 상태를 변화하면 프로퍼티를 통해서 값이 변화할 때만 변경한다.

의도를 간단히 설명해 보자면
* BottomState를 통해서 하체가 Idle 인지 Move인지 구분한다.
  Move의 경우 하체에 걷는 애니메이션을 재생한다.

* 머리의 상태 HeadState는 Idle, Attack 이 있다.
 

* 바라보는 방향 HeadDirState은 상하좌우 어디를 바라보는지 저장한다.
 HeadState가 Attack의 경우 공격하는 쪽을 계속 바라보도록 애니메이션을 재생한다.
 HeadState가 Idle의 경우 움직이는 방향을 바라보도록 스프라이트를 조정한다.
 
>```cs
public void UpdateBottomAnimation()
{
    switch (BottomState)
    {
        case EPlayerBottomState.Idle:
            Bottom.flipX = false;
            AnimatorBottom.Play("Idle");
            break;
        case EPlayerBottomState.MoveDown:
            Bottom.flipX = false;
            AnimatorBottom.Play("Walk_Down");
            break;
        case EPlayerBottomState.MoveUp:
            Bottom.flipX = true;
            AnimatorBottom.Play("Walk_Down");
            break;
        case EPlayerBottomState.MoveLeft:
            Bottom.flipX = true;
            AnimatorBottom.Play("Walk_Horiz");
            break;
        case EPlayerBottomState.MoveRight:
            Bottom.flipX = false;
            AnimatorBottom.Play("Walk_Horiz");
            break;
        case EPlayerBottomState.OnDamaged:
            break;
        case EPlayerBottomState.OnDead:
            break;
>
 >   	}
>}
>
>```

>```cs
public void UpdateFacing()
{
    if (HeadState == EPlayerHeadState.Attack)
    {
        AnimatorHead.enabled = true;
        //UpdateHeadAnimation();
    }
    else
    {
        AnimatorHead.enabled = false;
        UpdateHeadSprite();
    }
}
>```

>```cs
    public void UpdateHeadSprite()
    {
        switch (HeadDirState)
        {
            case EPlayerHeadDirState.Up:
                Head.flipX = false;
                Head.sprite = HeadSprite[0];
                break;
            case EPlayerHeadDirState.Down:
                Head.flipX = false;
                Head.sprite = HeadSprite[1];
                break;
            case EPlayerHeadDirState.Left:
                Head.flipX = true;
                Head.sprite = HeadSprite[2];
                break;
            case EPlayerHeadDirState.Right:
                Head.flipX = false;
                Head.sprite = HeadSprite[2];
                break;
        }
    }
>```

> 여기서 4.캐릭터가 바라보는 방향은 수평보다 수직에 우선순위가 있다. 또한 해결된다.
>```cs
> Rigidbody.velocity = vel;
> if (vel != Vector2.zero)
 {	 //4.캐릭터가 바라보는 방향은 수평보다 수직에 우선순위가 있다
 	 // 수직축은 안 누르면 바로 바로 바뀌도록 조건문에 포함 ()
     if (vel.y != 0 && (Input.GetKey(KeyCode.W) || Input.GetKey(KeyCode.S)))
     {
         BottomState = vel.y > 0 ? EPlayerBottomState.MoveUp : EPlayerBottomState.MoveDown;
         if (HeadState == EPlayerHeadState.Idle)
             HeadDirState = vel.y > 0 ? EPlayerHeadDirState.Up : EPlayerHeadDirState.Down;
     }
     else if (vel.x != 0)
     {
         BottomState = vel.x > 0 ? EPlayerBottomState.MoveRight : EPlayerBottomState.MoveLeft;
         if (HeadState == EPlayerHeadState.Idle)
             HeadDirState = vel.x > 0 ? EPlayerHeadDirState.Right : EPlayerHeadDirState.Left;
     }
 }
 else
 {
     BottomState = EPlayerBottomState.Idle;
     if (HeadState == EPlayerHeadState.Idle)
         HeadDirState = EPlayerHeadDirState.Down;
 }
```

![](https://velog.velcdn.com/images/garage_keeper/post/48a10746-5bfa-403f-b852-cb04f6aef557/image.gif)

3가지의 FSM을 관리하려다보니 조금 헷갈리는 부분도 있는것 같다. 
일단 만들어보고 문제가 생기면 수정하는 방향으로 가보자


