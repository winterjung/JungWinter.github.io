---
layout: post
title: 기술 문서 정리 - 06
tags: [article, python, 파이썬]
section: article
---

읽은 기술 문서를 정리해보는 글이다. 나중에 필요할 때 찾기 쉽도록 정리해두는 아카이빙도 겸한다.

### [Asynchronous Tasks with Falcon and Celery]

[Falcon]이라는 웹 프레임워크와 [Celery]를 사용해 비동기 작업을 다루고 docker compose로 묶고 테스팅을 어떻게 하는지 소개한다. 글에 쓰인 코드들은 [falcon-celery]레포에서 확인 가능.

#### Celery

Celery를 사용하는 `tasks.py`에서 브로커와 백엔드 환경 변수를 어떻게 설정하고, 로그를 설정하고 이를 `docker-compose.yml`로 구성하는 부분이 참고할만하다. [Flower]를 이용해 셀러리를 모니터링하고 이것도 docker-compose에 통합시킨다.

```yml
version: '3.5'

services:

  web:
    build: ./project
    image: web
    container_name: web
    ports:
      - '8000:8000'
    volumes:
      - './project:/usr/src/app'
    command: gunicorn -b 0.0.0.0:8000 app:app
    environment:
      - CELERY_BROKER=redis://redis:6379/0
      - CELERY_BACKEND=redis://redis:6379/0
    depends_on:
      - redis

  celery:
    image: web
    volumes:
      - './project:/usr/src/app'
      - './project/logs:/usr/src/app/logs'
    command: celery -A app.tasks worker --loglevel=info  --logfile=logs/celery.log
    environment:
      - CELERY_BROKER=redis://redis:6379/0
      - CELERY_BACKEND=redis://redis:6379/0
    depends_on:
      - web
      - redis
  redis:
    image: redis:3.2.11
```

#### Falcon 코드 예제

개인적으로 마음에 들지 않는 형식이라 Falcon을 쓸 일은 없을 것 같다.

```python
import falcon

class QuoteResource:
    def on_get(self, req, resp):
        """Handles GET requests"""
        quote = {
            'quote': (
                "I've always been more interested in "
                "the future than in the past."
            ),
            'author': 'Grace Hopper'
        }

        resp.media = quote

api = falcon.API()
api.add_route('/quote', QuoteResource())
```

#### 테스트

`unittest`를 사용해서 별로 마음에 들지 않는다. 원래라면 실제 동작을 수행하느라 20초 걸릴 테스트를 `unittest.mock.patch`으로 mocking해 우회한다.

### [Release of the new Melon IPFS Front-end]

IPFS를 사용해 [Melon]이라는 프로젝트의 [펀딩 사이트]를 만들었다는 글인데 사이트의 디자인이 마음에 든다. 폰트랑 타이포라이터같은 디자인, 군더더기 없는 레이아웃에 로고까지

### [RxJS motion graphic]

매주 월요일마다 RxJS에 관한 모션 그래픽을 만들어 아티클로 연재하는데 매우 직관적이고 이쁘다. 사이트 디자인도 마음에 든다.

### [Sqlalchemy - Basic Relationship Patterns]

백엔드를 주로 하고있지만 언제나 DB와 Model 설계는 제대로 알지 못한채로 중구난방으로 하고있다는 느낌을 받는다. 일단 모델간의 기초 관계를 제대로 알고자 One to Many, Many to One, One to One, Many to Many에 관한 레퍼런스를 읽어봤다. 짬날때 실제 코드로 뭐가 다른지 하나하나 뜯어 볼 필요가 있다.

### [Flask by Example – Setting up Postgres, SQLAlchemy, and Alembic]

Postgre, Config 설정과 Model, Flask-Migrate와 Alembic을 이용한 마이그레이션 파트가 볼만하다.

### [flask-sqlalchemy Quickstart]

SQLAlchemy를 flask에서 쓰도록 래핑한 flask-sqlalchemy의 퀵스타트. 코드를 살펴보니 쿼리와 세션 관리 측면이 용이한 듯 하다. 아직 그냥 sqlalchemy와 flask-sqlalchemy의 차이를 정확히는 모르겠다.

### [프로그래머스 블록체인 라이브러리]

블록체인에 관한 여러 자료를 잘 모아놨다. 비트코인, 이더리움, 하이퍼렛져, EOS, Qtum 등등. 한글 자료도 많다.

### [Factory boy], [Model mommy]

Factory boy는 범용이고 비슷한 object factory 라이브러리로 [model_mommy]가 있는데 이건 only for django다. 쓰기에 더 편해보이는데 flask에서 쓰려면 결국 factory boy밖에 선택지가 없는 듯.

#### 참고

- [테스트용 객체 생성을 편하게 - model_mommy]
- [Python testing: Factory Boy or Model Mommy]
- [factory boy - Common recipes]

### [Pony ORM]

ORM으로 DjangoORM이랑 SQLalchemy만 알고있었는데 우연히 Pony ORM을 알게되고 문서를 읽었는데 코드가 너무 이쁘다. real app에서 적용하기 전에 많은 테스트를 해봐야 확신이 들겠지만 지금 예제 코드로만 보건데 sqlalchemy보다 훨씬 우아한 문법을 가지고 있다. 자세한건 [firststeps]을 보자. 학식알리미에 적용시켜봐야겠다.

#### 참고

- [A Todo app with flask and Pony], flask-restful과 pony를 사용한 예제

[Asynchronous Tasks with Falcon and Celery]: https://testdriven.io/asynchronous-tasks-with-falcon-and-celery
[falcon-celery]: https://github.com/testdrivenio/falcon-celery
[Falcon]: https://falconframework.org/
[Celery]: http://www.celeryproject.org/
[Flower]: https://flower.readthedocs.io/en/latest/
[Release of the new Melon IPFS Front-end]: https://medium.com/melonport-blog/release-of-the-new-melon-ipfs-front-end-7959a4958039
[펀딩 사이트]: https://ipfs.io/ipns/melon.fund
[Melon]: https://melonport.com/
[RxJS motion graphic]: http://reactive.how/
[Sqlalchemy - Basic Relationship Patterns]: http://docs.sqlalchemy.org/en/latest/orm/basic_relationships.html
[Flask by Example – Setting up Postgres, SQLAlchemy, and Alembic]: https://realpython.com/flask-by-example-part-2-postgres-sqlalchemy-and-alembic/
[flask-sqlalchemy Quickstart]: http://flask-sqlalchemy.pocoo.org/2.1/quickstart/
[프로그래머스 블록체인 라이브러리]: https://programmers.co.kr/pages/blockchain
[Factory boy]: https://github.com/FactoryBoy/factory_boy
[Model mommy]: https://github.com/vandersonmota/model_mommy
[model_mommy]: https://github.com/vandersonmota/model_mommy
[테스트용 객체 생성을 편하게 - model_mommy]: https://raccoonyy.github.io/generating-dynamic-test-fixture-with-model_mommy/
[Python testing: Factory Boy or Model Mommy]: https://vincent.is/using-factory-boy-or-model-mommy/
[Pony ORM]: https://docs.ponyorm.com/
[firststeps]: https://docs.ponyorm.com/firststeps.html
[A Todo app with flask and Pony]: http://nafiulis.me/a-todo-app-with-flask-and-pony.html
[factory boy - Common recipes]: https://factoryboy.readthedocs.io/en/latest/recipes.html
