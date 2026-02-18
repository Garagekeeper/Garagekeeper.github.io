---
title: " Frustum Culling "
date: 2026-02-18 18:43:01 +0900
description: ""
categories: [Computer Science, Graphics] #[upper, lower]
tags: [graphics] #must be lower
math: true
mermaid: true
---

# 1.절두체 (Frustum)
위키에 따르면 절두체의 정의는 다음과 같다<br>
기하학에서 [절두체 (Frustum)](https://en.wikipedia.org/wiki/Frustum)는 입체(보통 원뿔이나 각뿔)를 절단하는 하나나 두 평행면 사이의 부분이다.
뭔가 직관적이지 않은 느낌이라면 이름을 그대로 해석하면 된다. 절두체 즉, '머리가 잘린 입체'이다. 아래의 그림까지 살펴보면 확실한 느낌이 온다.

![](https://upload.wikimedia.org/wikipedia/commons/thumb/c/cc/Square_frustum.svg/250px-Square_frustum.svg.png)

<br>

## View Frustum
> In 3D computer graphics, a viewing frustum or view frustum is the region of space in the modeled world that may appear on the screen; it is the field of view of a perspective virtual camera system. [(출처)](https://en.wikipedia.org/wiki/Viewing_frustum)
>
> 'View Frustum은 화면에 표시될 수 있는 모델링된 세계의 공간영역' 이라는 복잡한 설명이 있지만, 간단히 우리가 게임에서 카메라로 볼 수 있는 영역이라고 생각하면 될 거 같다.

<br>

![](https://upload.wikimedia.org/wikipedia/commons/thumb/0/02/ViewFrustum.svg/250px-ViewFrustum.svg.png)

* 총 6개의 평면으로 구성되어 있다.
  * Near, Far
  * Left, Right
  * Top, Down
* 이 절두체 안으로 들어온 오브젝트는 카메라에 표시된다.


# 2. 절두체 컬링
## 개념
![](https://bruop.github.io/static/9b99c8c3b3cc8dda378853f26d2b0a9d/eefce/RTR4.19.09.png)
<br>
위 그림은 View Frustum을 옆에서 바라본 것이고 뷰 프러스텀 밖에 있는 오브젝트들 (점선 표시)는 화면에 표시 되지 않는다. 어차피 표시되지 않는 오브젝트를 매 프레임 3D공간에 그리는 것은 비효율적이다. 그래서 아예 그리지 않을 오브젝트를 선별하는 것을 Culling이라고 하며, View Frustim을 기반으로 행하는 것을 Frustum Culling이라고 한다.<br>
(화면에 표시된 back face, occlusion, distance등은 다른 게시물에서 다루겠다.) 

## 원리
Frustum Culling을 간단하게 이해하기 위해서 오브젝트가 점이라고 가정하자. 어떤 오브젝트가 절두체 안에 들어있는지 판별하려면 어떤 방법을 사용해야할까.

### 벡터와 Culling
* 내적 : 두 벡터의 내적값을 통해서 벡터가 같은 방향을 가리키는지 확인할 수 있다.
* 외적 : 두 벡터를 외적하면 두 벡터와 직교 하면서 두 벡터로 만들어진 평행사변형의 넓이 만큼의 벡터가 생성된다. (방향 존재)

이를 이용해서 어떤 오브젝트가 절두체 안에 있는지 확인해보자 벡터의 회전이 들어가는 경우 처음에 보면 난해하다고 생각해서 **절두체가 직사각형**이고 **회전을 하지 않은** 경우를 확인해보자
* 3가지 단위벡터 forward, up, right (카메라 기준) 있다고 가정
* 절두체를 이루는 6개의 평면의 법선 벡터가 절두체 내부를 향하도록 설정
  * Left p : forward $\times$ up
  * Right p : up $\times$ forward
  * Up p : forward $\times$ right
  * Down p : right $\times$ forward
  * Far p  : $(0,0,-1)$
  * Near p : $(0,0,1)$
* 위의 법선 벡터들과 판별할 오브젝트의 위치벡터를 내적
  * 모든 값이 0보다 크거나 같으면 절두체 내부에 있음.
  * 하나라도 음수면 절두체 밖에 있음.

위의 내용이 절두체 컬링의 핵심이다. 저기서 좌우평면을 $\theta$, 상하평면을$\phi$ 만큼 회전시키고 카메라의 회전 값 (roll, pitch, yaw)을 기반으로 변환해서 판별하면 된다,
 
<br>

## 바운딩 볼륨
지금 까지는 오브젝트가 점인 경우였지만, 대부분의 경우 오브젝트는 각자의 형태를 가지기 마련이다.
오브젝트들의 모든 점을 검사하면 배보다 배꼽이 큰 경우가 생길 것이다. 이를 해결할 방법을 알아보자.

### 바운딩 스피어
* 오브젝트를 완전히 감싸는 가장 작은 구. 중심과 반지름만 알면 되니까 아주 직관적
* 물체가 회전해도 다시 계산할 필요없음.
* 길쭉한 물체에 대해서는 컬링 정확도 떨어짐

### AABB (Axis Aligned Bounding Box)
* 축에 평행한 면들로 만들어진 직육면체
* 오브젝트의 정점중에서 가작 Min, Max를 추출해 축에 평행한 직육면체를 만든다.
  * 정점중에서 가장 작은 x,y,z를 찾고 가장 큰 x,y,z를 찾아 이 2점으로 직육면체를 만든다
* 회전시에 다시 계산되어 비효율적일 수 있음
* 연산 속도가 빠름
* 정밀도 낮음
  * 8개의 점이 절두체 내부에 있는지 확인해야함
    * 최적화 방법 필요

### OBB (Oriented Bounding Box) 
* 방향을 가지는 직육면체
* 중심점, 축 방향, 반 폭으로 정의되는 직육면체
* 가장 근사한 상자를 만들 수 있음
* 절두체 판별 시 [분리축 정리(SAT)](https://en.wikipedia.org/wiki/Hyperplane_separation_theorem)를 통한 계산 
* 복잡하고, 많은 연산 필요 