---
layout: page
title: Tips
permalink: /algorithm/tips
parent: Algorithm
---


{: .no_toc }
# Tips
## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

## Remove n-th Node From End of List

 Pioneer가 n 만큼 먼저 땅을 밝히고, 이후에 Follower와 Pioneer가
 발맞추어 리스트의 끝까지 도달하면 된다. 이때 Pioneer를 한 칸 덜
 보내야 Follower가 삭제할 노드 바로 직전에 위치하도록 만들 수 있다.

```python
def remove_nth_from_end(head, n):
    pioneer = head
    follower = head

    while n > 0:
        n -= 1
        pioneer = pioneer.next

    if pioneer is None:
        # remove head
        return head.next

    while pioneer.next:
        pioneer = pioneer.next
        follower = follower.next

    # now, follower is right before the node that needs be deleted
    follower.next = follower.next.next

    return head
```


## DFS, Cycle, Topological Ordering

### Condition of Cycle

| | `visiting = False` | `visiting = True` |
| --- | --- | --- |
| `visited = False` | 아직 방문하지 않음 | **싸이클** |
| `visited = True` | 불가능한 경우 | 탐색이 끝남 |

### Topological Ordering
 DFS에서 노드에 대한 방문을 **완료** 했다는 의미는 즉 이 친구는
 Topological Ordering 에서 제일 나중에 방문해야 한다는 뜻이다. 따라서
 방문을 완료한 순서대로 리스트든 큐든 스택이든 차례로 넣으면 이게 곧
 거꾸로 된 Topological Ordering 이다. 그러므로,
  - 그래프를 만들 때부터 source와 sink를 거꾸로 한 다음 그냥 리턴하거나,
  - 리턴하기 직전에 뒤집어주면 된다.

``` python
class Graph:
    def __init__(self, n):
        self.node_map = {}
        self.node_set = set(range(n))  # edge가 없는 노드가 있을 수 있음

    def add_edge(self, src, snk):
        self.node_set.add(src)
        self.node_set.add(snk)

        sink_set = self.node_map.get(src, None)
        if sink_set:
            sink_set.add(snk)
        else:
            self.node_map[src] = {snk}

    def topological_ordering(self):
        visiting = set()
        visited = set()
        order = []

        def dfs(node):
            if node in visited:  # node_set 이 entry 이므로 진입하자마자 visited 체크를 해줘야 한다.
                return

            visiting.add(node)
            for succ in self.node_map.get(node, set()):
                if succ in visiting:  # 정확히 싸이클 케이스
                    raise TypeError("has cycle")
                if succ not in visited:  # 아직 방문 완료가 아닐 때에만 추가로 탐색한다
                    dfs(succ)

            # node 에 대한 방문을 완료했으므로, visiting/visited 처리를 완료하고 order에 넣는다.
            visiting.remove(node)
            visited.add(node)
            order.append(node)

        try:
            for node in self.node_set:
                dfs(node)
        except TypeError:
            return []

        order.reverse()
        return order
```


## Trie

``` python
class TrieNode:
    def __init__(self, c):
        self.c = c
        self.children = [None] * 26
        self.count = 0
        self.is_word = False

    def __getitem__(self, key):
        return self.children[ord(key) - ord('a')]

    def __setitem__(self, key, value):
        self.children[ord(key) - ord('a')] = value
```

 - `ord(c)` returns the ASCII code of the character `c`

``` python
def build_trie(root, str):
    node = root
    for c in str:
        if node[c]:
            cur = node[c]
        else:
            cur = TrieNode(c)
            node[c] = cur
        cur.count += 1
        node = cur
    node.is_word = True
```

 - `__getitem__` 랑 `__setitem__` 를 오버라이딩해서 `children`에 바로
   접근하지 않고 인덱스처럼 쓸 수 있음



## Palindrome

 Palindrome Substring (Subsequence가 아님!) 을 찾는 방법: 중간에서부터
 확인한다.

``` python
def count_from_center(string, left, right):
    if not string or left > right:
        return 0

    num_of_palindrome = 0
    while left >= 0 and right < len(string) and string[left] == string[right]:
        left -= 1
        right += 1
        num_of_palindrome += 1
    return (right - left - 1)
    # or, return num_of_palindrome for the number of palindromes found
```

 단순한 투 포인터를 이용한 기법이다. `left`, `right` 둘 다
 인덱스임. 이 함수를 활용하면 다음과 같이 "가장 길이가 긴
 palindrome"을 찾을 수 있다.


``` python
def longest_palindrome(string):
    if not string or len(string) < 1:
        return ''

    start, end = 0, 0  # index
    for i in range(len(string)):
        len_cand = max(count_from_center(string, i, i),
                       count_from_center(string, i, i + 1))
        if len_cand > (end - start):
            start = i - (cand - 1 ) // 2
            end = i + cand // 2

    return string[start:end + 1]
```

 - 현재 인덱스를 *중간*으로 시작해서 `right`를 `i`와 `i + 1`로 두 번씩
   호출해줘야 `aba`와 `abba` 같은 palindrome을 모두 처리할 수 있다.
 - `start`, `end`는 인덱스고 `len_cand`는 찾아낸 길이이므로 `end -
   start` 보다 길이가 크면 최대값을 업데이트하면 된다.


## Bisection

### Upper Bound and Lower Bound

 정렬된 리스트 `arr`과 찾고자 하는 키 값 `x`가 있을 때,
 - Upper Bound: `x` 보다 큰 값이 처음 나오는 위치
 - Lower Bound: `x` 보다 크거나 같은 값이 처음 나오는 위치

 예를 들어, `arr = [1, 3, 3, 5, 7]` 이 있고 `x = 3` 을 키 값으로 하면,
 - Upper Bound: 처음으로 커지는 값(`5`)이 나오는 위치인 `3`
 - Lower Bound: 처음으로 크거나 같은 값(첫번째 `3`)이 나오는 위치인
   `1`

 가 되고, 이렇게 찾은 위치에 값을 삽입하면 한 칸씩 뒤로 쭉 밀리면서
 삽입된다. 그러니까, Upper Bound에는 `x` 보다 크거나 같은 값을 넣을 수
 있고, Lower Bound에는 `x` 보다 작거나 같은 값을 넣을 수 있다.

 Pythonic 하게 설명하면,
 - Upper Bound를 ub(index)라고 한다면, `arr[:ub] <= x < arr[ub:]` 를
   만족한다.
 - Lower Bound를 lb(index)라고 한다면, `arr[:lb] < x <= arr[lb:]` 를
   만족한다.

 이걸 코드로 옮기면 다음과 같다.

``` python
def bisect_right(arr, x, low=0, high=None):
    """
    The return value idx is such that all element in arr[:idx] have elt <= x,
    and all elt in arr[idx:] have x < elt.
    So if x already appears in the list, arr.insert(x) will insert just after the rightmost
    x already there.
    """

    if low < 0:
        raise ValueError('low must be positive')

    if high is None:
        high = len(arr)

    # [low, high)
    while low < high:
        mid = (low + high) // 2

        if x < arr[mid]:
            high = mid
        else:
            low = mid + 1
    return low

```

``` python
def bisect_left(arr, x, low=0, high=None):
    """
    Return the index where to insert item x in a list arr, assuming arr is sorted.

    The return value idx is such that all element in arr[:idx] have elt < x,
    and all elt in arr[idx:] have x <= elt.
    So if x already appears in the list, arr.insert(x) will insert just after the leftmost
    x already there.
    """

    if low < 0:
        raise ValueError('low must be positive')

    if high is None:
        high = len(arr)

    # [low, high)
    while low < high:
        mid = (low + high) // 2

        if x <= arr[mid]:
            high = mid
        else:
            low = mid + 1
    return low
```

## Binary Search and Rotated Sorted Array

 정렬된 리스트(배열)에서 `k` (`0 <= k < len(arr)`) 번째 인덱스를
 기준으로 회전을 시킨 경우, 어디서 회전되었는지를 찾고 (피벗) 이 값을
 이용해서 이분탐색을 하면 `O(log N)` 만에 찾을 수 있다.

 참고로, 피벗을 찾은 다음 `sorted_arr = arr[pivot:] + arr[:pivot]`
 이렇게 정렬된 리스트를 복구해서 검색해도 말은 되지만, 복사 연산자
 때문에 `O(N)`을 한번 겪게 되어서 사이즈가 커지면 느려진다. 작으면
 별로 문제 안됨.

``` python
def search_from_rotated(nums, tofind):
    # 1. find pivot index (the smallest value)
    low = 0
    high = len(nums) - 1  # this is tricky - high is an index here

    while low < high:
        mid = low + (high - low) // 2  # same as (low + high) // 2, but avoid overflow
        if nums[mid] > nums[high]:
            # rotated in somewhere mid..high
            low = mid + 1
        else:
            # rotated in somewhere low..mid
            high = mid

    pivot = low  # the index of the smallest value

    # 2. now, binary search with this info
    low = 0
    high = len(nums)  # here, the range is half open: [low, high)

    # find correct range
    if nums[pivot] <= tofind <= nums[high-1]:
        # tofind is somewhere pivot..high
        low = pivot
    else:
        # tofind is somewhere low..pivot
        high = pivot

    while low < high:
        mid = low + (high - low) // 2
        if nums[mid] == tofind:
            return mid
        if tofind > nums[mid]:
            low = mid + 1
        else:
            high = mid

    return -1
```

 - 처음에 피벗 인덱스를 찾을 때는 `high` 역시 인덱스로
   쓰였다. `high`를 half-open 으로 둬볼려고 했는데, `1`씩 빼거나
   더해줘야 되서 코드가 더 복잡해져서 그냥 저렇게 하는게 깔끔하다.
 - 피벗을 찾은 이후에는 위 코드처럼 직접 이분 탐색을 구현해도 되고
   (여기서는 half-open 으로 구현), 아니면 아래와 같이 `bisect`를
   활용해도 된다.

 ``` python
    ... # search pivot
    # find Lower Bound to find equal value
    bi = bisect.bisect_left(nums, tofind, low, high)
    if bi < len(nums) and nums[bi] == tofind:
        return bi
    else:
        return -1

 ```

  - `bisect` 를 활용할 때에는 Lower Bound를 찾는게 편한데, 왜냐하면
    위에서 설명했듯이, 찾고자 하는 값 보다 "크거나 같은 값"이 처음
    나오는 위치를 찾아주기 때문이다. 그래서 `bisect_left` 리턴 값을
    곧바로 인덱스로 쓸 수 있다. 만약 Upper Bound로 찾았다면, 이는 "큰
    값"이 처음으로 나오는 위치 이므로, `bisect_right` (또는 그냥
    `bisect`) 리턴 값에서 1을 빼줘야 정확한 인덱스가 된다.
  - 추가로, `bisect`는 "값을 적절하게 삽입할 위치"를 찾아주기 때문에,
    실제로 리턴한 인덱스에 정말 찾고자 하는 값이 있는지 한번 더
    확인해야 한다.



## How to generate all possible substrings with specific length

``` python
def all_substrings(text: str, k: int):
    return set(text[i:i + k] for i in range(0, len(text) - k + 1))
```

 - Use slice operator. `list[start:end:stride]` means from `start` to
   `end` with each step `stride` (not including `end` because it is a
   right half-open interval). Thus, `text[i:i + k]` means a substring
   in `[i, i + k)` and this is a substring that has length `k`.
 - `range(start, end, stride)` is similar to slice. It generates a
   range `[start, end)` with step `stride`. Thus, `range(0,
   len(text) - k + 1)` will generate `[0, 1, ..., len(text) - k]`. We
   only need at most `len(text) - k` index because otherwise it
   overflows.
 - Accumulate those substrings as a set to remove duplcations.


## Comprehensions

``` python
[exp for x in sequences if ...]  # list
{exp for x in sequences if ...}  # set
{key:value for x in sequences if ...}  # dict
```

## Index of Sequences

 Python can have a negative index.
 - If an index is positive, it starts from `0` at the left most of the
   sequences.
 - If an index is negative, is starts from `-1` at the right most of
   the sequences.

``` python
sequences = ['a', 'b', 'c']
#             0    1    2    # positive
#            -3   -2   -1    # negative
```

## PowerSet

``` python
def powerset(nums):
    """
    e.g. nums = [1,2] then returns
    [[1,2],
     [1],
     [2],
     []
    ]
    """
    result = []
    partial = []

    def recurse(idx):
        if idx == len(nums):
            # finish
            result.append(partial[:])  # must be copied
            return

        partial.append(nums[idx])  # pick this item
        recurse(idx + 1)
        partial.pop()  # not pick this item
        recurse(idx + 1)

    recurse(0)
    return result
```

 - Simply use `seq[:]` to deepcopy a sequence.
