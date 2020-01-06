---
layout: post
title: 파이썬에서 부동 소수점 오차 해결하기
redirect_to: https://winterj.me/floating-point-in-python
tags: [파이썬, python, 팁]
---

### 부동 소수점의 문제
`float`자료형을 쓸 때, 특히 `if`를 이용해 조건을 판단하거나 값을 확인할 때는 항상 주의해야한다.
이를 간과하면 아래처럼 언뜻 보기에는 괴상한, 버그같은 일이 일어난다.

```python
>>> 0.1 * 3 == 0.3
False
>>> 1.2 - 0.1 == 1.1
False
>>> 0.1 * 0.1 == 0.01
False
```
어렴풋이 부동 소수점이라 그래 정도로만 알고 있었는데 이번 기회에 그 이유와 파이썬에서의 해결방법을 알아보았다.

### 부동 소수점의 표현 방식
컴퓨터에서 부동 소수점 숫자들은 2진분수로 표현되기에 꽤 많은 특정 값들은 정확히 표현될 수 없다.
일반적으로 입력하는 10진 부동 소수점 숫자는 2진 부동 소수점 숫자로 근사된다.
예를 들어 10진수 0.1은 2진분수로 정확하게 표현될 수 없고 `0.0001100110011001100110011001100110011001100110011...`처럼 무한히 반복되기에 특정 비트에서 멈추고 근사값을 얻는다.
0.1의 경우는 `3602879701896397 / 2 ** 55`이며 0.1에 가깝지만 정확히 동일하지는 않다.

```python
>>> decimal.Decimal(3602879701896397 / 2 ** 55)
Decimal('0.1000000000000000055511151231257827021181583404541015625')
>>> decimal.Decimal(0.1)
Decimal('0.1000000000000000055511151231257827021181583404541015625')
```

파이썬은 이를 표현할 때 정확히 표현하지 않고 10진수 근사값만을 표시한다.
0.1에 대해 실제 십진수 값을 출력한다면,

```python
>>> 0.1
0.1000000000000000055511151231257827021181583404541015625
```

이지만 파이썬은 반올림 된 값을 표시한다.

```python
>>> 1 / 10
0.1
```

이런 부동 소수점의 한계는 파이썬뿐만 아니라 부동 소수점을 지원하는 모든 언어에서 찾아볼 수 있다. (아니라면 알려주세요!)

### 대책
`math.fsum()`, `round()`, `float.as_integer_ratio()`, `math.is_close()` 함수 혹은 다른 방법을 통해서 실수를 방지할 수 있다.


#### `math.fsum()`
`math`모듈의 `fsum(iterable)`를 이용해 두 개 이상의 부동 소수점 합계와 관련된 반올림 오류 누적을 제거하여 정밀도 손실을 방지할 수 있다.
[math.fsum 문서](https://docs.python.org/3/library/math.html#math.fsum)

```python
>>> sum([.1, .1, .1, .1, .1, .1, .1, .1, .1, .1])
0.9999999999999999
>>> fsum([.1, .1, .1, .1, .1, .1, .1, .1, .1, .1])
1.0
```

다만 `fsum()`은 중간 중간 누적된 오류를 해결하기 위함이지 정확한 부동 소수점 연산을 위한게 아니다보니 유명한 `0.1 + 0.2`처럼 항상 원하는 결과를 기대할 수는 없다.
[스택오버플로우 : 왜 fsum은 부정확한가 ](http://stackoverflow.com/questions/34650535/python2-math-fsum-not-accurate)
[참고 사이트 : 0.30000000000000004.com](http://0.30000000000000004.com/)

```python
>>> fsum([0.1, 0.2])
0.30000000000000004
```


#### `round()`
파이썬에서 글로벌 함수로 있는 `round()`를 이용해 반올림 해줌으로써 해결할 수도 있다.

```python
>>> 0.1 + 0.1 + 0.1 == 0.3
False

>>> round(0.1, 1) + round(0.1, 1) + round(0.1, 1) == round(0.3, 1)
False
```

`decimal.Decimal(round(0.1, 1))`로 확인해보면 `Decimal('0.1000000000000000055511151231257827021181583404541015625')` 이기 때문에 사전 `round()`는 도움이 되지 않는다.

```python
>>> round(0.1 + 0.1 + 0.1, 10) == round(0.3, 10)
True
```

여기서 주의해야 할 점이 있다.
우리는 보통 학교에서 0, 1, 2, 3, 4는 내림, 5, 6, 7, 8, 9는 올림으로 배웠는데
컴퓨터에서는 반올림 방식에 따라 5가 내림이 되기도하고 올림이 되기도 한다.

```python
>>> round(0.125, 2)
0.12
>>> round(0.135, 2)
0.14
```

반올림 방식이 무엇인지는 `deciaml.getcontext()`로 확인해 볼 수 있다.

```python
>>> import decimal
>>> decimal.getcontext()
Context(prec=28, rounding=ROUND_HALF_EVEN, Emin=-999999, Emax=999999, ...)
```

`ROUND_HALF_EVEN` 방식은 짝수에서는 5일 때 내림, 홀수에서는 5일 때는 올림하는 방식이다.
다른 방식으로는

- `ROUND_CEILING`
- `ROUND_DOWN`
- `ROUND_FLOOR`
- `ROUND_HALF_DOWN`
- `ROUND_HALF_UP`
- `ROUND_UP`
- `ROUND_05UP`

들이 있으며 `decimal.getcontext().rounding=ROUND_HALF_UP`처럼 변경할 수 있다.
[참고 사이트](https://blog.udemy.com/python-round/)


#### `float.as_integer_ratio()`
파이썬 `float`자료형에는 `as_integer_ratio()`함수가 내장되어 있다.

```python
>>> x = 3.141592
>>> x.as_integer_ratio()
(3537118140137533, 1125899906842624)
>>> x == 3537118140137533 / 1125899906842624
True
```

연산의 오차보정용 이라기 보다는 원래 값을 손실 없이 다시 만드는 데 유용하다.


#### `math.is_close()`
파이썬 3.5에서 새로 추가된 `math`모듈의 함수다. [math.is_close 문서](https://docs.python.org/3/library/math.html#math.isclose), [PEP 485](https://www.python.org/dev/peps/pep-0485/)
`isclose(a, b, rel_tol=1e-9, abs_tol=0.0)`같은 형태이며
`a`와 `b`는 비교할 값들, `rel_tol`은 relative tolerance, `abs_tol`은 absolute tolerance다.
실질적으로 위에 나온 방법들보다 제일 간단하고 직관적이다.
다만 파이썬 3.5 이상부터 내장되어 있으며 그보다 하위 버전에서는 직접 `is_close()`함수를 만들어 쓸 수 있다. [math.is_close()의 원형](https://github.com/PythonCHB/close_pep/blob/master/is_close.py)

```python
# 간단한 버전
def isclose(a, b, rel_tol=1e-09, abs_tol=0.0):
    return abs(a-b) <= max(rel_tol * max(abs(a), abs(b)), abs_tol)
```

#### 기타
가장 널리 사용되는 방법으로는 아마 `abs(a - b) <= allowed_error` 가 아닐까 싶다.
그 외에도 개인만의 노하우가 있다면 PR을 넣어주세요! [링크](https://github.com/JungWinter/JungWinter.github.io)

### 참고
[Python3 Doc : Floating Point Arithmetic: Issues and Limitations](https://docs.python.org/3/tutorial/floatingpoint.html)
[모든 컴퓨터 과학자가 알아야 할 부동 소수점의 모든것](https://app.box.com/s/vlij64akloz25k0fmk24o1fqxdndg8ie)
[스택오버플로우](http://stackoverflow.com/questions/5595425/what-is-the-best-way-to-compare-floats-for-almost-equality-in-python)
