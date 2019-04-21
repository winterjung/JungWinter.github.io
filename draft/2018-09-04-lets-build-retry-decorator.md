---
layout: post
title: retry 데코레이터 만들기
tags: [python, practice]
---

- 기본형
- 최대 시도 횟수 지정
- 에러 유형
- 핸들러 등록
- backoff 알고리즘 제공
- jitter 적용
- give up 조건설정

```py
def retry(max_tries, error):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            # Set the default result
            result = None
            for tries in range(1, max_tries + 1):
                try:
                    result = func(*args, **kwargs)
                    break
                except error:
                    seconds = 2 ** tries
                    time.sleep(seconds)
                    continue
                except:
                    break
            return result
        return wrapper
    return decorator
```

```py
def retry(max_tries, handler):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            # Set the default result
            result = dict()
            for tries in range(1, max_tries + 1):
                try:
                    result = func(*args, **kwargs)
                    break
                except default_excepts as e:
                    seconds = 2 ** tries
                    give_up = handler(e, seconds)
                    if give_up:
                        break
                    continue
            return result
        return wrapper
    return decorator
```
