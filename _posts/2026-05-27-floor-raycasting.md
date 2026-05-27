---
title: "Floor RayCasting"
date: 2026-05-27 23:49:46 +0900
description: ""
categories: [Computer Science, Graphics] #[upper, lower]
tags: [graphics, raycast] #must be lower
math: true
mermaid: true
---
<br>

## Wall casting vs Floor(Ceiling) casting
Wall castring은 지난 게시물에서 살펴본 내용이다. 시야각 범위의 x를 순회하면서 ray를 쏴 벽과의 거리를 얻어오고, 그 거리에 반비례하게 세로선을 그린다.

Floor Casting은 조금 다르다. 
* 화면의 중앙부터 맨 밑까지 Y축을 순회
  * Y와 카메라의 Y의 차이를 이용해 현재 카메라와 YRow의 수평거리를 구한다
* 위에서 계산된 거리를 기반으로, 시야각의 왼쪽과 오른쪽을 투영
  * 이 양쪽 좌표 사이를 화면의 너비로 나눠서 X(한 픽셀)만큼 움직일 때 실제 벡터는 얼마나 움직이는지 계산(Step계산)
  * 왼쪽 픽셀부터 오른쪽 픽셀까지 순회하며 Step을 누적시키면 별도의 연산 없이 각 픽셀이 위치한 월드맵 좌표를 알 수 있다.

말로하니 조금 어렵다. 코드와 그림을 살펴보며 자세히 알아보자

## 카메라와 YRoW 간의 수평거리(비율)
```
0 (Y)			
|			------------------------------------
|	       /
|		  /
|		 /
|		/
|	 Cam  < - Height / 2;
|	    \
|		 \
|		  \
|		   \
▼		    ------------------------------------------ 
Height		Floor
```
위 그림에은 카메라를 측면에서 바라본 것을 형상화한 것이다. 먼저, 카메라는 화면의 정중앙에 위치한다.

카메라에서 바닥으로 쏘는 레이가 보이는데 이 부분만 잠깐 살펴보자
```

┌── dist ──┐
 ---------
 \       |   ┐
  \      |   │
   \     |   │
    \    | Height / 2 (Y=Height) : DeltaY
     \   |   │
      \  |   │
       \ |   ┘

```
여기서 Y = Height, 카메라와의 거리는 Height / 2가 되고, 카메라와 Floor까지의 수직거리는 dist이다. 

Y값이 줄어듦에 따라서 dist는 증가하고 Y는 줄어든다.


Y가 커지면 (카메라와의 거리가 멀어지면) dist값은 줄어드는 반비례 관계를 가진다.

```
CameraY = HEIGHT/2;
DeltaY = Y - HEIGHT/2;
```
여기서 Y가 Height일때의 dist값을 1이라고 가정하면. DeltaY일때 수직거리는 다음과 같다.
$$
1 : CameraY = \frac{1}{x} : DeltaY
$$

이 식을 정리하면 다음과 같다.
$$
x = \frac{DeltaY}{CameraY}
$$

이를 통해서 현재Y의 바닥과 카메라의 수직거리를 알 수 있게된다.

<br>

## 시야각의 왼쪽과 오른쪽을 투영

위에서 사용한 그림을 살짝 변형해서 보자
```
0 (X)			
|			------------------------------------
|	       /      |
|		  /       |
|		 /|       |
|		/ |       |  
|	 Cam  ▼       ▼
|	    \
|		 \
|		  \
|		   \
▼		    ------------------------------------------ 
Width		
````

이 그림은 카메라를 위에서 바라봤을때의 모습이다. 화살표를 보면 카메라에 가까운 화살표는 카메라가 왼쪽 끝에서 중앙까지 오는동안 2칸 움직였다. 그런데 오른쪽 화살표는 같은 조건에서 4칸 이동했다. 카메라에 가까우면 조금 이동하고 멀면 많이 이동한다. 여기서 앞서 구한 카메라와 바닥의 수평거리가 이용된다. x가 한칸 이동할 때 이 값을 곱해주면 정확한 위치가 나온다 (그림의 화살표 끝부분)

x가 움직일때마다 이 값을 누적시키면 해당 지점이 월드좌표의 어떤 곳인지 쉽게 알 수 있다.
이를 통해 단순히 바닥을 찍는 것 뿐 아니라 체크무늬등 여러 것들이 가능해진다


### 버퍼에 미리 바닥을 찍는것과의 차이
버퍼에 미리 바닥을 찍고 벽을 그리면 얼추 느낌이 나지만 정적이고, 체크무늬 등 여러 효과를 적용할 수 없다.

## 구현 결과
![ProfilerExample](/assets/img/FloorCast1.png)
![ProfilerExample](/assets/img/FloorCast2.png)
![ProfilerExample](/assets/img/FloorCast3.png)

## 구현 코드

```cpp
void DrawFloor()
{
	// 벽의 레이캐스팅은 X를 돌면서 벽에 ray를 쏴서 거리를 뽑아오고 그 거리에 비례해서 세로선을 그림
	// 바닥의 레이캐스트는 현재 row(y)과 카메라 사이의 거리비를 구한다.(-1 < dist < 1)
	//		- 최대 거리는 화면 정중앙	(height / 2)
	//		- 최소는 화면 최하단 (플레이어의 발 바로 앞) (height)
	//		- 이 값을 기준으로 현재 값을 뽑아 낼 수 있음
	//		- current Dist = (0.5 * Height) / (y - Height / 2) (-1 =< Dist <= 1)
	// 해당 거리를 통해서 실제 그 거리를 가진 맵 위의 점을 찾고 
	const int WIDTH = GScreen.HorSize;
	const int HEIGHT = GScreen.VerSize;

	for (int Y = HEIGHT / 2; Y < HEIGHT; Y++)
	{
		// 플레이어 시야의 왼쪽 끝
		double Ray_DirLeftEndX	= Player.DirX - Player.PlaneX;
		double Ray_DirLeftEndY	= Player.DirY - Player.PlaneY;

		// 플레이어 시야의 오른쪽 끝
		double Ray_DirRightEndX = Player.DirX + Player.PlaneX;
		double Ray_DirRightEndY = Player.DirY + Player.PlaneY;



		/*
		0			
		|			------------------------------------
		|	       /
		|		  /
		|		 /
		|		/
		|	Cam  < - Height / 2;
		|		\
		|		 \
		|		  \
		|		   \
		▼			------------------------------------- 
		Height		Floor

		높이가 Height / 2일때 거리 = 1 (무한)
		그렇다면 CurrentY일때 거리는 = 1/x (반비례 관계라서)
		1 : posZ = 1/x : p (반비례 관계니깐) 
		posZ/x = p;
		posZ/p = x;
		----
		\   | -
		 \  | CurrentY
		  \ | -
		   -
		*/

		// 현재 카메라 평면의 Y좌표
		double CameraY = 0.5 * HEIGHT;
		// 카메라 평면을 기준으로 현재 Y 좌표가 얼마나 떨어져 있는지
		int CurrentY = Y - HEIGHT/2;
		// 카메라 평면에서 현재 Y(row) 까지의 거리
		// Y가 Height/2일 때 무한(1) (가로 길이는 0)
		// Y가 Height일 때 1(0) (가로 길이는 width)
		//double HorDistFromCamToFloorRatio = CameraY / CurrentY;
		double HorDistFromCamToFloorRatio = CameraY / CurrentY;

		// x=0에서 x=width 로 한칸 이동할 때마다 X와 Y의 이동량 (delta)
		double FloorStepX = HorDistFromCamToFloorRatio 
        * (Ray_DirRightEndX - Ray_DirLeftEndX) / WIDTH;
		double FloorStepY = HorDistFromCamToFloorRatio 
        * (Ray_DirRightEndY - Ray_DirLeftEndY) / WIDTH;

		// 시야각의 맨 왼쪽 방향의 방향 벡터쪽의 바닥(현재 시야각에서 가장 왼쪽 바닥)
		double FloorX = Player.X + HorDistFromCamToFloorRatio * Ray_DirLeftEndX;
		double FloorY = Player.Y + HorDistFromCamToFloorRatio * Ray_DirLeftEndY;

		for (int X = 0; X < WIDTH; X++)
		{
			// 바닥 좌표의 정수부만 취함 (월드 좌표, 월드 내부에서 벡터가 가리키는 좌표)
			int CellX = static_cast<int>(FloorX);
			int CellY = static_cast<int>(FloorY);

			// 결국 화면의 x,y가 Map의 Cellx, Celly의 지점을 그리고 있다 라고 해석이 가능
			// 그래서 아래와 같이 체크무늬 모양도 가능

			// 실제 출력할 좌표
			int checkerBoard = (std::abs(CellX) + std::abs(CellY)) % 2;
			wchar_t tileChar = (checkerBoard == 0) ? L'·' : L' ';
			GScreen.PrintChar(tileChar, X, Y);

			FloorX += FloorStepX;
			FloorY += FloorStepY;
		}
	}
}
```