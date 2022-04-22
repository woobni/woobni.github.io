---
title: (Python) [구현] Programmers - 삼각 달팽이
date: 2022-04-07 23:36 +0900
lastmod: 2022-04-07 23:36 +0900
categories: [Coding Test]
tags: [Coding Test, Implementation Algorithm]
mermaid: true
math: true
---

# 문제

정수 n이 매개변수로 주어집니다. 다음 그림과 같이 밑변의 길이와 높이가 n인 삼각형에서 맨 위 꼭짓점부터 반시계 방향으로 달팽이 채우기를 진행한 후, 첫 행부터 마지막 행까지 모두 순서대로 합친 새로운 배열을 return 하도록 solution 함수를 완성해주세요.

![Untitled](/assets/img/2022-04-07-implementation0/Untitled.png)


### 제한사항

- n은 1 이상 1,000 이하입니다.

### **입출력 예**

![Untitled](/assets/img/2022-04-07-implementation0/Untitled%201.png)

---

# 문제 풀이

구현 문제이므로 좌표를 이용하여 풀어보고자 했다. 문제를 보면 하-우-상 방향으로 움직이고, 움직이는 칸은 만약 n=4이면 4-3-2-1, n=5이면 5-4-3-2-1 만큼 움직이는 것을 알 수 있다. 

![Untitled](/assets/img/2022-04-07-implementation0/Untitled.jpeg)

## 소스코드

```python
from typing import List

def solution(n: int) -> List[int]:
    answer = []

    # 삼각형 만들기
    arr = []
    for i in range(1, n+1):
        arr.append([0] * i)

    # 시작 좌표
    x, y = -1, 0 # 아래부터 내려가므로
    val = 1

    # 하-우-상 방향순으로 이차원 리스트에 값 채워넣기
    for i in range(n): # 방향(하, 우, 상)
        for _ in range(i, n):
            if i % 3 == 0: # 하
                x += 1
                arr[x][y] = val
            
            elif i % 3 == 1: # 우
                y += 1
                arr[x][y] = val
            
            else: # 상
                x -= 1
                y -= 1
                arr[x][y] = val

            val += 1

    for element in arr:
        answer += element

    return answer

print(solution(4))
print(solution(5))
print(solution(6))
```

[1, 2, 9, 3, 10, 8, 4, 5, 6, 7]
[1, 2, 12, 3, 13, 11, 4, 14, 15, 10, 5, 6, 7, 8, 9]
[1, 2, 15, 3, 16, 14, 4, 17, 21, 13, 5, 18, 19, 20, 12, 6, 7, 8, 9, 10, 11]