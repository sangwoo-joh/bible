---
layout: page
tags: [problem-solving, leetcode, python, dynamic-programming, backtracking]
title: Matchsticks to Square
---

# [Matchsticks to Square](https://leetcode.com/problems/matchsticks-to-square/)

 정수 배열 `matchsticks`가 주어지는데, `matchsticks[i]`는 `i`번째
 성냥의 길이를 나타낸다. **모든** 성냥을 사용해서 하나의 정사각형을
 만드려고 한다. 성냥을 **부러뜨릴 수 없고**, 대신 연결할 수는
 있다. 모든 성냥은 **딱 한번만** 사용할 수 있다.

 주어진 성냥으로 사각형을 만들 수 있는지 없는지 확인하자.

 성냥 배열의 길이는 1~15이고, 각 성냥의 길이는 $$ 1 \sim 10^8 $$ 이다.

 예를 들어 `matchsticks = [1, 1, 2, 2, 2]` 라면, 한 변의 길이가 2인
 정사각형을 만들 수 있다.

## 백트래킹 - 타임아웃

 일단 정사각형이 불가능한 경우부터 걸러낼 수 있다. 먼저 성냥 길이의
 합이 4의 배수가 아닌 경우는 불가능하다. 성냥 길이의 합이 4의 배수라면
 합을 4로 나눈 값이 곧 한 변의 길이가 된다는 것을 알 수 있고 이를
 `side`라고 하면, 성냥 길이 중 `side`보다 큰 값이 하나라도 있으면 역시
 불가능하다.

 이렇게 두 가지 쉽게 생각할 수 있는 케이스를 쳐내고 나면, 생각나는
 방법은 그냥 일일이 (...) 모든 변을 하나 씩 만들면서 상태 공간을
 탐색하는 것이다. 다만 목표가 뚜렷하기 때문에 어느 정도 프루닝이
 가능하여 백트래킹할 수 있다.

```python
def makesquare(matchsticks):
    s, n = sum(matchsticks), len(matchsticks)
    if s % 4 != 0:
        return False

    side = s // 4
    if any(x > side for x in matchsticks):
        return False

    def match(idx, matched):
        if idx == n:
            return matched[0] == matched[1] == matched[2] == matched[3] == side

        cur_len = matchsticks[idx]
        for i in range(4):
            if cur_len + matched[i] <= side:
                matched[i] += cur_len
                if match(idx + 1, matched):
                    return True
                matched[i] -= cur_len
        return False

    return match(0, [0, 0, 0, 0])
```

 유지할 상태는 두 가지이다. 살펴보려는 성냥의 인덱스와, 지금까지
 완성한 사각형의 변의 길이 4개이다. 성냥을 다 봤으면, 네 변이 다 같은
 `side`인지를 확인한다. 그렇지 않으면, 네 변 모두에 대해서 각각 지금
 인덱스의 성냥 길이를 추가할 수 있는지 살펴보고, 가능한 경우 그 변에
 성냥을 이어서 추가로 탐색한다. 이때 주의할 것은 탐색이 끝나고 나면
 원래 길이를 원복해줘야 한다는 것이다.

 이렇게하면 정직하게 모든 상태 공간을 탐색할 수 있지만, 모든 가능한 네
 변을 다 만들어보기 때문에 복잡도는 $$O(4^N)$$이 되어 터진다.

## 다이나믹 프로그래밍 + 비트 마스킹

 사실 이 문제는 잘 알려진 NP-Complete 문제 중 하나인 [Bin Packing
 Problem](https://en.wikipedia.org/wiki/Bin_packing_problem)으로
 환원할 수 있는 문제라고 한다. 한마디로 아직 선형 시간에 가능한
 솔루션이 없다. 그러니 최선을 다해도 지수 시간이다. 그렇지만
 $$4^N$$보다는 더 줄일 여지가 있다.

 먼저 생각해볼 부분은, 사각형을 만드는데 필요한 네 변을 전부 유지할
 필요가 있을까? 예를 들어, 사각형을 만드는 도중에 몇 개의 변을 이미
 완성한 상태에서 남은 성냥이 `[3, 3, 4, 4, 5, 5]`라고 해보자. 완성한
 변의 길이가 8이라고 하면, 여러 가능성이 있겠지만 다음과 같은 상태가
 가능할 것이다.

```
(4, 4), (3, 5), (3, 5)   -> 3 변 완성
(3, 4), (3, 5), (4), (5) -> 0 변
(3, 3), (4, 4), (5), (5) -> 1 변
```

 즉 같은 성냥 집합이라도, 이걸 어떻게 (재귀적으로 탐색해서) 이어
 붙이느냐에 따라서 결과(상태)가 완전히 달라진다. 다시 말해, **어떤
 성냥을 사용했는지**가 곧 상태를 결정한다.

 그런데 사용한 성냥을 하나의 집합으로 추적하더라도 정사각형을 만들지
 못하는 수많은 부분 문제(위에서 본 것처럼)로 갈 수 있을
 것이다. 따라서, 사용한 성냥 뿐 아니라 **얼마나 변을 완성했는지**도
 함께 추적해야 한다. 이때, 이전처럼 일일이 네 변을 유지할 필요가 전혀
 없다. 이전 접근에서 초반에 성냥의 합이 4로 나누어 떨어지는지를
 확인해서 불가능한 케이스를 판별했었는데, 여기서도 이 방법을 유사하게
 사용할 수 있다: 즉, 지금까지 사용한 성냥의 길이 합이 **변의 길이**로
 나누어 떨어지는지를 보면 된다.

 추가적으로 상태를 쳐낼 수 있는 것 중 하나는, *정사각형*이기 때문에,
 세 변만 완성하면 (전체 성냥 길이가 4로 나누어 떨어진다면) 나머지 한
 변은 자동으로 완성된다는 점이다.

 여기까지 문제를 재정의하고 나면, 부분 문제가 있는지 살펴볼 수
 있다. 작은 케이스에서 살펴보자. 성냥이 `[1, 1, 1, 1]` 있고 완성한
 변이 0개인 상태에서 시작해보자. 다음과 같이 반드시 중복되는 부분
 문제가 있다는 걸 알 수 있다. (`_`은 변을 만드는데 사용됐다는 의미)

```
(1, 1, 1, 1), 0

  +-- (1, _, 1, 1), 1  ---+- ....
  |                       |
  |                       +---- (1, _, _, 1), 2 *
  +-- (1, 1, _, 1), 1  ---+- ...
  |                       |
  ...                     +---- (1, _, _, 1), 2 *

*: 중복되는 상태
```

 따라서, 메모아이제이션을 하면 더 빨라질 수 있다.

 즉, (1) 어떤 성냥을 사용했는지, 그리고 몇 개의 변을 완성했는지를
 추적하면서 (2) 메모아이제이션을 한다. 이 두 가지를 제외하면 나머지
 전체적인 구조는 이전의 백트래킹과 같다.

```python
from functools import cache
def makesquare(matchsticks):
    s, n = sum(matchsticks), len(matchsticks)
    if s % 4 != 0:
        return False

    side = s // 4
    if any(x > side for x in matchsticks):
        return False

    @cache
    def match(used: Tuple[int], made: int):
        acc = 0
        for i in range(n):
            if used[i]:
                acc += matchsticks[i]
        if acc > 0 and acc % side == 0:
            made += 1

        if made == 3: # quick return
            return True

        margin = side - (acc % side)
        for i in range(n):
            if matchsticks[i] <= margin and not used[i]:
                used_l = list(used)
                used_l[i] = True
                if match(tuple(used_l), made):
                    return True
        return False
    return match(tuple([False] * n), 0)
```

 - 성냥의 사용 여부를 추적하기 위해서 불리언의 튜플을 사용했는데,
   왜냐하면 `@cache` 어노테이션으로 메모아이제이션 하려면 키 값을
   해싱할 수 있어야 하기 때문이다. 파이썬에서 리스트 타입은 해싱할 수
   없지만 튜플은 가능하다. 대신 이렇게 하려면 사용 여부를 변경해서
   재귀 호출을 할 때 조금 번거로운(리스트로 변환하여 값을 바꾸고 다시
   튜플로 변환) 작업을 해줘야 한다. 사실 이렇게 하지 않고 비트
   마스킹을 이용한 방법이 주된 방법이겠지만, 이 문제의 N이 작기도 하고
   파이썬에서는 속도 면에서 크게 차이가 나지 않아서 더 읽기 쉬운
   방법인 (해싱 가능한) 튜플을 썼다.
 - 사용 여부를 이용해서 모든 성냥 길이의 합 `acc`를 먼저 계산한다. 그
   후 이 합이 변의 길이 `side`로 나누어 떨어지는지를 확인한다. 만약
   나누어 떨어지면 지금 상태에서 가능한 변이 하나 더 추가된 것이므로
   `made`를 1 증가해준다. 좀더 상태를 쳐내기 위해서, 정사각형의 성질을
   이용해서 세 변이 완성되는 순간 재귀 호출을 끝낼 수 있다. (하지만
   실험해보니 4변을 다 확인하는 것과 큰 차이가 나지 않았다)
 - 재귀 호출을 타고 들어갈 때 반드시 확인해야 하는 것이 바로 **지금
   변에 추가 가능한 길이**이다. 여기서는 *어떤* 성냥이 변에 쓰였는지
   추적하지 않지만, 대신 지금까지 사용한 성냥과 만든 변의 수로 그 다음
   가능한 길이를 계산해볼 수 있다: 성냥 길이의 합을 변의 길이로 나눈
   나머지가 곧 지금 만드는 중인 변의 길이가 되는데, 이 값을 변의
   길이에서 빼주면 지금 만들 변에 추가 가능한 길이인 `margin`을 원복할
   수 있다.

 복잡도는 어떻게 될까? 성냥의 사용 유무에 따라서 모든 성냥을 훑어봐야
 하기 때문에, 시간 복잡도는 $$O(N \times 2^N)$$이 된다. 공간 복잡도의
 경우 먼저 최대 N번의 재귀 호출을 하게 되고 성냥의 사용 유무를 상태로
 하여 메모아이제이션을 하고 있기 때문에 $$O(N + 2^N)$$의 복잡도를
 갖는다.