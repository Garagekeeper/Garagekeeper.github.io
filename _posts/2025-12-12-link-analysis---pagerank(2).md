---
title: "Link Analysis - PageRank(2)"
date: 2025-12-12 18:10:43 +0900
description: "노드간의 관계를 분석하는 link analysis에 대해서 알아보자"
categories: [Computer Science, Link Analysis] #[upper, lower]
tags: [cs] #must be lower case
math: true
mermaid: true
---

## PageRank의 세 가지 의문점
$$
r=M\cdot r$$
* 위의 식이 수렴하는가?<br>
   ex) 수렴하지 않는 경우(진동)
   ![image1](https://velog.velcdn.com/images/garage_keeper/post/b94ce342-3d2c-47fe-a4d5-4e40ad3906ed/image.png)
   <br>
   $$
   r_a=1\space0\space1\space0
   $$
   <br>
   $$
   r_b=0\space1\space0\space1
   $$
   <br>
* 우리가 원하는 값으로 수렴하는가?
<br>
ex) 원하지 않는 값으로 수렴하는 경우
   ![image2](https://velog.velcdn.com/images/garage_keeper/post/9fd2718c-f0e2-4ff2-b2ca-01d3e185af3e/image.png)<br>
   $$
   r_a=1\space0\space0\space0
   $$
   <br>
   $$
   r_b=0\space1\space0\space0
   $$
   <br>
* 결과가 합리적인가


## PageRank의 문제점
* DeadEnd
  * out-going link가 없음
  * rank의 누수가 발생함
  <br>
  ![image3](https://velog.velcdn.com/images/garage_keeper/post/4cc642e3-1d20-40bc-b817-aa5dac7480ca/image.png)
 * 초기의 rank값은 각각 1/3이지만 수렴한 뒤에 값을 보면 
 <br>$r_y=0,\space r_a=0,\space m=0$이되며 
 <br>초기에 rank 값의 합이 1이었지만 0으로 변한 것을 볼 수 있다
 <br>rank값의 누수(leak out)가 발생한 것
 * 애초에 m 때문에 stochastic하지 않다.
 <br>
* Spider traps
  * 내부의 사이클에 갇힘
  * 모든 rank값을 흡수함
  * 즉 수렴하긴 하지만, 우리가 원하는 값이 아님<br>
  ![image4](https://velog.velcdn.com/images/garage_keeper/post/2ce23a37-0d03-4945-9d1e-303f418b80d1/image.png)<br>
  노드 m에 들어오면 사이클에 갇히게된다. 초기의 rank값은 각각 1/3이지만 수렴한 뒤에 값을 보면 
  <br>$r_y=0,\space r_a=0,\space m=1$이되며, m이 다른 노드들의 랭크값을 흡수한다.

* Solution: Teleport
  * 위의 상황들에 직면하면 확률에 의해 임의의 점으로 이동시킨다.
  * $\beta$의 확률로 기존 과정을 진행하고, $(1-\beta)$확률로 다른 점으로 이동한다.
    * $\beta$의 값은 0.8, 0.9를 사용한다고 알려져 있다.
  * Dead end의 경우 행렬을 column stochastic하게 만들어 줘야함
  ![image5](https://velog.velcdn.com/images/garage_keeper/post/829c53d2-ffa7-48d0-a8cc-0c0d096d01c7/image.png)
  ![image6](https://velog.velcdn.com/images/garage_keeper/post/0b1c0732-e769-46eb-b34f-d474ea172835/image.png)<center/>이 경우 행렬$M$의 m열을 수정해야한다.(0 to 1/3)<center/>

## Google's solution
* PageRank equation
$$
r_j=\sum_{i  \rightarrow j}\beta\frac{r_i}{d_i}+(1-\beta)\frac{1}{N}$$

* Google Matrix A
$$
A=\beta M+(1-\beta)\begin{bmatrix}\frac{1}{N}\end{bmatrix}_{N\times N}$$

* Google's PageRank equation
$$
r=A\cdot r$$
  
  * 이 방정식을 통해 계산을 할 때 많은 시간과 자원이 필요 (A is Dense matrix)
  
## PageRank의 효율적인 계산법
* $r=A\cdot r$를 계산하는데(특히 행렬의 곱) 충분한 메모리 자원이 있다면 좋겠지만 그렇지 않다
  * 10억개의 페이지가 있다면 행렬 A는 $10^{19}$개의 요소를 가진다. 각 요소가 차지하는 메모리 까지 계산하면 결코 작은수가 아니다.
* $A$를 정리해보자
  * 먼저 teleport 각 페이지의 rank에서 $(1-\beta)$ 만큼의 세금을 뗀 것이라 생각하고 이를 마지막에 분배
  $r=A\cdot r$
  $r_j=\sum_{i=1}^{N}[\beta M+\begin{bmatrix}\frac{1-\beta}{N}\end{bmatrix}]\cdot r_i$
  $r_j=\sum_{i=1}^{N}\beta M\cdot r_i+\begin{bmatrix}\frac{1-\beta}{N}\end{bmatrix}$
  **$r=\beta M\cdot r+[\frac{1-\beta}{N}]_N$**
  다소 복잡해 보이지만, 확실히 정돈되었다고 할 수 있다. 왜? M은 희소행렬이기에 메모리도 적게 먹고, 빠르다.
  또한 Dead end가 발생 한 경우 $1-\beta$ 보다 큰 값인 $1-S$를 더해준다. (S는 누수의 합)

* 위에서 정리한 식을 바탕으로 Page Rank를 계산하는 Cost
  * 메인 메모리가 $r^{new}$를 담기에 충분하다면
    * 디스크에서 $r^{old}$ 과 $M$을 읽어들인다.
    * 디스크에 $r^{new}$를 기록한다
    * Cost per iteration = $2\lvert r \rvert+ \lvert M \rvert$
  * 메인 메모리가 부족하다면?
    (1) $r^{new}$를 k개의 block으로 나눠서 업데이트
      * 매번 M과 $r^{old}$를 디스크에서 읽어야함
      * Cost = $k(\lvert M \rvert+ \lvert r \rvert)+\lvert r \rvert$
      
    (2) $M$을 stripe로 나눠서 업데이트
      * 각 stripe는 $r^{new}$에서 해당하는 목적지 정보만을 가지고 있음
      * $M$을 $k$번 호출하는 사태는 생기지 않지만 약간의 중복이 발생함
      * Cost = $\lvert M \rvert(1+\epsilon)+(k+1)\lvert r \rvert$
    
 
## Reference
http://mmds.org/