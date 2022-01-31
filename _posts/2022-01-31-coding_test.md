---
title: (Python) Leetcode - Implement Stack using Queues
date: 2022-01-31 23:00 +0900
lastmod: 2022-01-31 23:00 +0900
categories: [Coding Test]
tags: [Coding Test]
mermaid: true
math: true
---

# (Python) Leetcode - Implement Stack using Queues

# 문제

Implement a last-in-first-out (LIFO) stack using only two queues. The implemented stack should support all the functions of a normal stack (`push`, `top`, `pop`, and `empty`).

Implement the `MyStack` class:

- `void push(int x)` Pushes element x to the top of the stack.
- `int pop()` Removes the element on the top of the stack and returns it.
- `int top()` Returns the element on the top of the stack.
- `boolean empty()` Returns `true` if the stack is empty, `false` otherwise.

**Example 1:**

```python
Input
["MyStack", "push", "push", "top", "pop", "empty"]
[[], [1], [2], [], [], []]
Output
[null, null, null, 2, 2, false]

Explanation
MyStack myStack = new MyStack();
myStack.push(1);
myStack.push(2);
myStack.top(); // return 2
myStack.pop(); // return 2
myStack.empty(); // return False

# Your MyStack object will be instantiated and called as such:
# obj = MyStack()
# obj.push(x)
# param_2 = obj.pop()
# param_3 = obj.top()
# param_4 = obj.empty()
```

# 풀이

### push() 할 때 큐를 이용해 재정렬

대개 스택은 연결 리스트로 하고 큐는 배열로 구현한다. 여기서는 재밌게도 큐로 스택을 디자인하는 문제이다. 

파이썬의 리스트나 데크는 스택과 큐의 모든 기능을 제공한다. 여기서는 문제의 의도에 맞게 큐의 FIFO(First In, First Out)에 해당하는 연산만 사용해서 구현한다. 

```python
import collections

class MyStack:
    def __init__(self):
        self.q = collections.deque()

    def push(self, x):
        self.q.append(x)
        # After inserting new element, reorder it so that it comes to the front.
        for _ in range(len(self.q)-1):
            self.q.append(self.q.popleft())

    def pop(self):
        return self.q.popleft()

    def top(self):
        return self.q[0]

    def empty(self):
        return len(self.q)==0
```

큐를 데크로 선언했지만 문제의 의도에 맞게 여기서는 큐의 연산만을 이용해 구현한다. 

push() 할 때, 요소를 삽입한 후에 방금 삽입한 요소를 맨 앞에 두는 상태로 전체를 재정렬 한다. 

```python
 def push(self, x):
        self.q.append(x)
        # After inserting new element, reorder it so that it comes to the front.
        for _ in range(len(self.q)-1):
            self.q.append(self.q.popleft())
```

이렇게 하면 큐에서 맨 앞 요소를 끄집어 낼 때 스택처럼 가장 먼저 삽입한 요소가 나온다. 요소 삽입시 시간 복잡도는 O(n)이 되어(for-loop가 큐 전체를 한 번 지나기 때문) 다소 비효율적이긴 하다.

for-loop가 큐의 길이보다 1 작을 때까지 반복되게 하는 게 핵심이다. 새로 삽입된 요소는 큐의 맨 오른쪽에 위치하게 되고, popleft()를 사용해서 맨 왼쪽 요소를 빼서 맨 오른쪽에 위치시킨다. 이 과정을 큐의 길이보다 1 작을 때까지 반복한다. 

예를 들어 큐에 숫자가 3개 들어가있다면 두 번 재정렬 하면 맨 오른쪽에 위치한 요소가 맨 처음 위치하게 된다.