---
title: "Sprite casting"
date: 2026-05-27 23:49:46 +0900
description: ""
categories: [Computer Science, Graphics] #[upper, lower]
tags: [graphics, raycast] #must be lower
math: true
mermaid: true
---
<br>

## Sprite casting
![](https://lodev.org/cgtutor/images/raycastsprites1.jpg)
![](https://lodev.org/cgtutor/images/raycastsprites2.jpg)

저번 게시물까지는 벽, 천장, 바닥을 그릴 수 있었다.   
스프라이트 캐스팅은 말 그대로 Sprite(그림?, 위 사진의 기둥, 오크통, 초록 램프에 해당한다.) 이전까지의 캐스팅들과는 또 다른 방법을 사용한다.

간단하게 설명한 절차는 다음과 같고, 이후 항목별로 자세히 살펴보자

* 스프라이트들의 우선순위를 정한다
  * 먼곳이 우선순위가 제일 높다
    * 먼곳부터 그려야 가려지게 그릴 수 있다.
* 카메라와 스프라이트의 상대적인 위치를 구한다.
* 위에서 구한 위치(월드 좌표계)를 카메라 좌표계로 변환한다.
* 수직거리를 이용해 스프라이트의 사이즈를 계산한다.
* 스프라이트를 수직 스트라이프 한줄씩 그린다. (벽 뒤에 있으면 안그린다.)

<br>

### 스프라이트의 우선순위를 정한다.
스프라이트의 우선순위를 정하는 이유와 어떻게 우선순위를 부여할지 알아보자.

[그림출처](https://velog.io/@jungizz_/%EC%9C%A0%EB%8B%88%ED%8B%B0-%EC%89%90%EC%9D%B4%EB%8D%94-%EC%8A%A4%ED%83%80%ED%8A%B8%EC%97%85-Part-16-%EC%95%8C%ED%8C%8CAlpha)

뒤에서부터 그리면

![](https://velog.velcdn.com/images/jungizz_/post/e1a962c7-2258-43dc-b001-56be13dbe660/image.png)

앞에서부터 그리면

![](https://velog.velcdn.com/images/jungizz_/post/7efb6e6f-f47a-4c09-9d6d-7b1c9c49ca55/image.png)

어디서부터 그리는지에 따라서 출력되는 결과가 달라진다.  
컴퓨터 그래픽스에서는 이를 활용하기 위해서 Z버퍼를 사용하는데 픽셀을 덧칠할때(물체가 겹칠때) Z버퍼가 더 작은쪽의 픽셀을 칠한다. (Z버퍼가 작으면 카메라에 가깝다)  
이렇게하면 그리는 순서가 달라져도 동일한 결과가 나오기 때문에 그리는 순서를 환경에 따라서 선택하면 된다.  

> * 뒤에서부터 그리면 완전히 가려져도 일단 그려야하는 문제 발생  
> * 앞에서부터 그리면 반투명처리에 문제가 생긴다 (알파 소팅으로 어느정도 해결이 가능하지만 완벽한 것은 아님)

우리의 RayCasting 환경에서는 뒤에서부터 그리는 방식을 선택한다. Z버퍼도 단순히 벽과의 거리를 저장해서 스프라이트와 카메라의 거리를 비교할 때 사용한다.

먼거부터 그리기 때문에 화면(카메라)에서 먼 스프라이트가 높은 우선순위를 가지도록 스프라이트들을 정렬한다.

<br>

### 스프라이트의 상대적 위치(카메라로 부터)를 구한다.
스프라이트를 월드좌표에 배치하고 이걸 알맞게 화면에 그려주기 위한 첫 단계이다.
단순히 스프라이트가 카메라로부터 어떤 방향(위치)에 있나를 계산한다.

$$
SpriteVecX = SpritePosX - PlayerX \\
SpriteVecY = SpritePosY - PlayerY
$$

이 계산의 의미는 다음과 같다. 월드좌표에 존재하는 스프라이트의 위치 벡터에서 플레이어의 위치 벡터를 빼면 플레이어로부터 스프라이트로 향하는 벡터가 나타나고 이 벡터가 플레이어(카메라)에 상대적인 위치가 된다.

<br>

### 스프라이트의 상대적인 위치를 카메라 좌표계로 변환한다.

카메라 좌표계로 변환하기 전에 카메라 좌표계를 월드 좌표계로 변환하는 방법을 알아보자  

먼저 카메라 좌표계는 이 프로젝트의 환경에서는 2개의 축을 가진다. 화면 정중앙이 (0,0)이고 화면의 오른쪽이 X축, 화면 안쪽이 Y축이다. X가 증가하면 오른쪽으로 움직인 것이고, Y가 증가하면 화면 안쪽으로 움직인 것이다.

카메라 좌표계의 X축은 월드 좌표의 카메라 평면 벡터 방향, Y축은 플레이어의 방향벡터 방향이다.

카메라 좌표에서 X축(우측)으로 1 이동은 월드좌표 기준으로 1 * 카메라 평면 벡터 만큼 이동한 것이고  
Y축(정면)으로 1 이동은 월드좌표 기준으로 1 * 플레이어 방향벡터 만큼 이동한 것이다.

<br>

$$
\begin{pmatrix}worldX\\worldY\\ \end{pmatrix} = 
CamX \begin{pmatrix}PlaneDirX\\PlaneDirY\\\end{pmatrix} +
CamY \begin{pmatrix}PlayerDirX\\PlayerDirY\\\end{pmatrix}
$$

<br>

이를 식으로 나타내면 위처럼 나타낼 수 있고 이걸 행렬로 정리하면 아래와 같이 나온다.

<br>

$$
\begin{pmatrix}worldX\\worldY\\ \end{pmatrix} = 
\begin{pmatrix}PlaneDirX & PlayerDirX\\PlaneDirY & PlayerDirY\\\end{pmatrix} \begin{pmatrix}CamX\\CamY\\\end{pmatrix}
$$

<br>

이식은 중간에 새로 만들어진 행렬 (튜토리얼에선 fancy camera matrix라 한다. 여기서도 편의상 카메라 행렬이라 부르겠다)과 카메라 화면의 좌표를 곱한 것인데 이러면 카메라 화면의 좌표가 월드좌표로 변하게 된다.  
이걸 반대로 하려면 즉, 월드좌표를 카메라 좌표로 변환하려면 카메라 행렬의 역행렬을 곱해주면 된다. 

<br>

$$
{\begin{pmatrix}PlaneDirX & PlayerDirX\\PlaneDirY & PlayerDirY\\\end{pmatrix}}^{-1}\begin{pmatrix}worldX\\worldY\\ \end{pmatrix} = 
\begin{pmatrix}CamX\\CamY\\\end{pmatrix}
$$

<br>

카메라 행렬과 역행렬이 곱해져서 항등항렬 I가 되고 우변에는 카메라 좌표만 남는다. 2차원 행렬의 역행렬은 비교적 계산이 간단하다.
<br>
<br>
$$
{\begin{pmatrix}PlaneDirX & PlayerDirX\\PlaneDirY & PlayerDirY\\\end{pmatrix}}^{-1} = \frac{1}{PlaneX * PlayerDirY - PlayerDirX * PlaneY}  *
 \begin{pmatrix}PlayerDirY & -PlayerDirX\\-PlaneDirY & PlayerDirX\\\end{pmatrix}
$$

 <br>

 이 행렬을 카메라 좌표에 곱해주면 월드좌표로 변환돤다.

<br>

 ### 수직거리를 이용해 스프라이트의 사이즈를 계산한다.
 스프라이트의 크기를 정하기전에 스프라이트가 찍힐 좌표를 먼저 보정해준다.

 ```cpp
 // transformX와 transformY는 카메라좌표
 int SpriteScrrenX = int((GScreen.HorSize / 2) * (1 + transformX / transformY));
 ```

 이 식은 스프라이트가 찍힐 X좌표 크게 2가지로 나눠진다. 위치보정, 범위 변환이다. 먼저

 $\frac{transformX}{transformY}$ 이 부분을 설명하면 $transformY$ 가 증가하면 식의 값이 0에 가까워진다. 즉 화면에서 멀수록 가운데로 오게 해준다. 그런데 화면에서 멀면 왜 가운데로 와야할까?

 기찻길을 생각하면 이해하기 쉽다 기찻길 중간에서 서있으면 멀리 바라볼수록 기찻길이 중앙으로 모인다. 이 현상이 적용된 것이다.

 그런데 이 값은 $-1 \leq \frac{transformX}{transformY} \leq 1$ 인데 화면에는 음수 좌표계가 없으니 1을 더한다. 그래서 $0 \leq SpriteScrrenX \leq 2$로 변환되는데 이거 0~screenWidth까지 대응 시키기 위해서 screenWidth/2를 곱해준다.

 이렇게 스프라이트가 찍힐 위치를 정했으면 스프라이트의 크기를 정한다.
 코드를 보면서 살펴보면 먼저 높이를 구하고 스프라이트를 그릴 Y범위를 지정해준다
 그릴 좌표를 기준이 스프라이트의 정중앙이 되도록 그림
 ```cpp
 		// 스프라이트의 높이
		// 어안 렌즈 방지를 위해 실제 거리 말고 transformY 사용
		// 스프라이트의 높이가 화면에 들어가 있을수록 작아짐( 플레이어로 부터 멀리 있을수록 작아짐)
		int SpriteHeight = abs(int(GScreen.VerSize / transformY));
		
		int DrawStartY = -SpriteHeight / 2 + GScreen.VerSize / 2;
		if (DrawStartY < 0) DrawStartY = 0;
		int DrawEndY = SpriteHeight / 2 + GScreen.VerSize / 2;
		if (DrawEndY >= GScreen.VerSize) DrawEndY = GScreen.VerSize - 1;

		// 스프라이트의 너비
		int SpriteWidth = abs(int(GScreen.VerSize / transformY));

		int DrawStartX = -SpriteWidth / 2 + SpriteScrrenX;
		if (DrawStartX < 0) DrawStartX = 0;
		int DrawEndX = SpriteWidth / 2 + SpriteScrrenX;
		if (DrawEndX >= GScreen.HorSize) DrawEndX = GScreen.HorSize;
 ```

<br>

 ### 스프라이트를 수직 스트라이프 한줄씩 그린다. (벽 뒤에 있으면 안그린다.)
 ```cpp
for (int Stripe = DrawStartX; Stripe < DrawEndX; Stripe++)
{
	for (int j = DrawStartY; j < DrawEndY; j++)
	{
		// 현재 픽셀이 텍스쳐의 가로에서 몇번째인지 확인
		// (현재 위치- 시작 위치) * 텍스쳐 크기 / 전체 너비
		int texX = int(256 * (Stripe - (-SpriteWidth / 2 + SpriteScrrenX)) * SpriteTextureTest_RowSize / SpriteWidth) / 256;
		//int texX = int(256 * (stripe - (-spriteWidth / 2 + spriteScreenX)) * texWidth / spriteWidth) / 256;

		// 경계 안으로 들어 오도록
		if (texX < 0) texX = 0;
		if (texX >= SpriteTextureTest_RowSize) texX = SpriteTextureTest_RowSize - 1;

		// 1. transformY이 0이하면 화면의 뒤쪽
		// 2. i가 화면에 있는지
		// 3. 벽보다 가까이 있는지
		if (transformY > 0 && Stripe >= 0 && Stripe < GScreen.HorSize && transformY < GScreen.Zbuffer[Stripe])
		{
			//256 and 128 factors to avoid floats 실수를 피하기 위해서 이걸 곱했다는데 잘 몰루
			int d = j * 256 - GScreen.VerSize * 128 + SpriteHeight * 128;
			int texY = ((d * SpriteTextureTest_RowSize) / SpriteHeight) / 256;

			// 경계 안으로 들어 오도록
			if (texY < 0) texY = 0;
			if (texY >= SpriteTextureTest_RowSize) texY = SpriteTextureTest_RowSize - 1;

			wchar_t SpriteChar = Sprites[SpriteOrder[i]].SpriteTexture[texY][texX];


			//GScreen.PrintChar(SpriteChar, Stripe, j );

			// 공백 처리 (텍스처 배열에서 ' ' 즉, 빈 공간은 투명화 처리하여 그리지 않음)
			if (SpriteChar != L' ')
			{
				// GScreen의 i(가로), j(세로) 좌표에 글자(spriteChar)를 그리는 함수를 호출하세요.
				// 예시: GScreen.Buffer[j][i] = spriteChar;
				GScreen.PrintChar(SpriteChar, Stripe, j);
			}
		}
	}
}
 ```

* 왼쪽에서 오른쪽으로 진행하면서 스프라이트를 그린다
	* 위에서 아래순으로 스프라이트를 그린다.
   * 현재 카메라 좌표가 스프라이트 텍스쳐의 가로에서 몇번째인지 확인 (이걸 통해서 작은것도 크게 그릴 수 있음)
   * 스프라이트가 화면 뒤쪽, 그릴 좌표가 화면 범위 밖, 벽보다 가까이 있는지 확인
     * Z버퍼를 통해 해당 x값에 있는 벽과 비교
   * 현재 카메라 좌표가 스프라이트 텍스쳐의 세로에서 몇번째인지 확인 (이걸 통해서 작은것도 크게 그릴 수 있음)
   * 위에서 몇번째인지 계산한걸 토대로 스프라이트(2차원배열)의 어떤 값을 출력할지 정해잔다
   * 정해진 문자를 출력한다.

<br>

### 구현 사진

![ProfilerExample](/assets/img/Sprite_1.png)
![ProfilerExample](/assets/img/Sprite_2.png)
![ProfilerExample](/assets/img/Sprite_3.png)

<br>

### 전체 코드
```cpp

Sprites = new FSprite[NumOfSprite]
{
	{
		5.5, 2.5, new const wchar_t* [SpriteTextureTest_RowSize]
		{
			L" ▄ ",
			L"█O█",
			L" ▀ ",
		}
	},

	{
		22.5, 11.5, new const wchar_t* [SpriteTextureTest_RowSize]
		{
			{L"###" },
			{L"# #" },
			{L"###" },
		}	
	},

	{
		19, 3, new const wchar_t* [SpriteTextureTest_RowSize]
		{
			{L"@@@" },
			{L"@ @" },
			{L"@@@" },
		}
	},

};

void DrawSprite()
{
	// sortSprite Far to Close (가장 먼거부터 그려야, 가려질 수 있음, 앞에서부터 그리면 전부 다 나옴)
	// 벡터 거리 저장
	// 벡터 우선순위 초기화

	for (int i = 0; i < NumOfSprite; i++)
	{
		SpriteOrder[i] = i;
		SpriteDistance[i] = ((Player.X - Sprites[i].X) * (Player.X - Sprites[i].X) + (Player.Y - Sprites[i].Y) * (Player.Y - Sprites[i].Y));
	}

	SortSprite(&SpriteOrder, &SpriteDistance, NumOfSprite);

	for (int i = 0; i < NumOfSprite; i++)
	{

		// 카메라에서 스프라이트 까지의 상대적인 위치
		double SpriteX = Sprites[SpriteOrder[i]].X - Player.X;
		double SpriteY = Sprites[SpriteOrder[i]].Y - Player.Y;

		// 카메라 행렬(카메라 좌표계)
		// 카메라의 정면 (스크린 안쪽)은 플레이어의 방향벡터
		// 카메라의 오른쪽 (스크린의 오른쪽)은 카메라 평면 벡터
		// 카메라 좌표계 기준 정면으로 1 이동한건 월드 입장에서는 1 * 플레이어 방향벡터 만큼 이동한 것
		// 카메라 좌표계 기준 우측으로 1 이동한건 월드 입장에서는 1 * 카메라 평면의 방향벡터 만큼 이동한 것

		// 카메라 기준으로 Cx, Cy 이동 했으면 최종 월드좌표 Wx,Wy 는 다음과 같다.
		//[Wx]   [PlaneX  DirX]   [Cx]	Wx = PlaneX * Cx + DirX * Cy
		//[  ] = [            ] x [  ]
		//[Wy]   [PlaneY  DirY]   [Cy]	Wx = PlaneY * Cx + DirY * Cy
		// 
		// 여기서 Wx를 계산하는데, DirX * Cy는 왜 들어가는지 고민했는데, 이건 카메라 자체가 회전된 상태일 때 유효하다
		// 카메라가 회전하지 않았다면 초기 방향에따라서 DirX혹은 DirY가 0이다. (방향 벡터가 (1,0) (-1,0) 이런식으로 나오니까)
		// 그래서 (1,0)인 경우에 Wx = PlaneX * Cx + 0 * Cy가 되어서 Wx = PlaneX 가 된다. 하지만 방향 벡터가 회전했다면 
		// 예를 들어 (1,1)이라고 하면 카메라 평면은 (1,-1)이된다. 여기서 카메라의 좌표가 (0,0) 에서 (1,1)이 되면
		// 카메라 기준으로는 오른쪽으로 한칸 안쪽으로 한칸 이동한다. 이걸 월드 입장에서 보면 (1,-1) 방향으로 이동 후 (1,1) 방향으로 이동한게 된다.
		// 맨 처음에는 PlaneX * Cx 만큼 이동하고 1,1 방향으로 이동할 때, DirX * Cy 만큼 이동한다
		//          y  ▲
		//             |  ↘(Plane 방향)    ↗ (Dir 방향) 
		//	           |    ↘            ↗
		//             |      ↘        ↗
		//             |        ↘    ↗
		//             |          ↘↗
		//             -------------------▶ x
		//  방금 말한 상황이 이 그림인데 Wx는 Plane의 x방향으로 Cx만큼 이동한 것 +  Dir의 x방향으로 Cy 이동한 것 
		//  Wy는 Plane의 y방향으로 Cx만큼 이동, 으로 이동한 것 + dir의 y Cy만큼 이동방향으로 이동
		//  상대위치에 카메라의 역행렬을 곱해서 카메라 좌표의 X,Y를 구할 수 있다.(Y는 깊이, 화면 안쪽으로 들어가는)
		//transform sprite with the inverse camera matrix
		// [ planeX   dirX ] -1                                       [ dirY      -dirX ]
		// [               ]       =  1/(planeX*dirY-dirX*planeY) *   [                 ]
		// [ planeY   dirY ]                                          [ -planeY  planeX ]

		double invDet = 1.0 / (Player.PlaneX * Player.DirY - Player.DirX * Player.PlaneY);

		// 카메라 중심에서 좌우로 얼마나 떨어져 있는가
		double transformX = invDet * (Player.DirY * SpriteX - Player.DirX * SpriteY); // 화면의 좌우 (양수면 오른쪽)
		// 카메라 중심에서 앞뒤로 얼마나 떨어져 있는가.
		double transformY = invDet * (-Player.PlaneY * SpriteX + Player.PlaneX * SpriteY); // 화면 안쪽으로 얼마나 들어가 있는지

		// ????
		// 원근투영
		// 어떤 물체가 왼쪽으로 2미터 떨어진 위치에 있다.
		// 화면에 가까울때는 화면을 많이 돌려야하고, 크게 보인다.
		// 화면에서 멀때는 화면의 중앙에 가까운곳에 작게 보인다.
		// transformX / transformY 이게 화면에서 멀수록 가운데로오게 해준다.
		// 이 값은 -1 < X < 1인데 화면에는 음수 좌표계가 없으니까 +1
		// 0 < x < 2 범위의 X를 0~130까지의 정수로 변환
		// 스프라이트가 찍힐 X 좌표
		int SpriteScrrenX = int((GScreen.HorSize / 2) * (1 + transformX / transformY));

		// 스프라이트의 높이
		// 어안 렌즈 방지를 위해 실제 거리 말고 transformY 사용
		// 스프라이트의 높이가 화면에 들어가 있을수록 작아짐( 플레이어로 부터 멀리 있을수록 작아짐)
		int SpriteHeight = abs(int(GScreen.VerSize / transformY));
		//세로 비율 조정
		//SpriteHeight = static_cast<int>(SpriteHeight * 0.5);
		int DrawStartY = -SpriteHeight / 2 + GScreen.VerSize / 2;
		if (DrawStartY < 0) DrawStartY = 0;
		int DrawEndY = SpriteHeight / 2 + GScreen.VerSize / 2;
		if (DrawEndY >= GScreen.VerSize) DrawEndY = GScreen.VerSize - 1;

		// 스프라이트의 너비
		int SpriteWidth = abs(int(GScreen.VerSize / transformY));
		// 가로 비율 조장
		//SpriteWidth = static_cast<int>(SpriteWidth * 2.0);
		int DrawStartX = -SpriteWidth / 2 + SpriteScrrenX;
		if (DrawStartX < 0) DrawStartX = 0;
		int DrawEndX = SpriteWidth / 2 + SpriteScrrenX;
		if (DrawEndX >= GScreen.HorSize) DrawEndX = GScreen.HorSize;

		for (int Stripe = DrawStartX; Stripe < DrawEndX; Stripe++)
		{
			for (int j = DrawStartY; j < DrawEndY; j++)
			{
				// 현재 픽셀이 텍스쳐의 가로에서 몇번째인지 확인
				// (현재 위치- 시작 위치) * 텍스쳐 크기 / 전체 너비
				int texX = int(256 * (Stripe - (-SpriteWidth / 2 + SpriteScrrenX)) * SpriteTextureTest_RowSize / SpriteWidth) / 256;
				//int texX = int(256 * (stripe - (-spriteWidth / 2 + spriteScreenX)) * texWidth / spriteWidth) / 256;

				// 경계 안으로 들어 오도록
				if (texX < 0) texX = 0;
				if (texX >= SpriteTextureTest_RowSize) texX = SpriteTextureTest_RowSize - 1;

				// 1. transformY이 0이하면 화면의 뒤쪽
				// 2. i가 화면에 있는지
				// 3. 벽보다 가까이 있는지
				if (transformY > 0 && Stripe >= 0 && Stripe < GScreen.HorSize && transformY < GScreen.Zbuffer[Stripe])
				{
					//256 and 128 factors to avoid floats 실수를 피하기 위해서 이걸 곱했다는데 잘 몰루
					int d = j * 256 - GScreen.VerSize * 128 + SpriteHeight * 128;
					int texY = ((d * SpriteTextureTest_RowSize) / SpriteHeight) / 256;

					// 경계 안으로 들어 오도록
					if (texY < 0) texY = 0;
					if (texY >= SpriteTextureTest_RowSize) texY = SpriteTextureTest_RowSize - 1;

					wchar_t SpriteChar = Sprites[SpriteOrder[i]].SpriteTexture[texY][texX];


					//GScreen.PrintChar(SpriteChar, Stripe, j );

					// 공백 처리 (텍스처 배열에서 ' ' 즉, 빈 공간은 투명화 처리하여 그리지 않음)
					if (SpriteChar != L' ')
					{
						// GScreen의 i(가로), j(세로) 좌표에 글자(spriteChar)를 그리는 함수를 호출하세요.
						// 예시: GScreen.Buffer[j][i] = spriteChar;
						GScreen.PrintChar(SpriteChar, Stripe, j);
					}
				}
			}
		}
	}
}
```