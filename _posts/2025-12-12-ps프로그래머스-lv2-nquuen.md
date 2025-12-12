---
title: "[PS][프로그래머스-lv2] NQuuen"
date: 2025-12-12 21:34:58 +0900
description: ""
categories: [Computer Science, Algorithm] #[upper, lower]
tags: [cpp, ps, algorithm, dfs, backtracking] #must be lower
math: true
mermaid: true
---

[N-Queen](https://school.programmers.co.kr/learn/courses/30/lessons/12952?language=cpp)
### 1) 문제

> ![](https://velog.velcdn.com/images/garage_keeper/post/6e6578f0-b972-43a4-a29f-8ab8c0aa0662/image.png)

<br>


### 2) 문제 분석 및 풀이
#### 1) 설계, 분석
브루트포스 백트레킹에 사용하는 전형적이 예시
#### 2) 풀이
>``` cpp
#include <string>
#include <vector>
using namespace std;
>
vector<vector<bool>> board;
int N;
int answer;
bool PlaceQueen(int x, int y)
{
    //열 체크 (행은 이미 dfs구조에서 확인)
    for (int i = 0 ; i<N; i++)
    {
        if (board[i][y] == true) return false;
    }
  >  
    //왼쪽 대각선
    //좌상향 방향만 체크 (로직상 가장 밑에 놓여질 퀸이기 때문에)
    int nx = x;
    int ny = y;
    while (true)
    {
        nx--;
        ny--;
        if (nx < 0|| ny < 0) break;
        if (board[nx][ny] == true) return false;
    }
    >
    //오른쪽 대각선
    //우상향 방향만 체크 (로직상 가장 밑에 놓여질 퀸이기 때문에)
    nx = x;
    ny = y;
    while (true)
    {
        nx--;
        ny++;
        if (nx < 0 || ny >= N) break;
        if (board[nx][ny] == true) return false;
    }
    >
    return true;
}
>
void dfs(int row, int num)
{
    // 기저 조건
    // 퀸을 N개 배치하면 종료
    if (num == N)
    {
        answer++;
        return;
    }
    >
    // 각 row에는 퀸을 하나만 놓을 수 있으니
    // 각 Cell을 볼것이 아니라 row 기준으로 백트래킹
    // 0.현재 row에 퀸을 배치할 수 있는지 확인
    //   0-1 놓을 수 있으면 놓고 다음 row로 dfs;
    for (int i=0; i<N; i++)
    {
        if (board[row][i] == false)
        {
            if (PlaceQueen(row, i))
            {
                board[row][i] = true;
                dfs(row + 1 , num + 1);
                board[row][i] = false;
            }
        }
    }
}
>
int solution(int n) {
    board.resize(n, vector<bool>(n, false));
    N = n;
    dfs(0, 0);
    >
    return answer;
}
 >```

  
<br>
  