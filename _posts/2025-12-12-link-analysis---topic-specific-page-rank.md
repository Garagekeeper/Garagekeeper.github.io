---
title: "Link Analysis - Topic-specific Page Rank"
date: 2025-12-12 18:16:01 +0900
description: 페이지의 평판뿐 아니라, 주제와 적합한지도 고려해보자
categories: [Computer Science, Link Analysis] #[upper, lower]
tags: [cs] #must be lower case
math: true
mermaid: true
---

## Topic-Specific PageRank
* 목표: 각 페이지들의 Rank를 계산할 때 그들의 평판 뿐 아니라, 주어진 주제와 얼마나 비슷한 지 또한 포함시키는 것
* Allow search queries
* Random walker, teleport
  * 일반적인 Page Rank: 모든 페이지가 똑같은 확률을 가짐
  * Topic-specific Page-Rank:  teleport를 할 set이 정해져 있음
    * teleport할 때 S(주제와 관련있는 페이지들을 포함한 집합)에서 선택
    * 각 teleport 마다 다른 $r_s$를 얻음
    * menu, query, context등에서 얻어진 정보를 사용할 수 있음    
### 사용되는 행렬
>$$
A_{ij}=\beta M_{ij}+(1-\beta)/|S| \space\space\space\space if \space i \in S$$
>$$
A_{ij}=\beta M_{ij}+0 \space\space\space\space else$$

## 그래프에서의 Proximity 측정
* path의 길이, Network flow로 Proximity를 측정하는 것은 좋지 않다.
* 대신 SimRank 사용

### SimRank
* k-partie 그래프(k개 그룹의 그래프)의 고정된 노드에서 Random walk
* 만일 어떤 노드u를 S에 넣고 Topic Specific Rank를 진행해서 나온 결과는 u와의 similarity에 해당
  * 모든 노드에 대해서 계산 해야해서 큰 규모에는 적합하지 않다
* 결국 어느 한 노드로 teleport
>![](https://velog.velcdn.com/images/garage_keeper/post/ddd087f4-3fdb-4ce7-9661-9ea7d735acbb/image.png)<center>3Parties, 3 Types of node(Authors, Conferences, Tags)<center/>


