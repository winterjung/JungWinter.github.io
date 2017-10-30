---
layout: post
title: 마음찍어둔 문서 읽기 - 01
tag: article, python, tech
---

## 목적
[트위터 마음함](https://twitter.com/res_tin/likes)에 담아뒀던 기술 문서나 각종 링크들이 1k를 향해 달려가는데 추석을 맞아 이를 처리하고자 함.

## 리뷰
- [모나드라는 단어를 사용하지 않고 모나드를 설명하기 위한 시도](https://twitter.com/heejongahn/status/911832361788043264): 처음에는 중첩-if할 필요 없이 그냥 앞단에서 null checking하면서 null이면 throw하면서 진행하면 중첩구조가 안생길것 같은데 했다가 읽을 수록 다른 얘기인걸 깨달음. 맥락 얘기가 나온 이후론 이해가 잘 안간다. 어렴풋이 맥락에 따라 진행을 어떻게 달리할 것인가, 축적된 계산 결과를 어떻게 담을 것이냐에 대한 얘기 같은데... 마지막에 뜬금없이 이게 모나드라는 얘기가 나온후론 더 오리무중 대체 뭘까? 모나드는 타입인가? 패러다임인가?
- [[번역] 자바스크립트에 대한 애정을 언어의 90%를 쓰레기통에 버리면서 다시 발견했던 과정](https://rhostem.github.io/posts/2017-09-how-i-rediscovered-my-love-for-java-script-after-throwing-90-of-it-in-the-trash/): 함수형 프로그래밍을 접하고 `var`, `for loop`, `if`, `switch`, `this`등을 버리는 과정을 얘기하는데 각각의 과정마다 읽어야할 아티클이 덧붙여있다... one-tap에 넣어둔 이 아티클들을 읽어야 한다는걸 깜빡하고 있었다. 얘는 남겨놔야지
- [Build a “Serverless” Ping Service + Status Page in 5 Steps with StdLib Scheduled Tasks](https://codeburst.io/build-a-serverless-ping-service-status-page-in-5-steps-with-stdlib-scheduled-tasks-6bdd164e67dc): `stdlib`라는 서비스가 있는데 이걸 이용해 health check 페이지를 만드는 튜토리얼, 뭔진 잘 모르겠는데 아마 js인거 같으니 쓸일은 없을 것 같고 `stdlib`라는 이름이 마음에 들다. 홈페이지가 머테리얼 디자인 느낌이 풍기는 것도 좋고, 나중에 사이트를 만든다면 이렇게
- [How to make beautiful data visualizations in Python with matplotlib](http://www.randalolson.com/2014/06/28/how-to-make-beautiful-data-visualizations-in-python-with-matplotlib/): 완전 마음에 드는 글, GIF로 summary를 보여주는데 무척 매력적이다. Less is more, 완벽이란 더할게 없을 때가 아니라 뺄게 없을 때다. 그래프를 만들 때 왜 이렇게 만들어야 한다는 주석을 친절히 달아놨다. 시각화를 할 때 참고해야지.
- [오픈 소스 프로젝트를 더 잘 릴리스하는 요령들](https://spoqa.github.io/2017/09/28/foss-release-tips.html): 나는 아직 이 지식이 크게 필요는 없는 듯, [리드 더 독스](https://readthedocs.org/)에 대한 설명은 처음 봤는데 이런거였구나 싶다. [스핑크스](http://www.sphinx-doc.org/en/stable/)가 문서화 플랫폼으로 독보적인 지위에 있구나. 다음에 문서화 할 땐 스핑크스로 해봐야지.
- [A Quick Introduction: Hashing](https://dev.to/aunyks/a-quick-introduction-hashing): hash에 관한 짧은 소개 글, 보고 비트코인이나 이더리움에서 `000000...`으로 시작하는 블록은 얼마나 만들기 힘들까 간단히 돌려봤는데 생각보다 엄청 힘든 작업이란걸 알았다.

```python
>>> import hashlib as hash
>>> def test(cond):
	i = 0
	while True:
		result = hash.sha256("basekey{}".format(i).encode()).hexdigest()
		if result.startswith(cond):
			print(result)
			break
		i += 1

		
>>> test("000000")
00000053fee83bda59ede23460d072517bbb51f07b6078ed802ad34076114bbc
>>> test("c0ffee")
c0ffee96854cb14679c1fb37a22e2e28c6671161976a714710de737f1416ce9d
```

- [Microservices with Docker, Flask, and React](http://testdriven.io/): Docker, Flask, Postgres, AWS EC2를 이용해 RESTful API를 만드는 튜토리얼, 무척 잘 소개되어있고 시간내서 읽어야겠다. 1부는 Flask, Docker, 2부는 React, 3부는 migrate와 jwt, react router, 4부는 test, swagger, 5부는 EC2, 6부는 testing, coverage 등등 지금 보니 toby에 저장시킨 비슷한 아티클도 있네 ㅎㅎ;;
- [A simple Blockchain in Python](https://github.com/dvf/blockchain): 아까 [`js`버전으로 봤던거](https://github.com/lhartikk/naivechain)의 python버전이네. issue에 보니 test 추가 `help wanted`해놨던데 추석 때 해봐야지
- [Simple note taking from the command line](https://dev.to/ricardomol/note-taking-from-the-command-line-156): 뭔가 디지털 속의 아날로그 느낌도 나고 아주 좋다. 댓글에 있는 버전으로 추가했더니 훨씬 낫고 애용할 것 같다!

```bash
# .bashrc 혹은 .zshrc 등에 추가
notes() {
  if [ ! -z "$1" ]; then
    echo "$@" >> "$HOME/notes.md"
  else
    cat - >> "$HOME/notes.md"
  fi
}

$ notes 메모할 내용
$ more ~/notes.md
메모할 내용
```

- [openblockchains/awesome-blockchains Repo](https://github.com/openblockchains/awesome-blockchains): FAQ가 있는거랑 언어별(지금은 js, py, rb) 기본 구현체가 있다는게 눈에 띈다. 개인적으로 왜 간단한 블록체인 구현체마다 PoW방식만 있지 PoS는 없는지 궁금하다. 내가 만들 수 있다면 만들어서 PR날려야지, 아니면 difficulty 조정 알고리즘이 포함된 PoW를 만들어 날리든지..
- [How To Code in Python 3 튜토리얼](https://www.digitalocean.com/community/tutorial_series/how-to-code-in-python-3): 영어긴 하지만 이것만 봐도 파이썬 3을 꽤 잘 쓸 수 있을 듯. [`code`모듈을 사용한 디버깅](https://www.digitalocean.com/community/tutorials/how-to-debug-python-with-an-interactive-console)과 [파이썬3의 로깅](https://www.digitalocean.com/community/tutorials/how-to-use-logging-in-python-3) 부분이 잘 되어있다.

## TODO
1. - [[번역] 자바스크립트에 대한 애정을 언어의 90%를 쓰레기통에 버리면서 다시 발견했던 과정](https://rhostem.github.io/posts/2017-09-how-i-rediscovered-my-love-for-java-script-after-throwing-90-of-it-in-the-trash/)에 있는 레퍼런스 읽기
2. [A simple Blockchain in Python](https://github.com/dvf/blockchain) 이슈 해결
3. PoS python 구현
4. `notes` 커맨드 커스터마이징
