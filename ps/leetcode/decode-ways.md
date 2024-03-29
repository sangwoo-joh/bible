---
layout: page
tags: [problem-solving, leetcode, python, dynamic-programming, string]
title: Decode Ways
---

# [Decode Ways](https://leetcode.com/problems/decode-ways/)
 `A-Z`를 담은 메시지는 다음과 같은 규칙에 의해서 숫자로 인코딩할 수
 있다:

```python
'A' -> "1"
'B' -> "2"
...
'Z' -> "26"
```

 이렇게 인코딩된 메시지를 디코딩하려면, 이 맵핑을 거꾸로 적용해서
 숫자들을 묶어야 한다. 그러면 가능한 경우가 여러 개일 수 있다. 예를
 들면, "11106"은:

```python
(1 1 10 6) -> "AAJF"
(11 10 6)  -> "KJF"
```

 이때 `(1 11 06)` 으로 묶는 것은 불가능하다. 왜냐하면 `6`과 `06`은
 다르기 때문이다.

 숫자가 담긴, 즉 인코딩된 메시지가 주어졌을 때, 이걸 디코딩할 수 있는
 방법의 개수를 구하자. 이 개수는 32 비트 정수 범위 안임이 보장된다.

## Dynamic Programming
 내가 좋아하지 않는 토픽 두 가지가 섞인 문제다: 문자열 & 다이나믹
 프로그래밍. 문자열 문제는 대부분 수수께끼 같고, 다이나믹 프로그래밍은
 점화식을 끌어내기가 어렵기 때문에 별로 선호하지 않는다. 그래도 피할
 수 없으니 어쩌겠나. 되도록 즐기려고 노력할 수 밖에.

 주어진 숫자 문자열을 순서대로 가면서, 그 다음 가능한 선택지가 뭘지
 판별해 나가보자. 이때, 중복되는 부분이 있을까? "2326" 문자열을 가지고
 생각해보자.

```python
"2326"
"2" -> "3" -> (2, 6), (26)
"23" -> (2, 6), (26)
```

 즉, "2"로 디코딩한 경우, 그 다음은 "3"만 디코딩 가능하니까 (`32 >
 26`) 남은 2, 6으로 가능한 경우는 두 가지 뿐이다. 그 다음 "23"을
 디코딩한 경우, 역시 또 2, 6이 남고 이때 개수는 기존에 계산한 결과와
 같다. 즉, 이 문제는 부분 문제로 쪼갤 수 있고, 부분 문제 중에서
 중복되는 부분이 존재한다.

 그럼 어떻게 계산할 수 있을까? 인덱스를 기준으로 0부터 재귀적으로
 시작해서, 가능한 경우에 따라 문자 1개 (`i + 1`) 또는 2개 (`i + 2`) 씩
 확인한다고 해보자. 먼저 base case는 뭘까? 당연히 떠오르는 것 중
 하나는 지금 위치의 문자가 `0`이면 가능한 개수도 `0`이다. 왜냐하면
 인코딩이 `1`부터 시작하기 때문이다.

 또, 이렇게 하나 또는 둘 씩 가능한 경우를 확인하면서 문자열의 끝에
 도달한 경우에는 최종적으로 가능한 1개를 찾은 것이다. 그런데 여기서
 문자 1개 또는 2개를 확인하기 때문에 고려해야 할 사항이 있다. 예를
 들어 "26"을 디코딩한다고 해보자. 그러면 가능한 경우는 아래와 같다.

 1. `i=0`이고 1개를 확인(2) -> 그 다음 `i=1`이고 1개를 확인(6) ->
    `i=1` 이고 끝에 도달했으므로 1개 리턴
 2. `i=0`이고 2개를 확인(26) -> `i=2` 이고 끝에 도달했으므로 1개 리턴

 둘 다 끝에 도달한 건 맞지만 이때 1번과 2번의 경우 인덱스의 값이
 다르다. 첫번째 경우는 `i=1`일 때 끝나는데 이는 곧 `len - 1`이다. 반면
 두 번째 경우는 한번에 두 글자를 확인했기 때문에 `i=2`가 되고 이때는
 `len`과 같다. 따라서, "끝에 도달함"은 이 두 가지를 모두 고려해야
 한다.

 그런데 여기서 또 하나 tricky한 부분이 있다. `len - 1`, 즉 한 글자의
 유효함을 확인하기 전에, 현재 인덱스의 문자가 유효한지를 확인해야
 한다. 즉, 지금 인덱스의 문자가 `0`인지 아닌지를 확인해야
 한다. 왜냐하면, 이전까지 하나 또는 두개씩 잘 확인해오다가 갑자기
 마지막에 `0`이 나와버리면, 디코딩 가능한 경우가 없기 때문이다.

 횡설수설 말이 많았는데, 아직 내가 핵심을 간결하게 설명하지 못하는 것
 같다. 일단 코드로 작성해보자.

```python
from functools import cache
def num_decoding(s):

    @cache
    def count_valid(idx):
        # base 1: end of 2-char check
        if idx == len(s):
            return 1

        # base 2: impossible
        if s[idx] == '0':
            return 0

        # base 3: end of 1-char check
        if idx == len(s) - 1:
            return 1

        # accum 1-char
        answer = count_valid(idx + 1)

        # accum 2-char
        if int(s[idx:idx + 2]) <= 26:
            answer += count_valid(idx + 2)

        return answer

    return count_valid(0)
```

 - 앞서 말했듯 중복되는 부분이 있을 수 있기 때문에 (거의 무조건 있다고
   보면 된다) 기존 계산 결과를 메모아이제이션 하기 위해서 `@cache`
   어노테이션을 붙여준다.
 - 제일 먼저 `len(s)`인 경우를 확인해준다. 이건 마지막 직전에서 문자
   두 개를 확인할 때 가능한 케이스인데, 이걸 먼저 해줘야 이후 로직에서
   인덱스가 아웃 바운드 되지 않는다.
 - 그 다음은 먼저 지금 인덱스가 `0`인지 확인한다. 이럼 가능한 경우가
   없기 때문이다.
 - 그리고 최종적으로 끝에 도달했으면, 1개가 가능하다고 리턴한다.
 - 개수를 셀 때에도 먼저 1개 문자가 가능한지를 누적하고, 그 다음 2개가
   가능하다면(`<= 26`) 2개를 누적한다.
