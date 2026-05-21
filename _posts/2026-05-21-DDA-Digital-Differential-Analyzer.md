---
title: "DDA와 RayCast"
date: 2026-05-21 15:18:44 +0900
description: ""
categories: [Computer Science, Graphics] #[upper, lower]
tags: [graphics] #must be lower
math: true
mermaid: true
---

# Raycasting
Raycasting은 용어 그대로 Ray(광선)를 Cast(쏘다)해서 ray에 닿는 표면을 파악하는 기술이다.

Unity등에서 플레이어 밑에 바닥이 있는지 등을 확인하는데 사용하곤 했는데 랜더링 분야에서도 사용되었다고한다. 가장 유명한 예시는 이드 소프트웨어의 울펜슈타인 3D이다. 3D는 상상도 못하던 시절에 이 기법을 사용해 3D같은 느낌을 주었다고 한다.

![](https://upload.wikimedia.org/wikipedia/commons/thumb/e/e7/Simple_raycasting_with_fisheye_correction.gif/250px-Simple_raycasting_with_fisheye_correction.gif)

위 그림을 보면 이해가 쉽다.
오른쪽 미니맵을 보면 단순한 2차원 격자 구조 이지만, 왼쪽의 화면은 원근감이 적용된 사실적인 화면이다.

간략하게 동작하는 원리를 살펴보자
* 벽을 찾는다
* 벽 까지의 거리를 구한다.
* 벽을 출력한다 
  * 이 때 벽은 세로선으로 출력한다,
  * 가까운 벽은 길게 그리고 멀리 있는 벽은 짧게 그린다.
  * 정면과, 옆면의 출력을 다르게 해서 음영을 준다

사실 벽을 찾고 거리만 구하면 비교적 쉬운 내용이다. DDA알고리즘을 통해서 벽을 찾는 과정을 알아보자

<br>

앞으로 기술되는 내용은 [튜토리얼1](https://lodev.org/cgtutor/raycasting.html)과 [튜토리얼2](https://permadi.com/1996/05/ray-casting-tutorial-8/)을 참고했다.


## [DDA(Digital Differential Analyzer)](https://en.wikipedia.org/wiki/Digital_differential_analyzer_(graphics_algorithm))


DDA알고리즘은 원래 그래픽스에서 선을 긋는 용도로 사용되는 알고리즘이다. 주로 래스터라이징에 사용되는데, 2차원 격자에서 두 점을 잇는 직선을 긋는데 사용된다.

![](https://upload.wikimedia.org/wikipedia/commons/thumb/a/ab/Bresenham.svg/330px-Bresenham.svg.png)

위 그림을 통해서 설명하자면 선이 시작하는 지점과 선이 끝나는 점 사이의 격자들에 색칠을 하는 과정이라고 볼 수 있다. (이를 [래스터라이징](https://ko.wikipedia.org/wiki/%EB%9E%98%EC%8A%A4%ED%84%B0%ED%99%94)이라 한다);

이 방식을 응용해서 벽을 찾는 과정을 알아보자.



## 기본 개념



![](https://lodev.org/cgtutor/images/raycastgrid.gif)

초록색 플레이어가 앞에 벽이 존재한다는 걸 어떻게 알아낼까? 가장 간단한 방법은 플레이어의 시선 방향으로 앞으로 나아가면서 해당 지점에 벽이 있는지 확인하는 것이다.

<br>

![](https://lodev.org/cgtutor/images/raycastmiss.gif)

일정한 크기로 벡터를 더해서(확장해서) 체크를 하면 위 그림처럼 누락된 부분이 생긴다.

<br>

![](https://lodev.org/cgtutor/images/raycastmiss2.gif)

그렇다고 해서 너무 자주 검사하면 성능이 떨어진다.

<br>

![](https://lodev.org/cgtutor/images/raycasthit.gif)

그렇다면, 광선이 x에 수직인 직선 (x = 상수) 혹은 y에 수직인 직선 (y = 상수)에 만날때만 검사하면 어떨까? 검사횟수는 떨어지고, 누락은 없어진다.
기울기를 통해서 방향을 구하고 x=1체크 y=1체크 x=2체크 y=2체크 이런식으로 가도 되지만, DDA는 더욱 효과적인 해법을 제시한다.

<br>

## 카메라 평면
![](https://lodev.org/cgtutor/images/raycastingcamera.gif)

기본적인 각도, 라디안 등을 사용해서 할 수 있지만 여기서는 카메라 평면이라는 개념을 도입한다. 말 그대로 이 평면 앞에 있어야 볼 수 있는데 왜 사용된 것일까?

* 카메라 평면을 벡터로 표시하면 카메라의 시야각을 조절할 수 있다.

카메라 평면은 플레이어의 방향벡터에 수직인 벡터이다. 위 그림에서 좌우로 그러진 파란 선이 길어지면 빨간 선의 범위도 자연스레 늘어난다. 여기서 빨간선이 우리가 쏜 ray가 되고, 빨간 선의 범위가 곧 시야각이다.

* 어안렌즈 효과를 방지한다.

플레이어부터 벽까지의 거리가 아니라 카메라 평면까지의 수직 거리를 구해서 어안렌즈 효과를 방지할 수 있다고 한다.

여기서 주의할 점은 플레이어를 회전 시키면 반드시! 카메라 평면도 회전 시켜야 한다.

## 진행과정
진짜진짜 간단히 설명하면 다음과 같다.

* 진행은 1칸 단위로 한다(대각선 안됨)
* 다음 경계까지 가까운 칸으로 이동
* 해당 격자에 벽이 있으면 직전 칸에서 종료

여기서 포인트는 다음 경계까지 가까운 칸으로 이동이다.

![](https://lodev.org/cgtutor/images/raycastdelta.gif)

SideDist는 현재 위치에서 경계 (수직선(X), 수평선(Y)) 까지의 거리이다.
DeltaDist는 경계에서 그 다음 경계까지의 거리이다.
여기서 deltaDIstX를 피타고라스로 구할 수 있는데, x가 1이고 방향 벡터가 있으니 y증가분 (기울기)가 세로 성분의 기울기이다.

```deltaDistX^2 = 1^2 + (rayDirY * rayDirY) / (rayDirX * rayDirX))```
```deltaDistY^2 = 1^2 + (rayDirX * rayDirX) / (rayDirY * rayDirY))```

```deltaDistX = abs(1 / rayDirX)```
```deltaDistY = abs(1 / rayDirY)```

튜토리얼에서는 더욱 간소화해서 위와 같은 형태가 된다. 실제 거리가 아니라 두 값의 비율이 영향을 주기 때문에 `abs(|ray| / rayDirX)`가 `abs( 1 / rayDirX)` 로 간단하게 변환된 형태다.

ray는 sideDist가 작은(경계까지 거리가 짧은 것) 방향으로 이동한다
그런다음 sideDist를 갱신 시킨다 (deltaDist를 더함).
위 그림을 통해서 예를 들면 처음에는 sideDistY가 더 작기 때문에 y 방향으로 이동하고, deltaDistY를 더한다 (여기서 deltaDistY는 y = 1 직선을 만난지점에서 y = 2인 직언을 만난 지점까지의 거리) 

이제 sideDistX는 출발점에서 x = 1경게와 만나는 지점의 거리 (그림의 sideDistX), sideDistY는 출발점에서 y = 2 경계와 만나는 지점까지의 거리이다.
여기서 sideDistX가 더 작기 때문에 x 방향으로 이동한다. 같은 방식으로 빨간 ray가 그려진다.


벽을 찾았다면 거리를 구하는데 약간의 마법을 쓴다.

![](https://lodev.org/cgtutor/images/raycastperpwalldist2.png)

```deltaDistX = abs(1 / rayDirX)```
```deltaDistY = abs(1 / rayDirY)```

앞에서 이렇게 간소화 했던것이 유클리드 거리를 구해서 코사인 값을 취하는 것 보다 훨씬 간단해진다.
방향벡터 = (rayDirX, rayDirY) 인데 이것은 X 방향으로  rayDirX 만큼 이동하면 Y방향으로 rayDirY만큼 이동한다는 의미로 볼 수 있는데 

X 방향으로 1만큼 이동한다면? 수직방향으로 1 / rayDirX 만큼 이동하게 된다. X방향으로 움직일 때 마다 수직 누적이 생기면서 sideDistX가 수직 거리값을 누적 시키고 있었던 것. 

```cpp
if (side == 0) perpWallDist = (sideDistX - deltaDistX);
else           perpWallDist = (sideDistY - deltaDistY);
```
따라서 sideDistX는 X축 방향으로 이동하면서 생긴 수직 거리의 누적이다. 다만 여기서 벽이 감지된 위치의 격자 내부에 ray가 위치 하기 때문에 한 칸 뒤로 물러난 것이 실제 거리가 된다. 


그렇다면 이제 거리를 알아냈으니 가까운것은 길게, 먼것은 짧게 그리면 된다.

```cpp

      //Calculate height of line to draw on screen
      int lineHeight = (int)(h / perpWallDist);

      //calculate lowest and highest pixel to fill in current stripe
      int drawStart = -lineHeight / 2 + h / 2;
      if(drawStart < 0)drawStart = 0;
      int drawEnd = lineHeight / 2 + h / 2;
      if(drawEnd >= h)drawEnd = h - 1;
```

![ProfilerExample](/assets/img/RayCastSample1.png)
![ProfilerExample](/assets/img/RayCastSample2.png)
![ProfilerExample](/assets/img/RayCastSample3.png)

## 구현 코드
```cpp
void DrawGrid()
{
	const int WIDTH		= GScreen.HorSize;
	const int HEIGHT	= GScreen.VerSize;
	/*
	y
		\	     /
		 \ ____ /
		  \    /
		   \  /
							x
	여기서 카메라 플레인 (가로선)의 x범위를 -1 ~ 1로 만들어준다.
	dir + Plane에 이 감소된 범위를 곱해서 방향을 결정해준다
	음수는 카메라의 왼쪽, 양수는 카메라의 오른쪽, 0은 정 중앙
	*/
	for (int x = 0; x < WIDTH; x++)
	{
		double CameraX = 2 * x / double(WIDTH) - 1;
		double RayDirX = Player.DirX + Player.PlaneX * CameraX;
		double RayDirY = Player.DirY + Player.PlaneY * CameraX;

		// 현재 우리가 서 있는 위치
		int MapPosX = (int)Player.X;
		int MapPosY = (int)Player.Y;

		// ray가 출발해서 처음으로 x에 수직인 선을 만난 위치까지의 거리
		// ray가 출발해서 처음으로 y에 수직은 선을 만난 위치까지의 거리
		double SideDistX;
		double SideDistY;

		// ray가 그 다음으로 x축에 수직인 선을 만났을 때 처음 만났던점에서 지금까지의 거리
		// ray가 그 다음으로 y축에 수직인 선을 만났을 때 처음 만났던점에서 지금까지의 거리

		// 그림으로 보면 직각 삼각형 형태로 피타고라스로 계산할 수있다.
		// deltaDistX = sqrt(1 + (rayDirY * rayDirY) / (rayDirX * rayDirX))
		// deltaDistY = sqrt(1 + (rayDirX * rayDirX) / (rayDirY * rayDirY))

		// 이를 단순화하면 (계산하고 정리하면 이렇게 됨)
		// deltaDistX = abs(|rayDir|(길이) / rayDirX)
		// deltaDistY = abs(|rayDir|(길이) / rayDirY)

		// 여기서 한 술 더 떠서 |rayDir|을 1로 게산해 버리는데 DDA알고리즘에서
		// 길이가 중요한게 아니라 비율이 중요한거라 둘다 길이로 나눠버린다고 한다.
		// 0으로 나눌 수는 없으니까 큰 값을 넣어서 사실상 0으로 만든다.
		double DeltaDistX = (RayDirX == 0) ? 1e30 : std::abs(1 / RayDirX);
		double DeltaDistY = (RayDirY == 0) ? 1e30 : std::abs(1 / RayDirY);
		// 나중에 Ray의 거리를 구하는데 사용
		double PerpWallDist;

		//DDA는 반복할 때마다 맵을 정확히 한칸씩 이동하는데 (대각선 안됨)
		//한칸 넘었을 떄 X에 닿았는지 Y에 닿았는지 둘 중 하나만 먼저 발생
		//이동 방향은 ray의 방향에따라서 결정되고, 그방향을 여기에 저장한다.
		int StepX;
		int StepY;

		// 벽에 부딪혔나?
		bool bHit = false;
		// X에 수직선에 감지? Y수직선에 감지?
		// X쪽이면 0, Y쪽이면 1
		int Side;

		//초기 sideDist 계산
		//현재 위치에서 가장 가까운 다음 격자선까지 raY가 얼마나 가야하는가
		if (RayDirX < 0)
		{
			StepX = -1;
			// 왼쪽 방향으로 가면
			// 처음 x를 만날때 까지의 실제 거리 * deltaDistX 인데
			// deltaDistX는 다음과 같다 x가 +- 1증가할 때 ray의 전체 길이는 얼마나 증가했나?
			// 원래는 그런데 길이를 나눠나서 모호하게 보일 수 있음
			SideDistX = (Player.X - MapPosX) * DeltaDistX;
		}
		else
		{
			StepX = 1;
			SideDistX = (MapPosX + 1.0 - Player.X) * DeltaDistX;
		}

		if (RayDirY < 0)
		{
			StepY = -1;
			SideDistY = (Player.Y - MapPosY) * DeltaDistY;
		}
		else
		{
			StepY = 1;
			SideDistY = (MapPosY + 1.0 - Player.Y) * DeltaDistY;
		}

		// 진짜 DDA시작
		// 매 루프마다 격자 1칸을 이동(충돌할 때 까지)
		while (bHit == false)
		{
			//다음 격자로 이동, x나 y 방향으로
			if (SideDistX < SideDistY)
			{
				// 직전에 x를 먼저 만났으면 x 쪽으로 이동
				// 직전에 Y를 먼저 만났으면 Y 쪽으로 이동

				// ray 이동
				SideDistX += DeltaDistX;
				// 격자 이동
				MapPosX += StepX;
				// 직전에 x를 먼저 만나서 0으로 표시
				Side = 0;
			}
			else
			{
				SideDistY += DeltaDistY;
				MapPosY += StepY;
				Side = 1;
			}

			// 해당 격자에 벽이 있는지 확인, 있으면 종료
			if (WorldMap[MapPosY][MapPosX] == WALL) bHit = true;
		}

		// DDA가 끝나면 실제 Ray의 거리를 계산
		// 유클리드 효과거리를 사용하면 어안효과고 있다고 하는데.. 잘 몰루
		//		플레이어의 위치를 기준으로 유클리드 하면
		//		ㅁㅁㅁ
		//        p
		//		이 상황에서 p에서 각 벽까지의 거리가 다 달라서 중간게 제일 커보이고
		//		나머지는 작아보임 이러면 어안처럼 둥글게 보인다
		// 카메라 평면을 사용하면 
		//      ㅁㅁㅁ
		//      
		//      ------- 
		// 평면에서 벽까지의 수직거리를 측정해서 어느 점에서나 같은 값이 나온다.
		// 그래서 어안효과 사라짐.
		// perpWallDist이 값이 벽에 수직인 거리를 말하는 것

		// 벽을 발견했다? -> 벽에 들어와있다(한칸 더 갔다)
		// 한칸 뒤로가자 (ray도 온 만큼 뒤돌아가자)
		// 근데 왜 한칸 뒤로 가는게 실제 수직 거리이죠?
		//		아까 double deltaDistX = (rayDirX == 0) ? 1e30 : std::abs(1 / rayDirX);
		//		여기서 길이를 1로 바꿔버려서 대각선 성분이 없어지고 수직 성분만 남았다고!
		//		(솔직히 좀 놀라움)

		if (Side == 0)
			PerpWallDist = (SideDistX - DeltaDistX);
		else
			PerpWallDist = (SideDistY - DeltaDistY);

		if (PerpWallDist < 0.001)
			PerpWallDist = 0.001;

		// 화면에 그릴 높이 계산 (가까우면 길게)
		int LineHeight = (int)(HEIGHT / PerpWallDist);

		//세로로 벽을 그리는데, 그리기 시작하는 위치와 끝내는 위치를 정함
		//사실 식이 어떻게 되는지는 잘 몰루
		int DrawStart = -LineHeight / 2 + HEIGHT / 2;
		if (DrawStart < 0)DrawStart = 0;
		int DrawEnd = LineHeight / 2 + HEIGHT / 2;
		if (DrawEnd >= HEIGHT)DrawEnd = HEIGHT - 1;

		//TODO Draw velLine
		COORD StartPos = { static_cast<SHORT>(x), static_cast<SHORT>(DrawStart) };
		wchar_t WallChar = (Side == 1) ? L'\u25A0' : L'\u25A8';
		GScreen.PrintVer(WallChar, StartPos, DrawEnd - DrawStart);
	}
}
```