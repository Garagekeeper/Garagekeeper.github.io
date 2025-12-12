---
title: "The binding of Isaac 모작 (2)"
date: 2025-12-12 18:27:27 +0900
description: The binding of Isaac의 공격을 구현해보자
categories: [Portfolio, GunsGermsAndSteel] #[upper, lower]
tags: [unity, c#, addressable] #must be lower case
math: true
mermaid: true
---

저번에는 기초 이동을 만들었으니 
이번에는 기초 공격을 만들어보자.

요구사항은 다음과 같다
> **1. 방향키를 통해서 공격을 할 수 있다.**
> **2. 수직의 우선순위가 수평보다 높다**
> **3. 공격시 움직이는 방향에 상관없이 머리가 공격방향으로 고정 된다**
> **4. 투사체는 벽 장애물에 부딪히거나 사정거리에 도달하면 사라진다**
> _____________
> (추가)
> **5. 투사체는 최대사정거리 근처에서 바닥을 향해 떨어진다**
> **6. 투사체는 좌우 번갈아 가면서 소환된다.**

<br/><br/>

## 2.공격
### 1. 방향키를 통해서 공격을 할 수 있다.
이동과 비슷하게 구현했다.
※ 발사체 생성은 [Animation Event](https://docs.unity3d.com/2023.2/Documentation/Manual/script-AnimationWindowEvent.html)를 통해서 원하는 타이밍에 생성하자!
```csharp
Vector2 attackVel = Vector2.zero;
attackVel.x = Input.GetAxis("AttackHorizontal");
attackVel.y = Input.GetAxis("AttackVertical");

if (Input.GetKey(KeyCode.LeftArrow) && Input.GetKey(KeyCode.RightArrow))
    attackVel.x *= -1;
if (Input.GetKey(KeyCode.UpArrow) && Input.GetKey(KeyCode.DownArrow))
    attackVel.y *= -1;

UpdateAttack(attackVel);
```
<br/><br/>

### 2. 수직의 우선순위가 높다
이 항목은 저번에 했던방법과 비슷하게 진행하였다.
```csharp
public void UpdateAttack(Vector2 attackVel)
{
    if (attackVel != Vector2.zero)
    {
        HeadDirState = ECreatureHeadDirState.None;
        AnimatorHead.enabled = true;
        HeadState = ECreatureHeadState.Attack;

        if (attackVel.y != 0 && (Input.GetKey(KeyCode.UpArrow) || Input.GetKey(KeyCode.DownArrow)))
        {
            HeadDirState = attackVel.y > 0 ? ECreatureHeadDirState.Up : ECreatureHeadDirState.Down;
        }
        else if (attackVel.x != 0)
        {
            HeadDirState = attackVel.x > 0 ? ECreatureHeadDirState.Right : ECreatureHeadDirState.Left;
        }
    }
    else
    {
        AnimatorHead.enabled = false;
        HeadState = ECreatureHeadState.Idle;
    }
}
```
<br/><br/>

### 3. 공격시 움직이는 방향에 상관없이 머리가 공격방향으로 고정 된다

이부분은 저번에 머리 방향의 상태를 가지는
EPlayerHeadDirState의 값이 변경될 때 호출되는 UpdateFacing()을 통해서 간단히 구현했다

``` csharp
public void UpdateFacing()
{
    if (HeadState == ECreatureHeadState.Attack)
    {
        AnimatorHead.enabled = true;
        UpdateHeadAnimation();
    }
    else
    {
        AnimatorHead.enabled = false;
        UpdateHeadSprite();
    }
}
```

<br/><br/>
### 4. 투사체는 벽 장애물에 부딪히거나 사정거리에 도달하면 사라진다

* 벽 및 장애물에 충돌시 사라진다 -> collider2D
* 사거리에 도달하면 사라진다 -> 기본 스탯을 기준으로 잡고 코루틴을 사용한다 (기존스탯 lifetime 1초)
<br/><br/>

### 5. 투사체는 최대사정거리 근처에서 바닥을 향해 떨어진다
* 여러 방법이 있겠지만, 위에서 사용할 코루틴을 통해서 간단하게 구현했다.

``` csharp
private IEnumerator CoRserveDestroy(float lifetime)
{
    yield return new WaitForSeconds(lifetime * 0.8f);
    if (Mathf.Abs(Rigidbody.velocity.x) > Mathf.Abs(Rigidbody.velocity.y))
        Rigidbody.velocity += new Vector2(0, -2.0f);
    yield return new WaitForSeconds(lifetime * 0.2f);
    Destroy(gameObject);
}
```

<br/><br/>

### 6.투사체는 좌우 번갈아 가면서 소환된다
* 추후 구현 예정

![](https://velog.velcdn.com/images/garage_keeper/post/0c8a3f25-6aff-4889-a15a-9084a5f4f123/image.gif)