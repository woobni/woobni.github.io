---
title: (Python)Leetcode - Reorder Log Files
date: 2022-01-24 23:36 +0900
lastmod: 2022-01-24 23:36 +0900
categories: [Coding Test]
tags: [Coding Test]
mermaid: true
math: true
---

# [Python] 리트코드 - 로그 파일 재정렬(Reorder Log Files)

# 문제

You are given an array of `logs`. Each log is a space-delimited string of words, where the first word is the **identifier**.

There are two types of logs:

- **Letter-logs**: All words (except the identifier) consist of lowercase English letters.
- **Digit-logs**: All words (except the identifier) consist of digits.

Reorder these logs so that:

1. The **letter-logs** come before all **digit-logs**.
2. The **letter-logs** are sorted lexicographically by their contents. If their contents are the same, then sort them lexicographically by their identifiers.
3. The **digit-logs** maintain their relative ordering.

Return *the final order of the logs*.

**Example 1:**

```python
Input: logs = ["dig1 8 1 5 1","let1 art can","dig2 3 6","let2 own kit dig","let3 art zero"]

Output: ["let1 art can","let3 art zero","let2 own kit dig","dig1 8 1 5 1","dig2 3 6"]

Explanation:
The letter-log contents are all different, so their ordering is "art can", "art zero", "own kit dig".
The digit-logs have a relative order of "dig1 8 1 5 1", "dig2 3 6".
```

**Example 2:**

```python
Input: logs = ["a1 9 2 3 1","g1 act car","zo4 4 7","ab1 off key dog","a8 act zoo"]

Output: ["g1 act car","a8 act zoo","ab1 off key dog","a1 9 2 3 1","zo4 4 7"]
```

# 풀이

### 람다와 + 연산자를 이용

실무에서도 이 같은 로직은 자주 쓰인다고 하며 매우 실용적인 문제라고 한다. 핵심은 파이썬 내장 함수인 isdigit()을 이용해서 숫자 여부인지를 판별해 구분한다. 

```python
# Mark the type with the typing module.
from typing import List 

def reorderLogfiles(logs: List[str]) -> List[str]:
    letters, digits = [], []

    for log in logs:
        # Identifiers don't affect the order.
        if log.split()[1].isdigit():
            digits.append(log)
        else:
            letters.append(log)

    # If the letters are the same, do it in the order of identifiers.
    letters.sort(key=lambda x: (x.split()[1:], x.split()[0]))

    return letters + digits # A letter log precedes a digit log.
```

숫자로 변환 가능한 로그는 digits에 그렇지 않은 문자 로그는 letters에 추가된다. 

문제를 보면 문자 로그는 정렬을 해주어야 하므로 람다식을 이용하여 식별자를 제외한 문자열 [1:]을 키로 하여 정렬한다. 동일한 경우 후순위로 식별자 [0]을 지정해 정렬되도록 한다. 

마지막으로 +연산자를 이용하여 문자 로그가 숫자 로그보다 먼저 오도록 붙여준다.

---

### 참고

**람다 표현식**이란 식별자 없이 실행 가능한 함수를 말하며, 함수 선언 없이도 하나의 식으로 함수를 단순하게 표현할 수 있다. 

(여기서 식별자란 변수, 상수, 함수, 사용자 정의 타입 등에서 다른 것들과 구분하기 위해서 사용되는 변수의 이름, 상수의 이름, 함수의 이름, 사용자 정의 타입의 이름 등 '이름'을 일반화 해서 지칭하는 용어이다.)

만약 s가 [’2 A’, ‘1 B’, ‘4 C’, ‘1 A’]라면 sorted()로 정렬한 결과는 다음과 같다.

```python
s = [’2 A’, ‘1 B’, ‘4 C’, ‘1 A’]
sorted(s)

-> [‘1 A’, ‘1 B’, ’2 A’, ‘4 C’]
```

만약 람다를 사용하지 않고 직접 함수를 선언한다면 다음과 같은 형태가 된다. 

풀이에서 람다를 사용하지 않고 직접 함수를 선언한다면 다음과 같은 형태가 된다.

```python
def func(x):
	...
	return x.split()[1], x.split()[0]
	...

```

```python
def func(x):
...
	return x.split()[1], x.split()[0]
...
s.sort(key=func)
s

-> [‘1 A’, ’2 A’, ‘1 B’, ‘4 C’]
```

이제 람다 표현식을 사용하면 다음과 같다.

```python
s.sort(key=lambda x: (x.split()[1], x.split()[0]))
s

-> [‘1 A’, ’2 A’, ‘1 B’, ‘4 C’]
```