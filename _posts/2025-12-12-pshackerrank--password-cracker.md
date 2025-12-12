---
title: "[PS][hackerrank ] password-cracker"
date: 2025-12-12 21:51:50 +0900
description: ""
categories: [Computer Science, Algorithm] #[upper, lower]
tags: [cpp, ps, algorithm, dfs, backtracking] #must be lower
math: true
mermaid: true
---

[password-cracker](https://www.hackerrank.com/challenges/password-cracker/problem?isFullScreen=true)
### 1) 문제
>**문제 설명**
> There are n users registered on a website CuteKittens.com. Each of them has a unique password represented by pass[1], pass[2], ..., pass[N]. As this a very lovely site, many people want to access those awesomely cute pics of the kittens. But the adamant admin does not want the site to be available to the general public, so only those people who have passwords can access it.
>
Yu, being an awesome hacker finds a loophole in the password verification system. A string which is a concatenation of one or more passwords, in any order, is also accepted by the password verification system. Any password can appear  or more times in that string. Given access to each of the  passwords, and also have a string , determine whether this string be accepted by the password verification system of the website. If all of the  string can be created by concatenating password strings, it is accepted. In this case, return the passwords in the order they must be concatenated, each separated by a single space on one line. If the password attempt will not be accepted, return 'WRONG PWASSWORD'.
>

<br>


### 2) 문제 분석 및 풀이
#### 1) 설계, 분석
긴 문자열을 짧은 단어들로 분해 할 수 있는지 여부를 확인하는 문제
#### 2) 풀이
>```cpp
unordered_map<int, bool> cache;
vector<string> path;
unordered_set<string> dict;
string target;
>
bool dfs(int idx)
{
    if(idx == (int)target.size()) return true;
    //if (cache.count(idx)) return false;
    >
    for (auto& word : dict)
    {
        if (target.substr(idx, word.size()) == word)
        {
            path.push_back(word);
            if (dfs(idx + word.size())) return true;
            path.pop_back();
        }
    }
    //cache[idx] = false;
    return false;
}
>
string passwordCracker(vector<string> passwords, string loginAttempt) 
{
    dict.clear();
    cache.clear();
    path.clear();
    target = loginAttempt;
    >
    for (auto& pw : passwords)
    {
        dict.insert(pw);
    }
    >
    if (dfs(0))
    {
        string ans;
        for (int i=0; i < (int)path.size(); i++)
        {
            if (i >0) ans += " ";
            ans += path[i];
        }
        return ans;
    }
    else 
    {
        return "WRONG PASSWORD";
    }
}
>```
  

<br>

