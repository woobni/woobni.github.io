---
title: (Python) Progammers - 오픈채팅방
date: 2022-03-03 23:35 +0900
lastmod: 2022-03-03 23:35 +0900
categories: [Coding Test]
tags: [Coding Test]
mermaid: true
math: true
---

# 문제 설명

## **오픈채팅방**

카카오톡 오픈채팅방에서는 친구가 아닌 사람들과 대화를 할 수 있는데, 본래 닉네임이 아닌 가상의 닉네임을 사용하여 채팅방에 들어갈 수 있다.

신입사원인 김크루는 카카오톡 오픈 채팅방을 개설한 사람을 위해, 다양한 사람들이 들어오고, 나가는 것을 지켜볼 수 있는 관리자창을 만들기로 했다. 채팅방에 누군가 들어오면 다음 메시지가 출력된다.

"[닉네임]님이 들어왔습니다."

채팅방에서 누군가 나가면 다음 메시지가 출력된다.

"[닉네임]님이 나갔습니다."

채팅방에서 닉네임을 변경하는 방법은 다음과 같이 두 가지이다.

- 채팅방을 나간 후, 새로운 닉네임으로 다시 들어간다.
- 채팅방에서 닉네임을 변경한다.

닉네임을 변경할 때는 기존에 채팅방에 출력되어 있던 메시지의 닉네임도 전부 변경된다.

예를 들어, 채팅방에 "Muzi"와 "Prodo"라는 닉네임을 사용하는 사람이 순서대로 들어오면 채팅방에는 다음과 같이 메시지가 출력된다.

"Muzi님이 들어왔습니다.""Prodo님이 들어왔습니다."

채팅방에 있던 사람이 나가면 채팅방에는 다음과 같이 메시지가 남는다.

"Muzi님이 들어왔습니다.""Prodo님이 들어왔습니다.""Muzi님이 나갔습니다."

Muzi가 나간후 다시 들어올 때, Prodo 라는 닉네임으로 들어올 경우 기존에 채팅방에 남아있던 Muzi도 Prodo로 다음과 같이 변경된다.

"Prodo님이 들어왔습니다.""Prodo님이 들어왔습니다.""Prodo님이 나갔습니다.""Prodo님이 들어왔습니다."

채팅방은 중복 닉네임을 허용하기 때문에, 현재 채팅방에는 Prodo라는 닉네임을 사용하는 사람이 두 명이 있다. 이제, 채팅방에 두 번째로 들어왔던 Prodo가 Ryan으로 닉네임을 변경하면 채팅방 메시지는 다음과 같이 변경된다.

"Prodo님이 들어왔습니다.""Ryan님이 들어왔습니다.""Prodo님이 나갔습니다.""Prodo님이 들어왔습니다."

채팅방에 들어오고 나가거나, 닉네임을 변경한 기록이 담긴 문자열 배열 record가 매개변수로 주어질 때, 모든 기록이 처리된 후, 최종적으로 방을 개설한 사람이 보게 되는 메시지를 문자열 배열 형태로 return 하도록 solution 함수를 완성하라.

### 제한사항

- record는 다음과 같은 문자열이 담긴 배열이며, 길이는 `1` 이상 `100,000` 이하이다.
- 다음은 record에 담긴 문자열에 대한 설명이다.
    - 모든 유저는 [유저 아이디]로 구분한다.
    - [유저 아이디] 사용자가 [닉네임]으로 채팅방에 입장 - "Enter [유저 아이디] [닉네임]" (ex. "Enter uid1234 Muzi")
    - [유저 아이디] 사용자가 채팅방에서 퇴장 - "Leave [유저 아이디]" (ex. "Leave uid1234")
    - [유저 아이디] 사용자가 닉네임을 [닉네임]으로 변경 - "Change [유저 아이디] [닉네임]" (ex. "Change uid1234 Muzi")
    - 첫 단어는 Enter, Leave, Change 중 하나이다.
    - 각 단어는 공백으로 구분되어 있으며, 알파벳 대문자, 소문자, 숫자로만 이루어져있다.
    - 유저 아이디와 닉네임은 알파벳 대문자, 소문자를 구별한다.
    - 유저 아이디와 닉네임의 길이는 `1` 이상 `10` 이하이다.
    - 채팅방에서 나간 유저가 닉네임을 변경하는 등 잘못 된 입력은 주어지지 않는다.

### **입출력 예**

```python
record = ["Enter uid1234 Muzi", "Enter uid4567 Prodo","Leave uid1234","Enter uid1234 Prodo","Change uid4567 Ryan"]

result = ["Prodo님이 들어왔습니다.", "Ryan님이 들어왔습니다.", "Prodo님이 나갔습니다.", "Prodo님이 들어왔습니다."]
```

### 입출력 예 설명

입출력 예 #1문제의 설명과 같다.

# 문제 풀이

문제가 꽤 길게 느껴지는데 생각만큼 어렵지는 않은 문제이다. 

**핵심 포인트는 사용자의 행동에 따라 닉네임이 계속 바뀌는데 마지막 행동의 닉네임만 캐치하면 된다.**

```python
from typing import List

def solution(record: List[str]) -> List[str]:  
    answer = []
    userDB = dict()
    actions = []

    for event in record:
        info = event.split() # record -> [action, userid, nickname]
        action, userid = info[0], info[1]

        if action in ('Enter', 'Change'):
            nickname = info[2]
            userDB[userid] = nickname

        actions.append([userid, action])

    for actionInfo in actions:
        userid, action = actionInfo[0], actionInfo[1]
        
        if action == 'Enter':
            answer.append('{}님이 들어왔습니다.'.format(userDB[userid])) # answer.append(f'{userDB[userid]}님이 들어왔습니다.')
        elif action == 'Leave':
            answer.append('{}님이 나갔습니다.'.format(userDB[userid]))

    return answer
```

본 풀이에서, 만약 "Enter uid4567 Prodo” 라면 Enter는 action, uid4567은 userid, Prodo는 nickname으로 정의한다. 

먼저, 어떤 사용자가 어떤 닉네임을 사용하는지를 저장하기 위해 userDB를 정의한다. 그리고 사용자 아이디에 따른 행동을 나중에 출력해주기 위해 actions라는 리스트를 정의한다. 

그 다음, record를 순회하며 각 이벤트에 담긴 행동, 사용자, 닉네임을 분리해 각각 action, userid, nickname 변수에 넣어준다.

단, action이 ‘Leave’인 경우 nickname이 없으므로 action이 'Enter', 'Change'인 경우에만 저장한다. 'Enter', 'Change'인 경우에는 닉네임 변경이 일어날 수 있으므로 userDB에 userid를 키, nickname을 값으로 저장한다.

**딕셔너리 특성 상 기존에 저장되어 있지 않던 사용자라면 새로이 데이터가 추가될 것이고, 기존에 저장되어 있던 사용자라면 변경된 닉네임으로 갱신될 것이다.**

그리고 actions에 사용자에 따른 행동을 저장하기 위해 userid와 action 쌍을 저장한다. 

다음은 actions 리스트를 순회한다. 본 풀이에서는 **사용자의 닉네임 정보를 userDB에서 관리하므로 userid만 알면 닉네임을 가져올 수 있다. → userDB[userid]**

행동에 따라 출력할 문장을 answer 리스트에 추가한다.