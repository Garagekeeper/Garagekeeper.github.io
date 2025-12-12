---
title: "The binding of Isaac 모작 (4)"
date: 2025-12-12 18:33:13 +0900
description: The binding of Isaac의 맵과 맵 생성을 구현해보자
categories: [Portfolio, GunsGermsAndSteel] #[upper, lower]
tags: [unity, c#, addressable] #must be lower case
math: true
mermaid: true
---

![](https://velog.velcdn.com/images/garage_keeper/post/c0f5c294-6a6f-4b12-985c-a27096aa2808/image.PNG)
![](https://velog.velcdn.com/images/garage_keeper/post/0376566e-f6db-4a07-aa92-74e27ed16e5d/image.PNG)

아이작의 매력은 매판 맵과 아이템이 바뀌는 로그라이크 형식이라고 생각한다.
아이작의 맵은 시드를 기반으로 로직에 따라서 생성된다. 원래는 아이템을 작성 해야하는 차례이지만
시드를 기반으로 아이템도 바뀌다보니 자연스레 시드와 관련된 맵을 먼저 작성하게 되었다.

백준 문제와 제작자의 인터뷰를 통해 수월하게 진행할 수 있었다
[제작자 인터뷰](https://www.boristhebrave.com/2020/09/12/dungeon-generation-in-binding-of-isaac/)
[아이작 관련 백준 문제](https://www.acmicpc.net/problem/30043)


## 4.시드 및 맵 생성
맵 생성의 기본 골조는 다음과 같다
1. 랜덤으로 시드가 정해진다<br>
2. 시드를 기반으로 각 스탭마다 유사 난수 생성기를 통해서 방을 생성한다<br>
   -> 이떄 유사 난수 생성기는 [선형 합동 생성기](https://ko.wikipedia.org/wiki/%EC%84%A0%ED%98%95_%ED%95%A9%EB%8F%99_%EC%83%9D%EC%84%B1%EA%B8%B0)를 사용한다<br>
3. 시드가 같다면 같은 맵이 생성 된다<br>
  -> 맵 생성 알고리즘이 잘 짜여져 있다면 동일안 시드는 동일한 횟수로 유사 난수 생성기를 호출한다.

<br>
<br>

### 1. 랜덤으로 시드가 정해진다.
시드를 랜덤으로 정하는 것 자체는 어려울 것 없다.
시드의 길이와 사용될 문자열을 정한 뒤
일반 적으로 사용되는 Random 관련 함수를 사용한다.
GameManager.cs

```cs
private string GenerateSeed()
{
    string temp = "";
    for (int i = 0; i < 9; i++)
    {
        int tempInt = UnityEngine.Random.Range(0, SeedString.Length);
        if (i == 4)
            temp += "-";
        else
            temp += SeedString[tempInt];
    }
    return temp;
}
```

  <br><br>
  
### 2.시드를 기반으로 각 스탭마다 유사 난수 생성기를 통해서 방을 생성한다 
 1 )유사 난수 생성기 
 
 $$S_{n+1} = (aS_n+c)mod\,n\\
 a = 1103515245\quad c =12345 \quad n=2^{31}
 $$
 
GameManager.cs
```cs
public void Rand()
{
    // Sn+1 = (A x Sn + C ) Mod M
    Sn = (A * Sn + C) % M;
    Rand_Cnt++;
}
``` 

이를 사용하는 코드
```cs
    public int RandInt(int left, int right)
    {
        Rand();
        return (int)((Sn % (right - left + 1)) + left);
    }

    // P보다 작을 경우 true;
    public bool Chance(int p)
    {
        return (RandInt(1, 100) <= p);
    }

    public T Choice<T>(List<T> rooms)
    {
        int cnt = rooms.Count();
        int temp = RandInt(0, cnt - 1);
        return rooms[temp];
    }
```
</br></br>

2) 방 생성
* $8 \times 8$ 사이즈의 배열을 준비한다.
* N개의 일반 방을 생성한다.
  * N은 난이도, 현재 층수를 생각해서 원하는 대로 설정
  * $(4,4)$ 위치에 시작방을 생성하고 Queue에 넣은뒤 다음을 반복한다.
  * Queue의 front에 있는 방을 대상으로 우,하,좌,상 순으로 방을 만들 수 있는지 판단한다. (BFS)
  MapManager.cs

```cs
  //방들이 최대한 사이클을 생성하지 않도록
public bool CanCreateRoom(int x, int y)
{
    if (x < 0 || x >= s_mapMaxX) return false;
    if (y < 0 || y >= s_mapMaxY) return false;
    // 이미 방이 있다면 넘어간다.
    if (s_roomGraph[x, y] == 1) return false;
    // 인접한 방이 2개 이상이면 방을 생성하지 않는다.
    if (CheckAdjacenctRoomCnt(x, y) >= 2) return false;
    // 위조건을 만족해도 50보다 작아야 방을 생성
    if (!Managers.Game.Chance(50)) return false;
    return true;
}
```
  
```cs
//1.BFS를 통한 방 생성
while (roomCnt < Managers.Game.N)
{
    roomClassQueue.Enqueue(Managers.Game.Choice(Rooms));
    while (roomClassQueue.Count != 0)
    {
        RoomClass front = roomClassQueue.Dequeue();
        //Right, Down, Left, Up
        int[] dx = { 0, 1, 0, -1 };
        int[] dy = { 1, 0, -1, 0 };
        for (int i = 0; i < 4; i++)
        {
            if (roomCnt == Managers.Game.N) break;
            int nx = front.XPos + dx[i];
            int ny = front.YPos + dy[i];
            if (CanCreateRoom(nx, ny))
            {
                rc = new RoomClass(nx, ny);
                front._adjacencentRooms[i] = rc;
                rc._adjacencentRooms[(i + 2) % 4] = front;
                s_roomGraph[nx, ny] = 1;
                Rooms.Add(rc);
                roomCnt++;
                roomClassQueue.Enqueue(rc);
                XMin = Math.Min(XMin, nx);
                YMin = Math.Min(YMin, ny);
                XMax = Math.Max(XMax, nx);
                YMax = Math.Max(YMax, ny);
            }
        }
    }
}
```
  
 * 특수방을 생성한다.
    * 게임의 기획에 따라서 생성
    * 문서 상단의 링크를 참고하면 대략적인 느낌을 알 수 있음

<br><br>

### 3.시드가 같다면 같은 맵이 생성 된다
 선형 합동 생성기는 실행 횟수에 따라서 값이 달라진다. 즉 같은 횟수에서는 같은 값이 나온다.
 알고리즘이 정확한 순서를 따르고, 외부요소의 영향이 없다면 같은 모양의 방이 생성될 것이다.
 

<br><br>

### 4. 여담
 * 2차원 배열을 기준으로 한 좌표계와 unity의 좌표계가 달라서 (y의 증가방향이 달라서) 맵이 대칭되거나 회전하는 문제
   * 사실 꿈에도 모르고 있다가 맵의 데이터를 추출해보고 알게 되었다
   * 맵의 충돌 데이터를 메모장에 출력했더니 형태는 같지만 방향이 다른 데이터가 나왔다.
   * 유니티에서 시작방의 좌표가 $(0,0)$인 것도 문제가 되었다.
   <br>
 * 방의 클리어 체크는 맵에 몬스터나 보스가 없으면 클리어로 간주한다.
 	* 모든 방의 몬스터를 미리 소환하는 방식이 아님
    * 방에 입장하면 그 방에만 해당 몬스터가 소환되는 방식
   <br>
 * 맵의 장애물이나 몬스터를 하드코딩으로 소환 하는게 아니라 여러 경우를 미리 프리팹 형태로 만들고 Rand()를 통해 선택한다.
   * 실제 아이작 에디터를 살펴보면 여러 유형의 방이 미리 형태가 정해져 있다.
