---
layout: post
title: 기술 문서 정리 - 05
tags: [article, python, 파이썬]
---

오랜만에 읽은 글들을 정리해보려 한다.

## 파이썬

### [Token-Based Authentication With Flask]

플라스크를 사용해 JWT구현할 때 참고하면 좋은 글. Expire 된 토큰의 부정 사용을 방지하기 위해서 블랙리스트를 활용한다. 별로 우아해보이진 않지만 어쩔 수 없는 듯.

### [VSCode Python settings reference]

VSCode의 파이썬 세팅에 관한 레퍼런스. AutoComplete Setting을 통해 커스텀 패키지나 필요한 경로를 직접 지정해줄 수 있다. Linting settings에서 mypy, pydocstyle을 지정해줄 수 있다.

## Git

대부분 Git Hook과 관련된 글인데 트위터에서 커밋 전에 검사할 항목을 적어둔 포스트잇을 보곤 훅으로 어떻게 할 수 있을까 찾다보니 이렇게 됐다. 이슈번호 붙이기, 테스트 통과, 디버깅용 코드, 커밋 메시지 prefix, 오탈자, 최신 상태로 pull & rebase였는데 아마 훅들로 다 처리가 될 것 같다.

### [Git Hooks]

레퍼런스 비슷한 문서인데 각 hook 별로 예제와 인자들을 잘 정리해뒀다.

### [Writing Git Hooks Using Python]

글에서는 post-commit hook을 설정하는데 아래와 같은 파이썬 스크립트를 만들어서 활용했다. `subprocess`를 활용하면 될 듯.

```python
#!/usr/bin/python

import subprocess
import humanhash

# get the last commit SHA and print it after humanizing it
# https://github.com/zacharyvoase/humanhash
print humanhash.humanize(
    subprocess.check_output(
        ['git','rev-parse','HEAD']))
```

### [Git Hooks with Python]

- `pre-commit`: 코드 스타일, 테스트, 빌드를 체크해 볼 수 있다. (`flake8` 혹은 `pylint` 등) 0이 아닌 값을 반환하면 커밋 중단
- `prepare-commit-msg`: 기본 커밋 메시지를 수정해서 제공할 수 있다. 0이 아닌 값을 반환하면 커밋 중단
- `commit-msg`: 커밋 메시지가 작성된 후 실행됨. 커밋 메시지에 대한 파일 참조가 전달됨.
- `post-commit`: 커밋이 완전히 완료되면 실행됨. 0이 아닌 값을 반환해도 **중단 안됨**. 때문에 주로 통보 목적으로 사용

글에서 `commit-msg`를 예제로 보여주는데 커밋 메시지가 특정 길이보다 길고 정규식을 통과하는지 검사해본다. `sys.argv[1]`로 커밋 메시지가 작성된 파일 이름이 전달된다.

> 위에서 살펴본 hook은 클라이언트상에서 동작하지만 [Git맞춤 - 정책 구현하기]를 통해 서버에서 정책을 강제할 수 있다.

### [Hooking Custom Functionality into the Git Pipeline]

약간의 hack을 통해 interactive hook을 만드는 점이 흥미롭다.

```python
#!/usr/bin/python

import random
import sys

x = random.randint(1,10)
y = random.randint(1,10)

# This is required because git hooks are run in non-interactive
# mode. You aren't technically supposed to have access to stdin.
# This hack works on MaxOS and Linux. Mileage may vary on Windows.
sys.stdin = open('/dev/tty')

result = input("What is %d * %s:" % (x, y))

if int(result) == x * y:
  sys.exit(0)
else:
  print "INCORRECT!"
  sys.exit(-1)
```

[Token-Based Authentication With Flask]: https://realpython.com/blog/python/token-based-authentication-with-flask/
[Writing Git Hooks Using Python]: https://dzone.com/articles/writing-git-hooks-using-python
[Git Hooks with Python]: https://devslash.net/easy-git-hooks-with-python/
[Git맞춤 - 정책 구현하기]: https://git-scm.com/book/ko/v2/Git%EB%A7%9E%EC%B6%A4-%EC%A0%95%EC%B1%85-%EA%B5%AC%ED%98%84%ED%95%98%EA%B8%B0#r_an_example_git_enforced_policy
[Hooking Custom Functionality into the Git Pipeline]: https://courses.csail.mit.edu/6.S194/13/lessons/03-git/adding-custom-hooks-to-git.html
[Git Hooks]: https://www.atlassian.com/git/tutorials/git-hooks
[VSCode Python settings reference]: https://code.visualstudio.com/docs/python/settings-reference
