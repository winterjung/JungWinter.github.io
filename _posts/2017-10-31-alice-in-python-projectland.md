---
layout: post
title: "[번역] 파이썬 나라의 앨리스"
tags: [파이썬, python, 번역]
---

이 글은 파이썬 프로젝트 패키징에 관한 [Vicki Boykis](https://veekaybee.github.io/)의 [Alice in Python projectland](https://veekaybee.github.io/2017/09/26/python-packaging/)를 번역한 글입니다. 오역 혹은 그 외 잘못된 부분이 있다면 역자의 [트위터](https://twitter.com/res_tin), [메일](mailto:wintermy201@gmail.com), [PR](https://github.com/JungWinter/JungWinter.github.io)로 알려주시면 수정하겠습니다. 모든 저작권과 권리는 원작자인 [Vicki Boykis](https://veekaybee.github.io/)에게 있습니다.

![](https://raw.githubusercontent.com/veekaybee/veekaybee.github.io/master/images/alice_cards.jpg)


## 서문
파이썬 프로젝트 구조와 패키징 표준화는 아직 문제가 많다. 이는 최근에 내가 기계학습된 자연어 처리 앱을 패키징할 때 더욱 크게 느껴졌다.


JVM에서는 [경로가 올바르게 구조화 되어 있다면](https://maven.apache.org/guides/introduction/introduction-to-the-standard-directory-layout.html), 빌드 도구는 이를 이해하고 실행 가능한 JAR로 패키지를 생성한다.


그러나 파이썬에서 패키징을 하려 하니 그다지 쉽지 않았다. `virtualenv`, `pipenv`, `setuptools`를 사용해야 하나? `setup.cfg`가 있어야하나? `wheel`이랑 `egg`는 또 뭐고? 각 폴더에 `__init__.py`가 있어야 하나? 그 파일은 대체 뭐야? 동일한 `PYTHONPATH`에서 모듈을 어떻게 참조하지? 같은 여러 의문들이 들었다.


내가 코드를 작성할 때 정말 고맙게 생각하는 파이썬의 유연성이 프로젝트를 운영하는 데에는 장애물이 되었다.


그래서 간단한 예제를 통해 바닥부터 시작해 완전한 파이썬 프로젝트를 만들었으며 내가 배운 모든 것을 적었고, 같은 문제를 겪는 사람들에게 도움이 되리라 생각한다.


파이썬에서 패키징이 어떻게 그리고 왜 그렇게 작동하는지 찾기 위해 마법과 모험 그리고 정말 성가신 상대 경로 참조로 이루어진 여정을 떠나보자.


이 글은 아래의 순서로 이루어져있다.


![](https://raw.githubusercontent.com/veekaybee/veekaybee.github.io/master/images/package_flow.png)


이 글을 잘 이해하기 위해선 파이썬에 익숙해야하고([리스트 컴프리헨션](http://effbot.org/zone/python-list.htm)이 무엇인지 알고 어떻게 동작하는지 안다면 더 좋다), 객체 지향 프로그래밍 기초에 대한 이해가 있어야한다.


모든 코드는 [여기](https://github.com/veekaybee/textedit/tree/master/textedit)에 있다.


이 글이 살아있는 글이 되길 원하므로 뭔가 심각하게 잘못되거나 내가 놓친 부분을 발견한다면 부담갖지 말고 [PR](https://github.com/veekaybee/textedit/pulls)을 보내주길 바란다.


## Python hides the hurt 파이썬의 숨겨진 상처
파이썬 글을 조금의 자바 코드로 시작할텐데, 미리 양해를 바란다.


굳이 그러는 이유는 다른 언어와 비교하는 것이 때때로 무척 도움이 되기도 하고, 내가 현재 컴퓨터 과학 분야에서 일하면서 대부분의 강의를 자바로 진행했기 때문이다. 스파크와 스칼라를 예외로 치면 이전에는 보통 동적 타입 언어인 파이썬과 R을 써왔기 때문에 자바 문법이 완벽하지 않을 수 있다.


텍스트 파일을 읽고, 변경한 후 새로운 파일로 내보내는 예제를 보면 자바는 이것저것 써줘야할게 많다.

```java
import java.io.*;

public class Replace {

    public static void main(String[] args) throws IOException {
        File file = new File("alice.txt");

        try {
            FileReader freader = new FileReader(file);
            BufferedReader reader = new BufferedReader(freader);
            FileWriter writer = new FileWriter("new_alice.txt")
            String line;

            while ((line = reader.readLine()) != null) {
                String newLine = line.replaceAll("\\s+$", "").replaceAll("Alice", "Vicki");
                writer.write(newLine);
            }
        }
    }
}
```

하지만 파이썬에선 훨씬 간단하다.

```python
# 파일에 있는 모든 "Alice"를 "Dora the Explorer"로 바꾼다

with open('alice.txt', 'r') as input:
    with open('new_alice.txt', 'w') as output:
        for line in input:
            line = line.rstrip()
            newline = line.replace("Alice", "Dora the Explorer")
            output.write(newline)
```

몇 줄짜리 프로그램이라면 자바를 사용하기 싫을 수 있다. 그렇지만 자바의 암묵적 타입 변환 방지와 이것저것 적어줘야 하는 것들, 특히 빌드 도구가 주는 이점은 큰 프로그램을 패키징하기 쉽게 만든다.


자바에서는 3개의 클래스가 있다면 클래스들을 `project/src/main/java/program` 이라는 프로젝트 구조에 넣고, `package` 를 호출 하면 서로를 자동으로 참조한다. 각 프로그램은 단일 클래스로 자동으로 분리 되고, Maven에서 분리된 프로그램을 컴파일 하면 끝난다.


그러나 파이썬은 타입, 오브젝트, 경로를 추상화하고 빌드 시스템이 없기 때문에 패키징이 내부적으로 조금 더 복잡해진다.


파이썬이 무엇을 추상화하는지, 왜 다른 아키텍처를 가지게 됐는지 이해하기 위해 `처음`부터 시작해보자.

## 간단한 워드 프로세서 만들기
![](https://raw.githubusercontent.com/veekaybee/veekaybee.github.io/master/images/rabbit.jpg)

내가 _이상한 나라의 앨리스_를 쓰고 있는 루이스 캐롤이라 하자. 그러나 잉크와 종이 대신 21세기 기술을 사용하려 한다.


작가가 책을 쓰면서 자주 하는 일 중에 프로그램을 이용해 쉽게 자동화할 수 있는 것이 무엇이 있을까? 보통 마침표 뒤에 공백 넣기, 맞춤법 검사, 글자 바꾸기, 글자 수 세기같은 작업이 빈번하기 때문에 작가는 워드 프로세서를 애용한다.


우리는 이제 [정말 정말 단순한(정말로) 워드 프로세서](https://github.com/veekaybee/textedit)를 만들면서 파이썬 패키징이 어떻게 작동하는지 내부적으로 파헤치려 한다. 부디 즐겁기 바란다.

## Creating a single object 단일 객체 만들기
![](https://raw.githubusercontent.com/veekaybee/veekaybee.github.io/master/images/alice_door.gif)

파이썬 코드를 작성하기 위해, 작은 코드를 안전하게 테스트 할 수 있는 파이썬 REPL을 사용한다.


그리고 아래의 코드를 작성해보자.

`x = "Alice"`

이것은 변수를 선언하는 하나의 완전한 파이썬 코드다. 얼핏 보면 단순히 문자열을 만드는 것처럼 보이지만 실제로는 객체를 만드는 것이다. 이는 문자열뿐만 아니라 모든 것, 심지어 자료형마저도 객체기 때문이다.


객체는 파이썬의 기본 요소다. 그리고 모듈이라는 파일에 이 객체를 여럿 넣어 묶을 수 있다. 또 여러 모듈을 패키지로 묶을 수 있다. 패키지와 모듈 또한 객체라는 사실을 알게되면 조금 더 복잡해지겠지만 여기서는 간단하게 설명하겠다.

![](https://raw.githubusercontent.com/veekaybee/veekaybee.github.io/master/images/model.png)

`x`라는 객체가 생기면, 우리는 내부적으로 무슨일이 일어나는지 찾아볼 수 있다.


[파이썬 객체](https://docs.python.org/3/reference/datamodel.html)는 객체가 저장된 메모리 주소를 가리키는 포인터, 자료형, 값을 가진다. 이들은 객체에 대한 메타 정보이며 파이썬에서 다양한 구성요소들이 서로 어떻게 상호작용하는지 조사하는 데 도움이 된다.

![](https://raw.githubusercontent.com/veekaybee/veekaybee.github.io/master/images/object_props.png)

자바에서는 모든 것을 일일이 작성해줘야만 하므로 코드를 작성하며 변수의 자료형이 무엇인지, 클래스는 무엇인지, 어느 프로그램에 있는지 명시적으로 볼 수 있다.

```java
public class Alice {

    public static void main(String[] args) {
        String x = "Alice";
    }

}
```

파이썬에서 그 정보를 찾기 위해서는 조금 파헤쳐야한다. 먼저 우리는 객체의 메모리 주소를 찾을 수 있다.

```python
>>> id(x)
4409066472
```

그런 다음 `type()`을 호출함으로써 객체의 자료형을 볼 수 있다.


그러면 `x`가 `str`이라는 문자열 클래스의 인스턴스임을 볼 수 있고, `str` 또한 `type` 클래스의 인스턴스임을 볼 수 있다.

```python
>>> x = "Alice"
>>> type(x)
<class 'str'>
>>> type(str)
<class 'type'>
```

모든 것이 클래스와 객체다. 또 `__doc__`로 `str` 객체에 대해 더 많이 알 수 있다.

```python
>>> x.__doc__
"str(object='') -> str\nstr(bytes_or_buffer[, encoding[, errors]]) -> str\n\nCreate a new string object from the given object. If encoding or\nerrors is specified, then the object must expose a data buffer\nthat will be decoded using the given encoding and error handler.\nOtherwise, returns the result of object.__str__() (if defined)\nor repr(object).\nencoding defaults to sys.getdefaultencoding().\nerrors defaults to 'strict'."
```

(문자열이 구체적으로 어떻게 생성되는지 , `__doc__` 문자열이 어디서 오는지 궁금하다면 [파이썬 소스 코드](https://github.com/python/cpython/blob/2.7/Objects/stringobject.c)를 파헤쳐보는 게 재밌을 것이다.)


`dir()`을 호출하면, 그 클래스에서 작동하는 모든 객체 속성과 메소드를 볼 수 있다. 이들은 더 추상화된 파이썬 클래스에 의해 만들어지기도 하고, 특정 객체에만 있는 메소드도 있다. 예를 들어 `__add__`, `__dir__`, `__setattr__`는 대부분의 파이썬 객체가 가지고 있는 속성이고, `join`, `strip`, `replace` 는 오직 문자열만 가지고 있다.


```python
>>> dir(x)
['__add__', '__class__', '__contains__', '__delattr__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__getitem__', '__getnewargs__', '__gt__', '__hash__', '', '__iter__', '__le__', '__len__', '__lt__', '__mod__', '__mul__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__rmod__', '__rmul__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', 'capitalize', 'casefold', 'center', 'count', 'encode', 'endswith', 'expandtabs', 'find', 'format', 'format_map', 'index', 'isalnum', 'isalpha', 'isdecimal', 'isdigit', 'isidentifier', 'islower', 'isnumeric', 'isprintable', 'isspace', 'istitle', 'isupper', 'join', 'ljust', 'lower', 'lstrip', 'maketrans', 'partition', 'replace', 'rfind', 'rindex', 'rjust', 'rpartition', 'rsplit', 'rstrip', 'split', 'splitlines', 'startswith', 'strip', 'swapcase', 'title', 'translate', 'upper', 'zfill']
>>> x.__init__
<method-wrapper '__init__' of str object at 0x105ec4d88>
```

이는 `int` 를 만들어서 확인해볼 수 있다.

```python
>>> y = 9
>>> dir(y)
['__abs__', '__add__', '__and__', '__bool__', '__ceil__', '__class__', '__delattr__', '__dir__', '__divmod__', '__doc__', '__eq__', '__float__', '__floor__', '__floordiv__', '__format__', '__ge__', '__getattribute__', '__getnewargs__', '__gt__', '__hash__', '__index__', '__init__', '__int__', '__invert__', '__le__', '__lshift__', '__lt__', '__mod__', '__mul__', '__ne__', '__neg__', '__new__', '__or__', '__pos__', '__pow__', '__radd__', '__rand__', '__rdivmod__', '__reduce__', '__reduce_ex__', '__repr__', '__rfloordiv__', '__rlshift__', '__rmod__', '__rmul__', '__ror__', '__round__', '__rpow__', '__rrshift__', '__rshift__', '__rsub__', '__rtruediv__', '__rxor__', '__setattr__', '__sizeof__', '__str__', '__sub__', '__subclasshook__', '__truediv__', '__trunc__', '__xor__', 'bit_length', 'conjugate', 'denominator', 'from_bytes', 'imag', 'numerator', 'real', 'to_bytes']
```

마지막으로 [객체의 값](https://stackoverflow.com/questions/12693606/reason-for-globals-in-python)이다. 지금 우리는 파이썬 REPL에서 작업하고 있기에 `x` 는 글로벌 변수고, 이는 즉 전체 파이썬 네임 스페이스에서 사용가능 하다는 뜻이다. 아래와 같이 확인해볼 수 있다.

```python
>>> globals()
{'__loader__': <class '_frozen_importlib.BuiltinImporter'>, 'x': 'Alice', '__spec__': None, 'y': 9, '__name__': '__main__', '__doc__': None, '__builtins__': <module 'builtins' (built-in)>, '__package__': None}
```

`x`의 값을 얻기 위해 `globals`를 사용할 수도 있다.

```python
>>> globals()['x']
'Alice'
```

이제 하나의 객체가 어떻게 구성되어있는지 알았으므로, 얄팍한 REPL에서 벗어나 이 객체들이 서로 얽히게끔 가지를 뻗어보자.

## Combining objects into a program
![](https://raw.githubusercontent.com/veekaybee/veekaybee.github.io/master/images/tea_mouse.jpg)

### Refactoring a single program
Ok, so we’ve run our program. But, if we try to run it on any other file, not just alice.txt, we won’t be able to. Let’s make it a bit more robust. First, we’ll abstract out some of the hard coding referencing alice.txt.

## Combining programs into scripts
![](https://cdn-images-1.medium.com/max/1600/1*c8z_BEgJvo7ra4IKH28RfA.jpeg)

![](https://raw.githubusercontent.com/veekaybee/veekaybee.github.io/master/images/object_model.png)

## Combining scripts into a module
![](https://raw.githubusercontent.com/veekaybee/veekaybee.github.io/master/images/tea_party.png)

![](https://raw.githubusercontent.com/veekaybee/veekaybee.github.io/master/images/wordcountmenu.png)

## Project structure
But, there are signals we need to give to Python to read this correctly. CPython also needs to know how these things relate to each other in order to compile a package for us. And the code needs to be built in a way such that other people can seamlessly download it and use it without a lot of explanation from you.

### Modular Code
First, Python needs to have all of the files of the project in its same directory and subdirectory:. The top-level textedit is the distribution or package, and the lower-level one is the actual module.

### Unit Tests
Then, you’ll want to add tests. Unit tests help you make sure that your code runs as expected, even as you change it. You usually want to write a test for each function or package.

### Requirements.txt and package dependencies
We put a requirements.txt module at the top level of our module. This file that will tell people who install the package which Python packages are used in our programs and automatically install them when you run pip install . on the package. (We may have to run `pip install -r requirements.txt` separately, but that’s usually an edge case.)

### Documentation
Let’s add some documentation, as well. Good documentation is really important, paricularly to someone just coming into your project. And, even more so, for yourself tomorrow morning. :) The easiest way to add documentation is to add a  README to the top level of your module, the same place as requirements.txt.

### Scripts
If you have any shell scripts or additional helper methods associated with your project, you can create a bin directory for them. We don’t, so that folder will be empty. But scripts are usually present anywhere you have to deploy stuff, add it to cron, or generally put it in production in any way.

### `__init__.py`
And, finally and most importantly, the __init__.py, which we’ll want to add to every directory where you have runable Python modules.

### `__main__.py` driver
There is this concept in Java of a driver program that you can run and have it call all the other programs in the package.

### `setup.py`
Now that we have the scaffolding in place, we can add things that will help us set up the module after we import it from pip or download it.

## Sharing and using our package
![](https://raw.githubusercontent.com/veekaybee/veekaybee.github.io/master/images/alice_08c-alice_flamingo.png)
✨ We’ve installed our package locally! ✨

## Next steps
![](https://raw.githubusercontent.com/veekaybee/veekaybee.github.io/master/images/alice_dodo.png)
🎉 Congratulations! We’ve built a Python package! 🎉

### Testing environments
I mentioned before that there are several ways to create specific environments to build your applications so you’re isolated from whatever else is going on in your Python ecosystem.

### More advanced testing
Unittest is a great starting point, but there’s also pytest, nose, mock testing (when you have complex object dependencies), and much, much more. All of these work slightly differently.

### Continuous Integration
Once you build a package, you’ll probably want to make changes to it. And push those changes to some remote version-controlled repository so that others can use them. And you’ll want to automate this process so that you’re not manually doing pip install .. This process is known as continuous integration.

### Git Hooks and Version Control
We haven’t touched the subject of version control for your package, but you’ll want to add it to specific repositories. We can also add pre-commit and post-commit hooks to our code, which mean that the code gets checked, or some specific action is triggered, when you try to check in your code.

### Setup.cfg
In our module, there are no passwords or usernames. But if you’re working with databases, webforms, or any number of software options that require you to log in, you’ll need a separate setup.cfg file that stores your passwords and keeps them safe on your local or testing machine while not sharing them with others.

### Sphinx/reST
We’ve already written README.md. But what if you have multiple files that rely on each other? Or you want to use your docstrings to build documentation? Sphinx and reST are some ways popular Python packages are documented.

### Wheels
Once you’re done refining all of that, and you’re ready to go to production, you should build a wheel. Python wheels are similar to JAR packages in Java, and are a much faster and lightweight process to use in production environments. You can use setup.py to build wheels (which is why it’s so important to get it right initially), python setup.py bdist_wheel --universal.

### PyPi
This is the big one. If your module is stable enough, you can release it to PyPi, which means anyone in the world can download it through pip. There are some extra hoops you have to jump through here, namely in how you configure your setup.py file. For an easier way to do this, Flit is a potential option.

## Conclusion
Python project structure and packaging can be intimidating, but, if you take it step by step, it doesn’t have to be.
