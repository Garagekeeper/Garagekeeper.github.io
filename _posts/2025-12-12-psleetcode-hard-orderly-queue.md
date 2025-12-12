---
title: "[PS][leetcode hard] Orderly Queue"
date: 2025-12-12 20:54:21 +0900
description: ""
categories: [Computer Science, Algorithm] #[upper, lower]
tags: [cpp, ps, algorithm, union find] #must be lower
math: true
mermaid: true
---

[Orderly Queue](https://leetcode.com/problems/orderly-queue/description/)
### 1) 문제
> **문제 설명**
You are given a string s and an integer k. You can choose one of the first k letters of s and append it at the end of the string.
?
Return the lexicographically smallest string you could have after applying the mentioned step any number of moves.
  >
Example 1:
>
Input: s = "cba", k = 1
Output: "acb"
Explanation: 
In the first move, we move the 1st character 'c' to the end, obtaining the string "bac".
In the second move, we move the 1st character 'b' to the end, obtaining the final result "acb".
Example 2:
>
Input: s = "baaca", k = 3
Output: "aaabc"
Explanation: 
In the first move, we move the 1st character 'b' to the end, obtaining the string "aacab".
In the second move, we move the 3rd character 'c' to the end, obtaining the final result "aaabc".


<br>

### 2) 문제 분석 및 풀이
#### 1) 설계, 분석
* 횟수 상관없이 만들 수 있는 단어중 사전순으로 제일 작은 단어 만들기
  * K>1 이면 원하는 순서의 문자열을 만들 수 있음 (정렬 후 반환)
  * K == 1이면 문자열 회전 시키면서 가장 작은 것 출력
#### 2) 풀이
``` cpp
class Solution {
public:
    string orderlyQueue(string s, int k) {
        string temp = s;
        if (k > 1)
        {
            sort(temp.begin(), temp.end());
        }   
        else
        {
            string smallest = temp;
            for (int i=0; i<s.size(); i++)
            {
                temp = "";
                for (int j=0; j<s.size(); j++)
                {
                    temp += s[(j + i) % s.size()];
                }
                if (temp < smallest)
                    smallest = temp;
            }
            temp = smallest;
        }
        return temp;
    }
};
```