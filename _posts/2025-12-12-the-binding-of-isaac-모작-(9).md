---
title: "The binding of Isaac 모작 (9)"
date: 2025-12-12 18:52:25 +0900
description: The binding of Isaac의 사운드를 구현해보자
categories: [Portfolio, GunsGermsAndSteel] #[upper, lower]
tags: [unity, c#, addressable] #must be lower case
math: true
mermaid: true
---

## 9. 사운드
원래 소리없이 프로젝트를 마무리하려 했으나, 시연 녹화를 하다가 너무 밋밋해서 추가하게 되었다.
Sound Pool과 Unity에서 제공하는 AudioSource를 사용하였다.
<br>


### 1. Unity에서의 Sound
본격적으로 사운드를 추가하기 전에 Unity에서 사운드를 활용하는 방법을 알아보자.
[링크](https://docs.unity3d.com/kr/2023.2/Manual/AudioOverview.html)의 내용을 요약하면 다음과 같다
> * Audio Source 에서 소리를 재생
> * Audio Listener 에서 소리를 포착
> * Audio Effects/Mixer, Reverb Zones를 통해서 추가 효과를 줄 수 있음

<br>

#### 1) Audio Listener
**"오디오 리스너(Audio Listener) 는 마이크 같은 디바이스 역할을 합니다"**
설명 그대로 소리를 포착하는 역할.
씬에서 하나만 존재할 수 있으며 보통은 메인 카메라에 존재한다.

#### 2) Audio Source
**"오디오 소스(Audio Source) 는 씬에서 오디오 클립을 재생합니다. "**
**"오디오 소스는 모든 종류의 오디오 클립을 재생할 수 있으며, 2D나 3D로, 또는 혼합(스페이셜 블렌드)하여 재생하도록 설정할 수 있습니다."**

오디오 소스의 프로퍼티를 조절해서 효과를 주거나, 설정을 할 수 있다.
주요 프로퍼티는 다음과 같다.
> Audio Clip : 재생될 사운드 클립에 대한 레퍼런스입니다.
> Loop : 옵션을 활성화하면 재생이 끝날 때 Audio Clip 루프가 생성됩니다.
> Volume : Audio Listener 로부터 1월드 유닛(1미터) 거리에서 소리가 얼마나 크게 들리는지를 정의합니다.
> Spatial Blend	3D : 엔진이 오디오 소스에 미치는 효과의 정도를 설정합니다.
 &emsp; 2D의 경우 이 값을 0으로 설정 (모든 곳에서 동일하게 들림)

<br>

### 2. 게임에 사운드 추가하기
위의 정보를 종합해보면
1. 소리를 재생하고픈 오브젝트에 오디오 소스 컴포넌트를 붙인다
2. 소리를 재생한다.
3. 오디오 리스너에서 소리를 포착하여 시뮬레이션 한 뒤 컴퓨터에 출력한다.

1번에서 문제가 발생한다. 각 오브젝트에는 오디오 소스는 하나의 소리를 재생하기에 동시에 여러 소리를 재생할 수 없다
어떤 몬스터가 A 사운드를 항상 재생해야하고 가끔가다 B사운드를 낸다고 하면, B사운드를 재생할 때 기존에 A사운드가 사라진다.

지금부터는 이러한 문제를 해결하기위한 설계를 해보자.

#### 1) 설계
게임에 사용될 사운드를 크게 2가지로 분류해보자.

>1. 단발성 사운드 (각종 효과음)
&emsp; - 반복되는 소리를내는 오브젝트도 있지만, 다음 방으로 넘어가면 꺼지기 때문에 단발성으로 취급
2. 지속성 사운드 (BGM)
&emsp; - 스테이지 BGM, Boss Fight BGM, Scene BGM

단발성 사운드의 경우 한 오브젝트가 여러 소리를 동시에 재생할 수 있도록 해야한다.
 &emsp;- 하지만 오디오 소스는 하나의 클립만 재생가능
 &emsp;- 다른 오디오 소스에서 클립을 재생하면 가능
 &emsp;- 소리를 전담하는 오브젝트를 만들자 -> 많은 오브젝트를 인스턴스화 하는것은 부담 ->**오디오 소스 풀을 만들자**.
 
지속성 사운드
 &emsp;- 변환 주기가 매우 길기 때문에, 오브젝트의 컴포넌트에서 직접 재생한다.

<br>

위의 상항을 종합해서 설계를 해보면
> * 단발성 효과음을 위한 오브젝트풀을 만든다.
  * 오디오 소스를 가진 오브젝트를 여러개 만들어서 Queue로 관리하자.
  * 사용할 때 큐에서 뽑아오고 사용이 끝나면 큐에 반납 
> * 지속성 사운드는 풀과 관계없이 직접 재생


<br>


#### 2) 구현

SoundManager.cs
```csharp
using System.Collections.Generic;
using UnityEngine;

public class SoundManager
{

    private int _initialPoolsize;
    private Queue<GameObject> _sfxPool;
    private GameObject _parent;

    public void Init()
    {
        // GameScene으로 넘어오면서 @SoundPool 오브젝트가 DontDestroyOnLoad에 생성됨
        _parent = GameObject.Find("@SoundPool");
        _initialPoolsize = 30;
        _sfxPool = new Queue<GameObject>();
        InitPool();
    }

    public void InitPool()
    {
        if (_sfxPool.Count >= _initialPoolsize) return;

        // Pool에 미리 생성
        for (int i = 0; i < _initialPoolsize; i++)
            AddNewSFXSource();
    }

    public void AddNewSFXSource()
    {
        //오브젝트 인스턴스화 후에 큐에 집어넣음
        GameObject sfxobj = Managers.Resource.Instantiate("sfx",_parent.transform);
        ReturnSFXToPool(sfxobj);
    }

    public void ReturnSFXToPool(GameObject go)
    {
        // 큐에 넣을때는 active false로
        if (go == null) return;
        if (go.activeSelf == false) return;
        _sfxPool.Enqueue(go);
        go.SetActive(false);
    }

    public GameObject GetSFXFromPool()
    {
        // 큐에 남아있지 않으면 추가
        if (_sfxPool.Count == 0)
            AddNewSFXSource();

        // 큐에서 뽑아내서 반환
        GameObject sfx = _sfxPool.Dequeue();
        sfx.SetActive(true);

        return sfx;
    }

    // 오디오 클립 재생
    public GameObject PlaySFX(AudioClip clip, float volume = 1f, bool isLoop = false)
    {
        GameObject sfx = GetSFXFromPool();
        sfx.GetComponent<SFXSource>().Play(clip, volume, isLoop);
        return sfx;
    }

    // 게임 초기화등에 사용
    public void ReturnSFXToPoolAll()
    {
        foreach (Transform childTransform in _parent.transform)
        {
            if (_sfxPool.Count <= _initialPoolsize)
                ReturnSFXToPool(childTransform.gameObject);
            else 
                GameObject.Destroy(childTransform.gameObject);
        }
    }
}
```

SFXSource.cs
```csharp
using System.Collections;
using UnityEngine;

public class SFXSource : MonoBehaviour
{
    private AudioSource _audioSource;
    private Coroutine _coroutine;

    private void Awake()
    {
        _audioSource = GetComponent<AudioSource>();
        _audioSource.playOnAwake = false;
        // 2D 라서 0
        _audioSource.spatialBlend = 0f;
    }

    public void Play(AudioClip clip, float volume = 1f, bool isLoop = false)
    {
        StopAllCoroutines();

        _audioSource.clip = clip;
        _audioSource.volume = volume;
        _audioSource.loop = isLoop;
        _audioSource.Play();

        if (!isLoop)
            _coroutine = StartCoroutine(CDisableAfterPlay());
    }

    private IEnumerator CDisableAfterPlay()
    {
        yield return new WaitForSeconds(_audioSource.clip.length + 0.1f);
        Managers.Sound.ReturnSFXToPool(gameObject);
    }
}

  
소리를 재생할 때는 SoundManager의 Play 함수를 이용한다.
재생이 끝나면 자동으로 풀에 반납됨.
**ReturnSFXToPool에는 오브젝트 삭제를 넣지 않음**
  &emsp;- 한번 풀의 사이즈를 초과해서 SFX오브젝트를 생성한다는건 앞으로도 많이 필요하다고 볼 수 있음
  &emsp;&emsp;- 게임이 진행되면서 사운드 수요가 자연스레 많아짐 사이즈에 제한을 두면 불필요한 자원 사용