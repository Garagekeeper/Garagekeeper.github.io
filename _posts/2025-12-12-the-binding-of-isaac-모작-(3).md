---
title: "The binding of Isaac 모작 (3)"
date: 2025-12-12 18:29:41 +0900
description: The binding of Isaac의 폭발을 구현해보자
categories: [Portfolio, GunsGermsAndSteel] #[upper, lower]
tags: [unity, c#, addressable] #must be lower case
math: true
mermaid: true
---

이번 파트는 생각보다 간단하니 간략하게 살펴보자


## 3.폭탄
위의 사진에서 찾아 볼 수 있는 아이작 조작의 특징은 다음과 같다
1. E키를 통해서 폭탄을 소환한다.<br>
2. 개수 제한이 있다.<br>
3. 일정 시간이 지난후 약 2.5칸의 크키로 폭빌한다 (WhiteFlash 추후 적용)<br>
4. 플레이어와 몬스터에게는 다른 데미지를 적용한다.
<br>

<br>
<br>

### 1. E키를 통해서 폭탄을 소환한다.
### 2. 개수 제한이 있다.
 MainCharacter.cs
```cs
if (Input.GetKeyDown(KeyCode.E))
    SpawnBomb();
```

MainCharacter.cs
```cs
 public void SpawnBomb()
{
    GameObject go = Managers.Resource.Instantiate("Bomb");

    Bomb bomb = go.GetComponent<Bomb>();
    bomb.SetInfo(this);
    BombCount--;
}
```
  
  현재 addresible을 사용하여 처음 게임을 시작할 때 필요한 리소스들을 딕셔너리 형태로 메모리에 저장하고 있다.
(추후에 기회가 된다면 자세히 써보겠다)
  
위의 코드는 "Bomb"이라는 키 값을 가지는 프리팹을 인스턴스화 해주고 객체의 정보를 기입한다.

  <br><br>
  
### 3. 일정 시간이 지난후 약 2.5칸의 크키로 폭발한다 (WhiteFlash 추후 적용)  
  
Bomb.cs
```cs
 public void SetInfo(Creature Owner)
{
    this.Owner = Owner;
    transform.position = Owner.transform.position;
    _animator = GetComponent<Animator>();

    _coroutine = StartCoroutine(CoReserveExplosion(time));
}
```

Bomb.cs
```cs
 private IEnumerator CoReserveExplosion(float limit)
 {
    yield return new WaitForSeconds(limit);
    BombExplosion();
}


public void BombExplosion()
{
    _animator.Play("BombExplode");
    Collider2D[] hit = Physics2D.OverlapBoxAll(transform.position, new Vector2(Range, Range), 0);
    foreach (Collider2D collider in hit)
    {
        var temp = collider.gameObject;
        //TODO 폭탄의 경우 여러 물체와 상호작용한다
        //모든 Object의 조상을 만들어서 Object 타입을 통해 적절한 상호작용으로 교체하자.
        temp.GetComponent<Creature>()?.OnDamaged(Owner, _skillType);
    }
}

public void DestroyBomb()
{
    Destroy(gameObject);
}
```

  역시 별다른 내용은 없고, 일정 시간이 지난 후 범위만큼의 정사각형 안에 포함된 Collider를 반환하고
  안에 있는 콜라이더가 Creature일 경우 데미지를 입는 과정을 진행한다.
  
  Animation Event를 통해서 원하는 시점에 객체를 Destroy한다.
<br><br>


### 4. 플레이어와 몬스터에게는 다른 데미지 공식을 적용한다.

Creature.cs
```cs
public virtual void OnDamaged(Creature owner, ESkillType skillType)
{
    switch (skillType)
    {
        case ESkillType.BodySlam:
            //TODO
            break;
        case ESkillType.Bomb:
            Hp -= BombDamage;
            break;
        case ESkillType.Projectile:
            Hp -= AttackDamage;
            break;
        case ESkillType.Fire:
            break;
        case ESkillType.Spike:
            break;
    }

    Debug.Log(Hp);

    if (Hp < 0)
    {
        OnDead();
        return;
    }

}
```
  
MainCharacter.cs
```cs
public override void OnDamaged(Creature owner, ESkillType skillType)
{
    Hp -= DamageByOtherConstant;

    Debug.Log(Hp);

    if (Hp <= 0)
    {
        OnDead();
        return;
    }
}
 ```


  함수 오버라이딩을 통해서 간단히 구현 할 수 있었다.
  
  아이작에서는 플레이어가 받는 데미지가 항상 일정하다 (하트 반칸, 하트 한칸) 그래서 특정 아이템을 먹거나 하면 
  이 값이 달라지기 때문에 별도의 변수로 관리하려고 했다.
  
  <br><br>
  사실 이번에 폭탄을 구현할 때 Animation이 제일 까다로웠다.
  그걸 제외하면 클래스의 관계나 Addresible을 통한 리소스 관리에 대해서 조금 생각해 볼 수 있었다.
  
  
  (이 글을 쓰는 시점에는 Item과 관련한 코드를 작성중인데 죽을 맛이다, 아이작에는 아이템이 왜이리 많은지.....)
