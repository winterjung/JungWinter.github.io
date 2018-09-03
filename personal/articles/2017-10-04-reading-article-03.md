---
layout: post
title: 마음찍어둔 문서 읽기 - 03
tags: [article, python, 파이썬, 리뷰]
---

## 트위터 마음함에 있는 아티클 리뷰
오늘의 성과: 899개 → 865개

> 하루 사이에 5개가 더 늘어났었다... 그리고 제목을 바꿀까 고민이네 마음찍어둔 문서 읽기 라니 뭔가 멋없다 그래도 꼬박꼬박 30개씩은 줄고있네 ㅠㅠ

## 리뷰
### [Getting Started on Geospatial Analysis with Python, GeoJSON and GeoPandas](https://www.twilio.com/blog/2017/08/geospatial-analysis-python-geojson-geopandas.html)
예전에 담아뒀던 아티클이 `GeoJSON`에 관련된 내용이었는데 같이 봐야할 것 같다. 도시공학과를 다니면서 ArcGIS나 QGIS를 사용해 이런 지리적 정보를 다루고 편집하곤 했는데 여기서 소개되는 `shapely`, `geopandas`, `geojsonio`를 이용하면 재밌는 프로젝트를 할 수 있을 것 같다. 과연 한국쪽도 지원이 될지는 모르겠지만... 베셀이랑 프로젝션이 있으려나?

### [Functional Programming in JavaScript? Yes, Please.](https://dev.to/twhite/functional-programming-in-javascript-yes-please)
js의 `filter`를 사용해 함수형 프로그래밍의 맛을 보여준다. 복잡한 조건이면 나도 필터나 맵같은 걸 쓰겠지만 웬만하면 리스트 컴프리헨션으로 하는 버릇이 있다. 한 두번씩 써버릇하면 손에 익지 않을까 싶다. 재사용성을 따지자면 필터가 나은 듯 하고.

```js
// 기존의 for loop 방식
const cats = [];

for (let i = 0; i < animals.length; i++) {
  if (animals[i].species === ‘cat’) {
     cats.push(animals[i]);
  }
}

// filter 사용
const isCat = function(animal){
  return animal.species === ‘cats’;
});

const cats = animals.filter(isCat);
```

```python
[i for i in animals if i["species"] == "cat"]
```

### [Using Python and Docker for data science](https://medium.com/@elvismiranda213/using-python-and-docker-for-data-science-c104e7b50497)
Data science글이라기 보단 docker에 대한 글이다. 도커로 파이썬 컨테이너를 띄워 `numpy`, `scipy`, `scikit-learn`을 설치하고 `Decision tree`로 과일을 예측한다. 이 글은 아래의 이미지가 제일 유용하다.

<div markdown="0" align="center">
    <img src="https://cdn-images-1.medium.com/max/800/1*olksnxFjYHNp0MmoaJKDNA.png">
</div>

### [Intro to Threads and Processes in Python](https://medium.com/@bfortuner/python-multithreading-vs-multiprocessing-73072ce5600b)
프로세스와 스레드를 잘 설명하고 그래프로 멋지게 시각화한 글이다! 프로세스안에 여러 개의 스레드가 동작하고 있고, 파이썬의 GIL때문에 `IO바운드 작업`에서는 `멀티스레드`가, `CPU바운드 작업`에서는 `멀티프로세스`가 좋다고 한다. 멀티스레드와 멀티프로세스를 각각 `API calls`, `IO Heavy Task`, `CPU Intensive`상황에서 비교 그래프로 보여준다! 이건 번역해보고 싶은 글이다.

### [Building a RESTful API with Go](https://dev.to/codehakase/building-a-restful-api-with-go)
`Go`언어로 간단한 RESTful API서버를 만드는 글. golang도 다음에 크롤러 만들 일이 있으면 써보려고 하는 언어인데 C언어랑 무척 비슷하단 느낌을 받는다. 파이썬 보다 아름답진 않지만 고루틴을 통한 병렬 프로그래밍이 쉽다는 점과 성능이 나름 매력적이다.

### [Why you should use standard HTTP methods when designing REST APIs](https://dev.to/suhas_chatekar/why-should-you-use-standard-http-methods-while-designing-rest-apis)
REST API에 있는 HTTP 메소드들에 대해 설명한다. GET, POST, PUT, PATCH, DELETE에 대해 설명해주고 특히 PUT과 PATCH모두 리소스를 업데이트 하는 메소드인데 무엇이 다른가에 대해 `Idempotence`(멱등성) 개념으로 설명해준다! 이 멱등성이란건 메소드를 반복해도 결과가 동일함을 의미하는데 **PUT**은 카운터를 6으로 set하는 `idempotence 메소드`, **PATCH**는 카운터를 1 증가시키는 `non-idempotence 메소드`로 풀이한다. RESTful API를 만든다면 이런 개념을 확실히 알아놔야겠다. 참고로 GET, PUT, DELETE는 **멱등 연산**, POST, PATCH 는 **비멱등 연산**이라 보는데 PUT과 DELETE는 서버의 상태를 변경시키긴 하지만 여러번 반복해도 결과는 똑같으니 멱등이라 본단다.

### [Micro frontends—a microservice approach to front-end web development](https://medium.com/@tomsoderlund/micro-frontends-a-microservice-approach-to-front-end-web-development-f325ebdadc16)
프론트엔드도 마이크로서비스적 접근이 가능한지 처음 알았다. 앵귤러로 짠 부분, 리액트나 뷰로 짠 부분을 마이크로하게 독립시켜 `다른 URL에 다른 SPA`를 실현한다. 서로는 `공유된 이벤트 버스`(e.g. [chrisdavies/eev](https://github.com/chrisdavies/eev))를 사용해 통신한다고 한다. 흥미롭네~

<div markdown="0" align="center">
    <img src="https://cdn-images-1.medium.com/max/800/1*JO_lCFjbSo70mfeOmD9NgA.png" width="400">
</div>

### [Rythm.js: A javascript library that makes your page dance](https://okazari.github.io/Rythm.js/)
음악에 맞춰 컴포넌트들을 움직이게 해주는 라이브러리인데 재밌다ㅋㅋ 나중에 쓸 일이 있을 것 같다.

### [Building a movie app interface with Vue.js](https://hackernoon.com/building-a-movie-app-interface-with-vue-js-cdc8aeb5db0b)
Vue를 사용해 멋진 UI인터페이스를 만들어보는 글. 나는 아직 Vue를 배울 생각은 없지만 practice로써 좋은 글이다. 뭔가 뚝딱뚝딱 하더니 이쁜 효과가 들어가고 미려한 UI가 만들어지고 대단하다.

<div markdown="0" align="center">
    <img src="https://cdn-images-1.medium.com/max/1000/1*8qAioM8zBKS-DV6iU9snZg.png" width="500">
</div>

### [How Does the Blockchain Work?](https://medium.com/@micheledaliessi/how-does-the-blockchain-work-98c8cd01d2ae)
블록체인에 대해 세세하게 짚어주며 설명해주는 글. 블록체인과 비트코인 입문으로 딱 적당한 무게감인데 굳이 영어로 된 입문 글 보다는 좋은 한글 문서도 많다. 개인적으로는 아래의 3개 **한글 문서**를 추천한다.

- [신뢰의 기술, 블록체인](https://brunch.co.kr/@bumgeunsong/16)
- [원리부터 파악하는 비트코인](https://blog.iwanhae.ga/introduction_of_bitcoin/)
- [비트코인과 블록체인 기술](http://d2.naver.com/helloworld/8237898)

이 문서들 말고도 정리해둔게 여럿있는데 나중에 따로 블록체인 입문하기용 글을 쓰면서 첨부할 생각이다.

### [Modularization and dependency management: three steps to better code](https://dev.to/ice_lenor/modularization-and-dependency-management-three-steps-to-better-code)
좋은 코드를 위해 모듈화와 의존성 관리에 신경쓰라고 말하는 글. 직접적으로 서로 연결되어 있는 모듈보다는 래핑을 거치고 추상화하는게 좋단다. 작게는 함수를 분리하는 것 부터 크게는 API분리까지. 항상 기억하자

<div markdown="0" align="center">
    <img src="http://smartpuffin.com/wp-content/uploads/2017/05/New-Mockup-1.png" width="300">
    <img src="http://smartpuffin.com/wp-content/uploads/2017/05/New-Mockup-2.png" width="300">
</div>

### [DelogX - Dynamic markdown-based blogging platform, Python + Flask](https://deluxghost.me/)
파이썬과 플라스크로 만든 마크다운 기반 블로그 프레임워크라길래 흥미롭지만 사실 이제와선 지킬이든 뭐든 쓰기편하고 테마만 이쁘면 뭐든 상관없다 주의가 되어버렸다. 지킬도 테마만 바꿔주고 `style.css`만 적당히 개조해 커스터마이징하면 써볼만한 수준이 된다.

### [How do you know your code is bad?](https://dev.to/bob/how-do-you-know-your-code-is-bad)
강하게 결합되고, 깨지기 쉽고, 재사용할 수 없는 코드를 나쁜 코드라고 본다. 그러면서 인용한 말이 꽤 인상적이다.

> You want a banana but what you get is a gorilla holding a banana and the entire jungle with it. — Joe Armstrong

`바나나`를 원했지만 `바나나를 들고있는 고릴라가 있는 정글`가 튀어나온다는 [고릴라-바나나 문제](https://www.johndcook.com/blog/2011/07/19/you-wanted-banana/)로 알려져 있나보다. 처음 들어보네. 하여튼 요약하자면 **코드와 모듈의 커플링을 피해라**다.

### [[번역] 초보자를 위한 함수형 자바스크립트 CURRYING 가이드](https://www.sujinlee.me/blog/currying-in-functional-javascript/)
오랜만에 보는 한글 문서지만 번역글이었다 ㅎㅎ;; 어쨌거나 js에서 커링 함수에 대한 간단한 예제를 보여주며 세련되게 커링함수를 만드는 법을 알려준다. 함수형 프로그래밍을 배운적은 없지만 그래도 낯설지는 않아 내심 다행이다. 참고로 파이썬에서는 `functools`의 `partial`을 사용하면 커링함수를 쉽게 만들 수 있다. 

<div markdown="0" align="center">
    <img src="https://pbs.twimg.com/media/DJX-brxUQAEPyeF.jpg" width="500">
</div>

### [Functional Programming HOWTO - Python 3.6.2 documentaion](https://docs.python.org/3.6/howto/functional.html#generators)
파이썬에서 함수형 프로그래밍을 어떻게 하는지 알려주는 글일 줄 알고 담아뒀는데 그냥 레퍼런스였다. `iterator`와 `generator`를 활용하고 컴프리헨션과 `operator`, `functools`를 잘 활용하자 정도록 요약가능하다.

### [Python - Simple and Concise Logging Setup](http://dev.prodigi.us/post/python-simple-and-concise-logging-setup/)
파일과 콘솔 동시에 로깅하는 간단한 예제. `FileHandler`와 `StreamHandler`를 만들고 `setFormatter`로 포맷을 지정한 후 `getLogger(__name__)`으로 받아왔던 `logger`에 `addHandler`해주면 된다.

### [The Anatomy of ERC20](https://dev.to/aunyks/the-anatomy-of-erc20-bfg)
이더리움의 [ERC20제안](https://github.com/ethereum/EIPs/issues/20)에 대해 분석하는 글이다. ERC20은 강제규약은 아니고 토큰을 만들 때 이런게 있으면 좋다 라는 정도로 제안하는 규약인데 현재 발행되는 토큰의 대부분이 이 ERC20을 기반으로 만들어지고 있다. `balanceOf`, `totalSupply`, `transfer`, `transferFrom`, `approve`, `allowance` 함수들을 포함하고 있는데 이 글에서는 각각의 함수들을 풀어 설명해주고 있다. 꽤 좋은 글 같은데 별도의 번역 포스팅으로 올려야겠다.

```s
contract ERC20 {
   function totalSupply() constant returns (uint theTotalSupply);
   function balanceOf(address _owner) constant returns (uint balance);
   function transfer(address _to, uint _value) returns (bool success);
   function transferFrom(address _from, address _to, uint _value) returns (bool success);
   function approve(address _spender, uint _value) returns (bool success);
   function allowance(address _owner, address _spender) constant returns (uint remaining);
   event Transfer(address indexed _from, address indexed _to, uint _value);
   event Approval(address indexed _owner, address indexed _spender, uint _value);
}
```

## TODO
- [Intro to Threads and Processes in Python](https://medium.com/@bfortuner/python-multithreading-vs-multiprocessing-73072ce5600b) 번역
- 블록체인 입문하기용 안내서 제작
- [The Anatomy of ERC20](https://dev.to/aunyks/the-anatomy-of-erc20-bfg) 번역
