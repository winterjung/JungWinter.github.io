---
layout: post
title: 전문가를 위한 파이썬 - 1
tag: 책, 파이썬
---

`전문가를 위한 파이썬`(한빛미디어, 2016)을 읽으며 배운 짤막한 팁들을 정리한다.

## 1장 파이썬 데이터 모델

### 매직 메서드 `__getitem__()`을 통한 속성 접근  
`obj[key]`를 평가하기 위해 인터프리터는 `obj.__getitem__(key)`를 호출한다.

```python
class Deck:
    def __init__(self):
        self._cards = [("A", "Spades"), ("2", "Spades"), ...]  # 일부 코드 생략
    def __getitem__(self, index):
        return self._cards[index]
```

```python
>>> deck = Deck()
>>> deck[0]
("A", "Spades")
>>> deck[-1]
("K", "Clubs")
```

`__getitem__()`메서드에서 `[]`연산자를 사용하므로 슬라이싱도 자동으로 지원된다.


### `__repr__()`을 통해 객체를 문자열로 표현할 수 있다.  
구현을 하지 않으면 `Vector object at 0x10e100070`처럼 출력되지만  
구현하면 `Vector(1, 2)`처럼 표현할 수 있다.  

> 이 때, 포멧 문자열의 `%r`플레이스 홀더를 사용한다.  
> `Vector(1, 2)`와 `Vector("1", "2")`는 다르다.

또한, `__str__()` 메서드가 구현되어 있지 않으면 `__repr__()`을 호출한다.


## 2장 데이터 구조체

### 지능형 리스트와 가독성
리스트 컴프리헨션 혹은 리스트 내포 라고도 한다.

```python
>>> numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
>>> odds_str = []
>>> for i in numbers:
...    if i % 2 == 1:
...        odds.append(str(i))
>>> odds_str
['1', '3', '5', '7', '9']
```

위의 구문은 `map()`과 `filter()`, `lambda`를 써서 구현할 수 있지만 가독성이 매우 나쁘다.  
리스트 컴프리헨션을 사용하면 빠르고 읽기 좋게 만들 수 있다.

```python
>>> new_odds_str = [str(i) for i in numbers if i % 2 == 1]
>>> new_odds_str
['1', '3', '5', '7', '9']
```

### 튜플 언패킹

```python
>>> coordinates = (37.1234, 131.1234)
>>> latitude, longitude = coordinates
>>> latitude
37.1234
>>> longitude
131.1234
```

`b, a = a, b`도 튜플 언패킹을 이용한 것이다.  
2명의 공대생이 자리를 바꾸기 위해선 3개의 의자가 필요하지만  
파이썬에서는 2개로도 충분하다. (`XOR`을 이용한 방법은 논외로 치자.)

### `*`을 사용해 초과항목 잡기  
함수를 선언할 때 `*args`, `**kwargs`처럼 사용하기도 하지만 여기선 보다 일반적인 사용법을 말한다.

```python
>>> a, b, *rest = range(5)
>>> a, b, rest
(0, 1, [2, 3, 4])
>>> a, b, *rest = range(3)
>>> a, b, rest
(0, 1, [2])
>>> a, b, *rest = range(2)
>>> a, b, rest
(0, 1, [])
>>> *rest, a, b = range(5)
>>> rest, a, b
([0, 1, 2], 3, 4)
```

### 슬라이싱
슬라이싱을 사용할 때 `s[a:b:c]`에서 `a`는 시작점, `b`는 중단점, `c`는 보폭(stride)이다.  
이때 `b`는 자기자신을 포함하지 않는다.  
그렇기에 길이계산의 이점(슬라이싱 되는 길이는 `b`-`a`)이 있고, 변수를 사용한 시퀀스 분할에도 이해가 쉽다.

```python
>>> l = [1, 2, 3, 4, 5, 6]
>>> x = 2
>>> l[:x]
[1, 2]
>>> l[x:]
[3, 4, 5, 6]
```

### 슬라이스 객체를 만들면 파싱에 더 도움이 된다.
```python
>>> data = """
0.....6..................25..........
2016  Fluent Python          ￦55,000
2015  Effective Python       ￦24,000
"""
>>> YEARS = slice(0, 6)
>>> TITLE = slice(6, 25)
>>> PRICE = slice(25, None)
>>> items = data.split("\n")[2:]
>>> for item in items:
...    print(item[TITLE], item[PRICE])
Fluent Python          ￦55,000
Effective Python       ￦24,000
```

`item[6:25]`, `item[25:]` 이렇게 사용했다면 각각의 슬라이스가 매직넘버가 되어 무엇을 의미하는지 나만아는 상태였을 것이다.

### 다차원 슬라이싱  
`NumPy`의 2차원 `numpy.ndarray`배열은 `a[m:n, k:l]` 구문이 가능하다.  
일반 파이썬의 2차원 리스트는 당연히 안된다.  
참고로 `a[i, j]`를 평가하기 위해 `i`, `j`를 튜플로 묶어 `a.__getitem__((i, j))`를 호출한다.  
그렇기에 일반 2차원 리스트에 `a[1, 2]`를 하면 `TypeError: list indices must be integers, not tuple`에러 메시지가 뜬다.

### 슬라이스 할당
```python
>>> l = list(range(10))
>>> l
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
>>> l[2:5] = [20, 30]
[0, 1, 20, 30, 5, 6, 7, 8, 9]
>>> l[2:5] = 10
TypeError: can only assign an iterable
>>> l[2:5] = [10]
[0, 1, 10, 5, 6, 7, 8, 9]  # 하나로 뭉개진다.
```

일반 파이썬에서는 이렇게 동작하지만 `NumPy`에서 `l[2:5] = 10`을 하면 에러가 나지않고 브로드캐스팅을 통해 `[0, 1, 10, 10, 10, 5, 6, 7, 8, 9]`가 된다. 주의할 것

### 리스트의 리스트
리스트 컴프리헨션을 이용해서 2차원 리스트를 만들 때 참조를 주의해야한다.

```python
# 올바른 사용법
>>> board = [["_"] * 3 for i in range(3)]
>>> board
[['_', '_', '_'], ['_', '_', '_'], ['_', '_', '_']]
>>> board[1][2] = "X"
>>> board
[['_', '_', '_'], ['_', '_', 'X'], ['_', '_', '_']]
```

```python
# 잘못된 사용법
>>> weird_board = [["_"] * 3] * 3 
>>> weird_board
[['_', '_', '_'], ['_', '_', '_'], ['_', '_', '_']]
>>> weird_board[1][2] = "O"
>>> weird_board
[['_', '_', 'O'], ['_', '_', 'O'], ['_', '_', 'O']]
```

위의 코드는 실질적으로

```python
row = ["_"] * 3
board = []
for i in range(3):
    board.append(row)
```

와 같기에 세 개의 행이 모두 동일한 객체를 참조하게 된다.

#### 쓰지 못한 것들
Atom 에디터로 글을 쓰는데 너무 느려터졌다. 프로젝트 폴더도 날라가고  
PyCharm으로 바꾸든가 VS Code를 써보든가 해야겠다.  
하여튼 `+= 실행 결과`에 대한 것과 `list.sort()`와 `sorted()`에 관한 것 두 개를 덜썼다.  
