---
title: "[PS][백준 6555] the Sierpinski Fractal"
date: 2025-12-12 23:21:12 +0900
description: ""
categories: [Computer Science, Algorithm] #[upper, lower]
tags: [cpp, ps, algorithm, recursion, implementation] #must be lower
math: true
mermaid: true
---

# [The Sierpinski Fractal](https://www.acmicpc.net/problem/6555)
![](https://velog.velcdn.com/images/garage_keeper/post/14775eb9-53ab-4a32-9f6f-bfc2e3ed9799/image.png)
## 문제 분석 및 풀이
### 설계 분석
별찍기와 비슷한 문제 규칙을 찾아서 재귀로 주어진 형태를 출력하자.

기존에 *로 출력하는 대신에 정해진 모양이 있다는데 조금 달랐고 완성된 프랙탈의 우측에는 공백을 출력하지 않는다는 걸 알아야함. (테스트 케이스의 어디선가 막히는듯)

* 전체 삼각형의 높이와 너비를 구한다.
* 위쪽, 왼쪽 아래, 오른쪽 아래 순으로 재귀적으로 그린다.

### 풀이

```cpp
#include <iostream>
#include <vector>
using namespace std;

int N;

void DrawSierpinskiFractal(vector<string>& canvas, int height, int x, int y)
{
    if (height==2)
    {
        canvas[y][x-1] = '/';
        canvas[y][x] = '\\';
        canvas[y+1][x-2] = '/';
        canvas[y+1][x-1] = '_';
        canvas[y+1][x] = '_';
        canvas[y+1][x+1] = '\\';
        return;
    }

    int half = height / 2;
    DrawSierpinskiFractal(canvas, half, x , y);
    DrawSierpinskiFractal(canvas, half, x - half , y + half);
    DrawSierpinskiFractal(canvas, half, x + half , y + half);
}


int main()
{
    while(true)
    {
        cin >> N;

        if (N==0) break;
        
        int height = 1 << N;
        int width = 1 << (N+1);
        vector<string> canvas(height, string(width, ' '));

		// 높이 x좌표 좌표 (상단 꼭짓점 좌표)
        DrawSierpinskiFractal(canvas, height, height, 0);
        for (int i=0; i<height; i++)
        {
            int end = width - 1;
            while(canvas[i][end] == ' ') end--;
            cout << canvas[i].substr(0, end+1) << "\n";
        }
        cout << "\n";
    }
   
    return 0;
}


```