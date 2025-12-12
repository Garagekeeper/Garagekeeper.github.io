---
title: "The binding of Isaac 모작 (6)"
date: 2025-12-12 18:39:37 +0900
description: The binding of Isaac의 보스들을 구현해보자
categories: [Portfolio, GunsGermsAndSteel] #[upper, lower]
tags: [unity, c#, addressable] #must be lower case
math: true
mermaid: true
---

## 6. Boss, Monster
아이작에는 많은 종류의 몬스터와 보스들이 있다.
이번 포스팅에는  보스 몬스터 Gurdy, GurdyJr에 관해서 포스팅을 해볼까 한다
보스의 세부 기획은 [링크](https://bindingofisaacrebirth.fandom.com/wiki/Gurdy)를 참고하여 만들었다.<br>
![](https://velog.velcdn.com/images/garage_keeper/post/700de753-8693-4c28-8d14-db4439ad416a/image.png)
<p align="center">
  Gurdy
</p>

![](https://velog.velcdn.com/images/garage_keeper/post/c2cc8f2b-37c2-4460-bce1-d0e14bc0481a/image.png)
<p align="center">
  Gurdy Jr
</p>

<br>
<br>

### 1. 기본 구조
#### 1) 보스는 코루틴을 이용해서 Update문 외부에서 동작한다.
  UpdateAiTick을 통해서 주기를 조절 할 수 있다.
```cs
private void Awake()
{
    Init();
    StartCoroutine(CoUpdateAI());
}

protected override IEnumerator CoUpdateAI()
{
    while (true)
    {
        switch (BossState)
        {
            case EBossState.Idle:
                UpdateIdle();
                break;
            case EBossState.Skill:
                UpdateSkill();
                break;
            case EBossState.Move:
                UpdateMove();
                break;
            case EBossState.Dead:
                break;
            case EBossState.Explosion:
             break;
        }

        if (UpdateAITick > 0)
            yield return new WaitForSeconds(UpdateAITick);
        else
         yield return null;
    }
}
```
 
  위의 코드를 간단하게 살펴보면
  Awake시 코루틴을 실행해서 BossState에 따라서 적절한 함수를 실행한다.
  이를 기반으로 Boss를 상속받는 Class들은 UpdateIdle(), UpdateSkill(),  UpdateMove() 등의 함수를 override 해서 각 Class에 맞게 작성해서 사용한다.

#### 2) UpdateSkill()
``` cs
protected override void UpdateSkill()
{
    if (_coWait != null) return;

    float delay = 0;

    AnimatorBottom.Play(_skillName[(int)_currentSkill], 0, 0);
    if (_skillName[(int)_currentSkill] != AnimatorBottom.GetCurrentAnimatorClipInfo(0)[0].clip.name)
         return;
    delay = AnimatorBottom.GetCurrentAnimatorClipInfo(0)[0].clip.length;

    StartWait(delay);
 }
```
  
UpdateSkill()은 대부분의 보스가 공통으로 사용하게 될거 같아 살펴보고 지나가자
* 현재 실행중인 코루틴(재생중인 Animation)이 있으면 종료한다. (_coWait이 존재하면)
* Boss의 현재 Skill에 해당하는 Animation을 재생한다.
* 현재 재생중인 Animation의 길이를 추출한다.
* 위에서 추출한 길이만큼 wait을 시작한다 (_coWait을 등록한다.)
  * 이를 통해서 Animation이 끝까지 재생됨을 보장

  
### 2.Gurdy
Boss_Gurdy.cs
```cs
    protected override void UpdateIdle()
    {
        base.UpdateIdle();

        //0.가장 가까운 목표 탐색
        Target = FindClosetTarget(this, Managers.Object.MainCharacters.ToList<Creature>());

        int randomValue = Random.Range(0, 100);

        //1. Skill 실행
        if (randomValue < 40f)
        {
            // 40% 확률로 SkillA 실행
            _currentSkill = EBossSkill.SkillA;
        }
        else if (randomValue < 70f)
        {
            // 30% 확률로 SkillB 실행 (40~70)
            _currentSkill = EBossSkill.SkillB;
        }
        else
        {
            // 나머지 30% 확률로 SkillC 실행 (70~100)
            _currentSkill = EBossSkill.SkillC;
        }
        BossState = EBossState.Skill;
    }

    protected override void UpdateSkill()
    {
        if (_currentSkill == EBossSkill.SkillA)
        {
            if (_coWait != null) return;
            string skillName = "";
            Vector2 dV = Target.transform.position - transform.position;

            //플레이어가 위쪽인 경우
            if (dV.y > 0)
            {
                if (dV.x > 0)
                    skillName = _skillName[(int)_currentSkill] + "_L";
                else
                    skillName = _skillName[(int)_currentSkill] + "_R";
            }
            //플레이어가 아래쪽인 경우
            else
            {
                if (Mathf.Abs(dV.x) > Mathf.Abs(dV.y) && dV.x > 0)
                    skillName = _skillName[(int)_currentSkill] + "_L";
                if (Mathf.Abs(dV.x) > Mathf.Abs(dV.y) && dV.x < 0)
                    skillName = _skillName[(int)_currentSkill] + "_R";
                if (Mathf.Abs(dV.x) < Mathf.Abs(dV.y))
                    skillName = _skillName[(int)_currentSkill] + "_D";

            }

            AnimatorBottom.Play(skillName, 0, 0);
            if (skillName != AnimatorBottom.GetCurrentAnimatorClipInfo(0)[0].clip.name)
                return;
            float delay = AnimatorBottom.GetCurrentAnimatorClipInfo(0)[0].clip.length;

            StartWait(delay);
        }
        else
        {
            base.UpdateSkill();
        }
    }

    public void Generate5Projectil(int vec)
    {
        Vector2 dV = Vector2.zero;
        if (vec == 0)
        {
            dV = Vector2.right;
        }
        else if (vec == 1)
        {
            dV = Vector2.down;
        }
        else if (vec == 2)
        {
            dV = Vector2.left;
        }

        GenerateProjectile(dV, false, true);
        for (int i = 1; i <= 2; i++)
        {
            GenerateProjectile(VectorRotation2D(dV, 10f * i), false, true);
        }
        for (int i = 1; i <= 2; i++)
        {
            GenerateProjectile(VectorRotation2D(dV, -10f * i), false, true);
        }
    }

    //Spawn Boils
    public void SkillB()
    {
        foreach (Monster m in Managers.Object.Monsters)
        {
            if (m.MonsterType == EMonsterType.Boil) return;
        }

        Managers.Object.Spawn<Monster>(transform.position + new Vector3(0.5f, -3f), 0, "Boil");
        Managers.Object.Spawn<Monster>(transform.position + new Vector3(-0.5f, -3f), 0, "Boil");
    }

    //Spawn 2 fly or 1 pooter
    public void SkillC()
    {
        int randomValue = Random.Range(0, 100);

        if (randomValue > 50)
        {
            Managers.Object.Spawn<Monster>(new Vector3(transform.position.x + 1f, -1.65f), 0, "Fly");
            Managers.Object.Spawn<Monster>(new Vector3(transform.position.x - 1f, -1.65f), 0, "Fly");
        }
        else
        {
            Managers.Object.Spawn<Monster>(new Vector3(transform.position.x - 1f, -1.65f), 0, "Pooter");
        }
    }
```

  차근차근 살펴보면 다음과 같다
  * UpdateIdle(): 주석에 적힌대로 확률에 따라서 적절한 스킬로 넘어간다
  * UpdateSkill(): SkillA의 경우 Player의 위치에 따라서 스프라이트가 달라진다.
    * 각 방향에 맞는 Animation을 추가했고 대응하는 코드를 추가 (사실상 base.UpdateSkill()에서 skillName 부분만 수정한것)
    * 나머지 스킬들은 base.UpdateSkill() 사용
  * 아래에 있는 3개의 함수는 Animation Event에 의해서 호출된다.
    * ![](https://velog.velcdn.com/images/garage_keeper/post/5c7dc854-938c-4d99-94da-a3464d393f98/image.PNG)
  * SkillA
    * ![](https://velog.velcdn.com/images/garage_keeper/post/677c2857-2356-4228-a98d-39ece3017124/image.gif)

  * SkillB
    * ![](https://velog.velcdn.com/images/garage_keeper/post/47630488-348e-4238-950d-41116087a700/image.gif)
	

  * SkillC
    * ![](https://velog.velcdn.com/images/garage_keeper/post/218f6f6c-ba6f-4954-8608-743a7135ee13/image.gif)
    * ![](https://velog.velcdn.com/images/garage_keeper/post/33f44bd7-f4ca-4e40-a33d-20aecf3b8180/image.gif)
  
  _~~**GIF 프레임 때문에 Animation이 제대로 안보인다**~~_


<br><br>
  
### 3.GurdyJr
```cs

private void FixedUpdate()
{
    if (BossState == EBossState.Dead) return;

    if (Rigidbody.velocity.magnitude > 0.01f)
    {
        _previousVelocity = Rigidbody.velocity;
    }

        Rigidbody.velocity = _vel;
}

protected override void UpdateIdle()
{
    if (Managers.Object.MainCharacters.Count == 0) return;
    if (BossState == EBossState.Dead) return;

    _vel = Vector3.zero;

    //0. 가장 가까운 목표 탐색 및 거리 계산
    Target = FindClosetTarget(this, Managers.Object.MainCharacters.ToList<Creature>());

    //Phase1
    if (Hp > MaxHp / 2)
    {
        int rnd = Random.Range(0, 3);
        //SkillA
        if (rnd == 0)
        {
            _currentSkill = EBossSkill.SkillA;
        }
        //SkillB
        else if (rnd == 1)
        {
            _currentSkill = EBossSkill.SkillB;
        }
        //SkillC
        else if (rnd == 2)
        {
            _currentSkill = EBossSkill.SkillC;
        }
    }
    //Phase2
    else
    {
        _currentSkill = EBossSkill.SkillC;
        //SkillC continuously
    }
    BossState = EBossState.Skill;
}

protected override void UpdateSkill()
{
    if (CreatureState == ECreatureState.Dead) return;
    if (_coWait != null) return;

    sequence.Kill();
    sequence = null;
    sequence = DOTween.Sequence();

    switch (_currentSkill)
    {
        case EBossSkill.SkillA:
            SkillA();
            break;
        case EBossSkill.SkillB:
            SkillB();
            break;
        case EBossSkill.SkillC:
            SkillC();
            break;
        default:
            break;
    }

    sequence.Play();
    float delay = 0;
    delay = sequence.Duration();
    StartWait(delay);
}

//Spawn Pooter
public void SkillA()
{
    sequence.Append(DOTween.To(() => 0f, x => Bottom.sprite = Managers.Resource.Load<Sprite>("boss_021_gurdyjr_6"), 0f, 0f));
    sequence.Append(transform.DOShakeScale(1, 0.1f, 10, 90, false));
    sequence.Insert(0.5f, DOTween.To(() => 0f, x => Bottom.sprite = Managers.Resource.Load<Sprite>("boss_021_gurdyjr_2"), 0f, 0f));
    sequence.Insert(0.5f, DOTween.To(() => 0f, x => SpawnPooter(), 0f, 0f));
    sequence.Append(DOTween.To(() => 0f, x => Bottom.sprite = Managers.Resource.Load<Sprite>("boss_021_gurdyjr_8"), 0f, 0f));
    sequence.Append(transform.DOShakeScale(0.5f, 0.1f, 10, 90, false));
    sequence.OnComplete(() => { BossState = EBossState.Idle; _currentSkill = EBossSkill.Normal; });
}

//Jump and generate 8 projectile
public void SkillB()
{
    sequence.Append(DOTween.To(() => 0f, x => x = 1, 0f, 0.5f));
    sequence.Append(transform.DOShakeScale(1, 0.1f, 10, 90, false));
    sequence.Join(transform.DOJump(transform.position, 3, 1, 0.5f));
    sequence.Insert(0.7f, DOTween.To(() => 0f, x => Bottom.sprite = Managers.Resource.Load<Sprite>("boss_021_gurdyjr_4"), 0f, 0f));
    sequence.Insert(0.9f, DOTween.To(() => 0f, x => Bottom.sprite = Managers.Resource.Load<Sprite>("boss_021_gurdyjr_8"), 0f, 0f));
    sequence.Insert(0.95f, DOTween.To(() => 0f, x => Generate8Projectil(), 0f, 0f));
    sequence.Append(transform.DOShakeScale(0.5f, 0.1f, 10, 90, false));
    sequence.OnComplete(() => { BossState = EBossState.Idle; _currentSkill = EBossSkill.Normal; });
}

//charge attack
public void SkillC()
{
    sequence.Append(transform.DOShakeScale(3f, 0.1f, 10, 90, false));
    sequence.Join(DOTween.To(() => 0f, x => Bottom.sprite = Managers.Resource.Load<Sprite>("boss_021_gurdyjr_11"), 0f, 0f));
    sequence.Join(DOTween.To(() => 0f, x => ChargeAttackt(), 0f, 0f));
    sequence.Append(DOTween.To(() => 0f, x => _vel = Vector2.zero, 0f, 0.5f));
    sequence.Join(DOTween.To(() => 0f, x => Bottom.sprite = Managers.Resource.Load<Sprite>("boss_021_gurdyjr_8"), 0f, 0f));
    sequence.OnComplete(() => { BossState = EBossState.Idle; _currentSkill = EBossSkill.Normal;});
}

public void SpawnPooter()
{
    Managers.Object.Spawn<Monster>(transform.position - new Vector3(0, 0.5f, 0), 0, "Pooter");
}

public void Generate8Projectil()
{
    Vector2 dV = Vector2.right;
    for (int i = 0; i < 8; i++)
    {
        GenerateProjectile(VectorRotation2D(dV, 360 / 8 * i), false, true);
    }
}

public void ChargeAttackt()
{
    _vel = (Target.transform.position - transform.position).normalized * Speed;
}

private void OnCollisionEnter2D(Collision2D collision)
{
    if (collision.gameObject.tag == "Player" || collision.gameObject.tag == "Projectile")
    {
        return;
    }
    if (_currentSkill == EBossSkill.SkillC)
    {
        _vel = Vector3.Reflect(_previousVelocity, collision.GetContact(0).normal);
    }
}

private void OnDestroy()
{
    sequence.Kill();
    sequence = null;
}
```
  
   차근차근 살펴보면 다음과 같다
  * UpdateIdle(): 체력에 따라 phase가 나뉜다.
    * phase1에서는 확률에 따라서 적절한 스킬로 분기한다
    * phase2에서는 한가지 스킬만 사용한다
  * UpdateSkill(): 앞서 본 Gurdy와 달리 override해서 사용 하였다
  	* 기존의 UpdateSkill()을 Animation clip이 아니라 DOTween.Sequence를 사용한 버전으로 조금 바꾼것.
    * [DOTween](https://dotween.demigiant.com/)을 사용 자세한 내용은 [Documentation](https://dotween.demigiant.com/documentation.php)을 살펴보자
      * GurdyJr의 경우 Sprite의 변화가 비교적 적고, 점프 흔들리는 효과가 많다
      * Animation으로 흔들리는 효과를 구현하다 ~~화가나서~~ 해당 라이브러리를 알게 되었다.
      * DOShakeScale() 같이 미리 정의된 함수를 사용해서 조금 더 편리했던거 같다.
      * sequence를 사용해서 Animation clip을 사용하듯 사용할 수 있었다.
  	  * 람다를 통해서 Animation Event와 유사하게 사용할 수 있었다.
    
  * SkillA
    * ![](https://velog.velcdn.com/images/garage_keeper/post/000a2193-8c9c-442d-be46-6db7fccaf67f/image.gif)

  * SkillB
    * ![](https://velog.velcdn.com/images/garage_keeper/post/90231a9c-c301-4c15-8b92-edfa0c945add/image.gif)
	
  * SkillC
    * ![](https://velog.velcdn.com/images/garage_keeper/post/a31dc543-3964-47d0-bcba-ea593046cc19/image.gif)

<br><br>


### 4.여담
  
 * Animation vs DOTween
   * sprite 위주면 Animation, transform 위주면 DOTween이 편했던 것 같다.
     * Shake, Jump등 많은 기능이 미리 구현되어 있어서 편했다.
     * 프레임이 아니라 시간 단위라 스크립팅이 편했다.
  

 
