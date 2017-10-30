---
layout: post
title: 마음찍어둔 문서 읽기 - 02
tag: article, python, tech
---

## 트위터 마음함에 있는 아티클 리뷰
오늘의 성과: 924개 → 894개

## 리뷰
### [스칼라의 도입을 회고하며](https://medium.com/rainist-engineering/%EC%8A%A4%EC%B9%BC%EB%9D%BC%EC%9D%98-%EB%8F%84%EC%9E%85%EC%9D%84-%ED%9A%8C%EA%B3%A0%ED%95%98%EB%A9%B0-d491125abeb9)
9XD해커톤 때 스칼라를 보니 뭔가 파이썬스럽다는 느낌(사실 비슷한 부분은 `def`밖에 없는데)을 받았었다. 평소에 스칼라 같은 강타입 언어를 써보고 싶었으나 실용적인가 라는 의문에 미뤄뒀었는데 이 글을 읽어보니 나름 쓸만하다라는 느낌을 받았고 `<-`나 `=>` 같은 문법이 뭔가 멋져보인다. 파이썬에도 `map`, `filter`, `reduce`가 있는데 굳이 스칼라를 써야할까 라는 망설임은 있다. 견고한 건 좋아보이는데 일단 소마부터 끝내고...

### [Design Patterns: Strategy Pattern](https://dev.to/henriguy/design-patterns-strategy-pattern)
나는 디자인 패턴을 하나도 모르는데 그래서 그런지 이런 글을 읽을 때마다 재미있다. 평소에 쓰던게 이런 패턴이구나 싶기도 하고 이렇게도 쓸 수 있겠구나 싶기도 하고. 학식알리미를 만들면서 아 이렇게 설계하는건 뭔가 아닌거 같은데 스탠다드를 모르겠네 할 때가 많아서 그런지 알수록 재밌다. 전략 패턴은 뭔가 부품을 조립하는 느낌이기도 하네. 그래서 컴포지트 패턴을 찾아봤더니 얘는 인스턴스를 조립하는 느낌. 전략 패턴은 음 좀 can-do를 조립하는 느낌.

### [Surgical Time Tracking in Python](https://blog.sicara.com/profile-surgical-time-tracking-python-db1e0a5c06b6)
매우 좋은 글이다! 나는 따로 프로파일러를 쓰지 않고 실행 시간을 추적할 필요가 있을 때 데코레이터 만들어서 때워버리는데 여기선 좀 더 발전시킨 코드를 보여준다. `cProfile`에 대해 조목조목 불편한 점을 말하는데 구구절절 나와 같은 의견이었고 데코레이터로 기본적인 time tracker를 만들고 그걸 partial time tracker로 업그레이드 하는 부분이 좋았다. 마지막으로 [pyflame](https://github.com/uber/pyflame)과 [flamegraph](https://github.com/brendangregg/FlameGraph)의 조합을 사용해 `pyflame -t python your_code.py | flamegraph > profile.svg`명령어로 프로파일된 결과를 svg로 보여주는데 매우 멋졌다. 다음에 쓸 일이 있다면 저 두개를 꼭 사용해봐야지.

### [logzero: Robust and effective logging for Python 2 and 3](https://github.com/metachris/logzero)
옛날에 보고 오 로깅툴이 깔끔하고 이쁘다 써봐야지 라고 생각만 하다가 결국 까먹어 버렸는데 다시 생각났다. 파이썬의 기본 `logging`모듈보다 이쁘고, 간결하다. 이점만으로도 쓸 이유로는 충분할 듯 싶다.

<div markdown="0" align="center">
    <img src="https://raw.githubusercontent.com/metachris/logzero/master/docs/_static/demo_output.png" width="300px">
</div>

### [An introduction to Vue.js - Chapter 1 - Basic Setup](https://dev.to/neradev/an-introduction-to-vuejs---chapter-1---basic-setup)
소마 프로젝트를 진행하면서 프론트를 Vue로 만들까 했었으나 결국 안만들기로 했었는데 나중에 프론트 앱을 만든다면 봐도 좋을 것 같다.

### [A guide to logging in Python](https://opensource.com/article/17/9/python-logging)
파이썬의 로깅모듈에 대해 파고들어가는 글. 탐구정신이 느껴지는 좋은 글이다. 하나하나 단계를 거쳐 나가며 프로세스 모델을 추가해 나가는 느낌이 좋다. 덤으로 안티 패턴에 대한 얘기도 있는데 `.format()`보다는 `%s, argument`를 쓰고 에러가 났을 때 `except Exception as error`로 받아 `%s, error`를 쓰지 말고 `exc_info=True`를 로 전달하라한다. 커스텀 formatter를 만들어서 json으로 로깅하는 부분도 기억해두면 좋을 것.

<div markdown="0" align="center">
    <img src="https://opensource.com/sites/default/files/u128651/6-kd6eiyr.jpg" width="300px">
</div>

### [당신이 AWS 계정을 만들고 가장 먼저 해야 할 일 들과 하지 말아야 할 일 들](http://www.awskr.org/2017/01/your-aws-first-days-todo-list/)
돈도 돈이고 귀찮아서 개인적으로는 AWS를 사용하지 않고 IAM설정만 해뒀는데 다음에 백지부터 AWS계정을 만들고 사용해야할 일이 있다면 제일 먼저 읽어봐야할 글이다.

### [The PI fractal](https://friendlyfieldsandopenmaps.com/2017/09/18/the-pi-fractal/)
출발은 단순히 결과 이미지를 소개하는 [트윗 타래](https://twitter.com/udaqueness/status/910705735742492672)였다. 여기선 룰과 시작, 그리고 경과에 따라 점점 pi를 표시하는 오토마타를 소개시켜주는데 원리를 알려주진 않는다. 유투브를 보고 레딧과 아티클을 봤더니 유전 알고리즘을 통해 이런 규칙을 학습시킨 결과물이라 한다! 이걸 보고 너무 아름답다고 생각했다. fitting하고 싶은 이미지를 goal로 설정해 규칙을 학습시킨다. [유투브 동영상](https://www.youtube.com/watch?v=dPozHBz6Fqw), [레딧](https://www.reddit.com/r/math/comments/70vzlx/the_pi_fractal_lots_more_information_in_the/), [레딧 유저의 구현 코드](https://gist.github.com/skeeto/bfa7936c2b37d27ea06918847b4739bd)

<div markdown="0" align="center">
    <img src="https://pbs.twimg.com/media/DKN6I3kU8AAC5xh.jpg" width="300px">
</div>

### [Storybook + Vue로 UI 컴포넌트 문서 만들기](https://medium.com/@changjoopark/storybook-vue%EB%A1%9C-ui-%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8-%EB%AC%B8%EC%84%9C-%EB%A7%8C%EB%93%A4%EA%B8%B0-38c21414d1bb)
vue도 vue지만 [storybook](https://github.com/storybooks/storybook)이라는 도구가 매우 멋지다. 인터랙티브하게 UI컴포넌트를 만들고 테스팅할 수 있는 도구인데 얘도 프론트할 때 써봐야겠다.

### [Optimising the front end for the browser](https://dev.to/sanjsanj/optimising-the-front-end-for-thebrowser)
내용보다는 내용을 표현하는 이미지 구성 기법에 눈이 간다. 나는 flow나 점진적으로 살이 덧붙여나가지는 표현 방법을 좋아하는데 이에 딱 들어맞는다. 특히 `How the browser makes network calls`단락에서 branch로 표현되는 이미지는 무척 인상적이다!

<div markdown="0" align="center">
    <img src="https://github.com/sanjsanj/otfe-assets/blob/master/otfe-crp-4-4.png?raw=true" width="600px">
</div>

### [How to Docker Compose a developer environment: an open source example](https://dev.to/danlebrero/how-to-docker-compose-a-developer-environment-an-open-source-example-72k)
도커글을 볼 때 마다 느끼는점, 정말 좋은 기술이긴 한데 써보자니 의욕이 안난다... 뭔가 강제로 도커와 도커 컴포즈를 써야만 하는 상황이 없을까? 추상적인 이점은 알겠는데 내 프로젝트와 도커가 결합되면 어디가 편해지고 어떻게 코드가 변하는지 감이 안오다보니 머리속에서만 맴돌고있다. [도커를 사용해야하는 이유](https://dev.to/danlebrero/why-to-docker-compose-your-developer-environment)를 보면 일관된 개발 환경을 구성할 수 있다를 말해주는데 회사를 다녀본 것도 아니고 다른 사람과 협업을 많이 해본것도 아니고 이 점 때문에 불편해본 적이 없는게 가장 큰 문제인 것 같다.

### [Beautiful terminal spinners in Python](https://github.com/ManrajGrover/halo)
완전 내취향이다! 노드의 빙글빙글 돌아가는게 부러웠는데 이거 가지고 놀아봐야겠다. PR열린 것들 보니까 with context지원이나 keyword argument지원 내용이 있는데 merge되면 더 좋아질 것 같다. 이슈보니 윈도우는 아직 지원이 안된다고 하는데 어떻게 고쳐볼 수 없을까 싶다.

<div markdown="0" align="center">
    <img src="https://raw.githubusercontent.com/ManrajGrover/halo/master/art/doge_spin.gif">
</div>

<div markdown="0" align="center">
    <img src="https://raw.githubusercontent.com/ManrajGrover/halo/master/art/persist_spin.gif">
</div>

## TODO
- `pyflame`+`flamegraph`와 [what-studio의 `profiling`](https://github.com/what-studio/profiling) 사용해보기
- `logzero` 사용해보기
- 도커와 도커 컴포즈로 서비스 구성해보기
- halo 윈도우 서포팅 찾아보기
