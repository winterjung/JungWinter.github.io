---
layout: post
title: "[번역] 파이썬 나라의 앨리스"
tags: [파이썬, python, 번역]
---

이 글은 파이썬 프로젝트 패키징에 관한 [Vicki Boykis](https://veekaybee.github.io/)의 [Alice in Python projectland](https://veekaybee.github.io/2017/09/26/python-packaging/)를 번역한 글입니다. 오역 혹은 그 외 잘못된 부분이 있다면 역자의 [트위터](https://twitter.com/res_tin), [메일](mailto:wintermy201@gmail.com), [PR](https://github.com/JungWinter/JungWinter.github.io)로 알려주시면 수정하겠습니다. 모든 저작권과 권리는 원작자인 [Vicki Boykis](https://veekaybee.github.io/)에게 있습니다.

![](https://raw.githubusercontent.com/veekaybee/veekaybee.github.io/master/images/alice_cards.jpg)

- [Intro](#intro)
- [Python hides the hurt](#python-hides-the-hurt)
- [Building a simple word processor](#building-a-simple-word-processor)
- [Creating a single object](#creating-a-single-object)
- [Combining objects into a program](#combining-objects-into-a-program)
  - [Refactoring a single program](#refactoring-a-single-program)
- [Combining programs into scripts](#combining-programs-into-scripts)
- [Combining scripts into a module](#combining-scripts-into-a-module)
- [Project structure](#project-structure)
  - [Modular Code](#modular-code)
  - [Unit Tests](#unit-tests)
  - [Requirements.txt and package dependencies](#requirementstxt-and-package-dependencies)
  - [Documentation](#documentation)
  - [Scripts](#scripts)
  - [`__init__.py`](#__init__.py)
  - [`__main__.py` driver](#__main__.py-driver)
  - [`setup.py`](#setuppy)
- [Sharing and using our package](#sharing-and-using-our-package)
- [Next steps](#next-steps)
  - [Testing environments](#testing-environments)
  - [More advanced testing](#more-advanced-testing)
  - [Continuous Integration](#continuous-integration)
  - [Git Hooks and Version Control](#git-hooks-and-version-control)
  - [Setup.cfg](#setupcfg)
  - [Sphinx/reST](#sphinxrest)
  - [Wheels](#wheels)
  - [PyPi](#pypi)
- [Conclusion](#conclusion)
- [Acknowledgements](#acknowledgements)

## <a id="intro"></a>서문
파이썬 프로젝트 구조와 패키징 표준화는 아직 문제가 많다. 이는 최근에 내가 기계학습된 자연어 처리 앱을 패키징할 때 더욱 크게 느껴졌다.


JVM에서는 [경로가 올바르게 구조화 되어 있다면](https://maven.apache.org/guides/introduction/introduction-to-the-standard-directory-layout.html), 빌드 도구는 이를 이해하고 실행 가능한 JAR로 패키지를 생성한다.


그러나 파이썬에서 패키징을 하려 하니 그다지 쉽지 않았다. `virtualenv`, `pipenv`, `setuptools`를 사용해야 하나? `setup.cfg`가 있어야하나? `wheels`랑 `egg`는 또 뭐고? 각 폴더에 `__init__.py`가 있어야 하나? 그 파일은 대체 뭐야? 동일한 `PYTHONPATH`에서 모듈을 어떻게 참조하지? 같은 여러 의문들이 들었다.


내가 코드를 작성할 때 정말 고맙게 생각하는 파이썬의 유연성이 프로젝트를 운영하는 데에는 장애물이 되었다.


그래서 간단한 예제를 통해 바닥부터 시작해 완전한 파이썬 프로젝트를 만들었으며 내가 배운 모든 것을 적었고, 같은 문제를 겪는 사람들에게 도움이 되리라 생각한다.


파이썬에서 패키징이 어떻게 그리고 왜 그렇게 작동하는지 찾기 위해 마법과 모험 그리고 정말 성가신 상대 경로 참조로 이루어진 여정을 떠나보자.


이 글은 아래의 순서로 이루어져있다.


![](https://raw.githubusercontent.com/veekaybee/veekaybee.github.io/master/images/package_flow.png)


이 글을 잘 이해하기 위해선 파이썬에 익숙해야하고([리스트 컴프리헨션](http://effbot.org/zone/python-list.htm)이 무엇인지 알고 어떻게 동작하는지 안다면 더 좋다), 객체 지향 프로그래밍 기초에 대한 이해가 있어야한다.


모든 코드는 [여기](https://github.com/veekaybee/textedit/tree/master/textedit)에 있다.


이 글이 살아있는 글이 되길 원하므로 뭔가 심각하게 잘못되거나 내가 놓친 부분을 발견한다면 부담갖지 말고 [PR](https://github.com/veekaybee/textedit/pulls)을 보내주길 바란다.


## <a id="python-hides-the-hurt"></a>파이썬의 숨겨진 상처
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

## <a id="building-a-simple-word-processor"></a>간단한 워드 프로세서 만들기
![](https://raw.githubusercontent.com/veekaybee/veekaybee.github.io/master/images/rabbit.jpg)

내가 _이상한 나라의 앨리스_를 쓰고 있는 루이스 캐롤이라 하자. 그러나 잉크와 종이 대신 21세기 기술을 사용하려 한다.


작가가 책을 쓰면서 자주 하는 일 중에 프로그램을 이용해 쉽게 자동화할 수 있는 것이 무엇이 있을까? 보통 마침표 뒤에 공백 넣기, 맞춤법 검사, 글자 바꾸기, 글자 수 세기같은 작업이 빈번하기 때문에 작가는 워드 프로세서를 애용한다.


우리는 이제 [정말 정말 단순한(정말로) 워드 프로세서](https://github.com/veekaybee/textedit)를 만들면서 파이썬 패키징이 어떻게 작동하는지 내부적으로 파헤치려 한다. 부디 즐겁기 바란다.

## <a id="creating-a-single-object"></a>단일 객체 만들기
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

## <a id="combining-objects-into-a-program"></a> Combining objects into a program


![tea_muse](https://raw.githubusercontent.com/veekaybee/veekaybee.github.io/master/images/tea_mouse.jpg)


Usually, after we write a couple one-liners, we want to write a complete snippet of code that does something. For example, let's say Lewis Carroll has written a couple pages and wants to do a word count on the file he's written (writers always want to compulsively do word counts).

We'll work with `alice.txt` here: 

```
Alice was beginning to get very tired of sitting by her sister on the bank, and of having nothing to do: once or twice she had peeped into the book her sister was reading, but it had no pictures or conversations in it, ‘and what is the use of a book,’ thought Alice ‘without pictures or conversations?’
So she was considering in her own mind (as well as she could, for the hot day made her feel very sleepy and stupid), whether the pleasure of making a daisy-chain would be worth the trouble of getting up and picking the daisies, when suddenly a White Rabbit with pink eyes ran close by her.
There was nothing so very remarkable in that; nor did Alice think it so very much out of the way to hear the Rabbit say to itself, ‘Oh dear! Oh dear! I shall be late!’ (when she thought it over afterwards, it occurred to her that she ought to have wondered at this, but at the time it all seemed quite natural); but when the Rabbit actually took a watch out of its waistcoat-pocket, and looked at it, and then hurried on, Alice started to her feet, for it flashed across her mind that she had never before seen a rabbit with either a waistcoat-pocket, or a watch to take out of it, and burning with curiosity, she ran across the field after it, and fortunately was just in time to see it pop down a large rabbit-hole under the hedge.
In another moment down went Alice after it, never once considering how in the world she was to get out again.
```

We are now working with a file, `wordcount.py`, and a text file, in the same directory. 

```bash
mbp-vboykis:python_packaging vboykis$ tree
├── alice.txt
└── wordcount.py
```
The loop is still a single logical piece of code. Let's save this as a file and execute it on the command line:  

`vboykis$ python wordcount.py
Word Count: 274`

Let's take a look inside: 

```python
with open('alice.txt', 'r') as file:
	file_contents = file.read()
	print('Word count:', len(file_contents.split()))
		
---
Word count: 274
```

We still have objects that we are performing operations on. `file` is an object, as is `file_contents` and `len(file_contents.split())`, but now we're getting more interesting types: 

```python
type(file_contents)
str

type(file)
_io.TextIOWrapper

type(len(file_contents.split()))
int
```

When we run `python wordcount.py`, a [couple things happen](https://tech.blog.aknin.name/2010/04/02/pythons-innards-introduction/): 

1) Python parses command line arguments (everything after `python` on the CLI)

2) Python checks for any dependencies (aka `import` statements) and pulls them in. 

3) Python looks in our current working directory (`python_packaging`) for anything related to the file, and uses the path setup from when Python was installed on your system. 

Let's check what our `PYTHONPATH` is:

```python
>>> import os
>>> os.getcwd() #where we're currently running code
'/python_packaging'

>>> print('\n'.join(sys.path)) # all the paths Python checks for packages

python3/3.5.1/Frameworks/

Python.framework/Versions/3.5/lib/python35.zip
python3/3.5.1/Frameworks/

Python.framework/Versions/3.5/lib/python3.5
python3/3.5.1/Frameworks/

Python.framework/Versions/3.5/lib/python3.5/plat-darwin
python3/3.5.1/Frameworks/

Python.framework/Versions/3.5/lib/python3.5/lib-dynload
/usr/local/lib/python3.5/site-packages
```

4) Python assembles a Python virtual machine, [CPython](https://en.wikipedia.org/wiki/CPython), to execute and interpret the code. 

5) If there are no external dependencies (aka packages), as is the case in our word count program, a special attribute, ``__name__`` is initialized to ``"__main__"`` in the __main__ [namespace](https://docs.python.org/3/library/__main__.html). 

Program: 

```python
print("__name__:", __name__)

with open('texts/alice.txt', 'r') as file:
	file_contents = file.read()
	print('Word count:', len(file_contents.split()))
```
Output:
 
```bash
mbp-vboykis:python_packaging vboykis$ python wordcount.py 
__name__: __main__
Word count: 274

```

How do we get that file elsewhere? For example, how do we import functions from wordcount if we're in the REPL?  The REPL is, essentially, a special case of a file being executed directly in our current working directory.

However, if we run `wordcount.py` form the REPL, since we're importing the `wordcount` module from elsewhere, the name immediately becomes that module. 

```python
>>> import wordcount
>>> wordcount.__name__
'wordcount'
```

6) All of the code gets translated at run-time to byte code using the CPython interpreter. Python 
generates a copy of the file with the `.pyc` extension in a folder called `__pycache__`.

If you want to see what the byte code of our program looks like, [digging into it](http://akaptur.com/blog/2013/08/14/python-bytecode-fun-with-dis/) can be fun.

7) As the code is executed, Python reads all of our objects and loops. The interpreter allocates memory to the code and the special Python structures for each object are created. This is where the `id` for each object is created, and why we can call these things after we run the code. 
 
We've just created a Python runtime environment, told Python which directories it should be reading from, imported some stuff, allocated memory, and given some output. Basically all of the things we do to run a program. 
 
### <a id="refactoring-a-single-program"></a>Refactoring a single program 

Ok, so we've run our program. But, if we try to run it on any other file, not just `alice.txt`, we won't be able to. Let's make it a bit more robust. First, we'll abstract out some of the hard coding referencing `alice.txt`. 

Second, we'll turn the code from a simple loop, into a function that acts like an API so that we can access a single pointer to our loop from other places in the file, and from other files. 

Finally, we'll add `if __name__ == '__main__'` to the program, put all the functions we want to execute under the if block,  and run the file that way. 

This is a really common pattern for running multiple functions and keeping track of them.  As we noted in the last section, 

> If there are no external dependencies (aka packages), as is the case in our word count program, a special attribute, ``__name__`` is initialized to ``"__main__"`` in the __main__ [namespace](https://docs.python.org/3/library/__main__.html)
> 
If the program is the main program you're running, Python will run everything in order under that statement.  

Note that if you don't have `if __name__ == '__main__'`, the code won't run anything since the function is initialized but not executed. 

This is a really important pattern to understand, because it looks into how Python references the current module you're working with in relation to other modulers. 

Let's also: adding a function that counts sentences, as well.  

Program: 

```python
def word_count(filename):
	with open(filename, 'r') as file:
		file_contents = file.read()
		print('Total words:   ', len(file_contents.split()))

def sentence_count(filename):
	with open(filename, 'r') as file:
		file_contents = file.read()
		print('total sentences:    ', file_contents.count('.')		+file_contents.count('!')+file_contents.count('?'))


if __name__ == '__main__':
	word_count('pool_of_tears.txt')
	sentence_count('pool_of_tears.txt')
```
Output:	

```bash
vboykis$ python wordcount.py
Total words:    2098
total sentences:     117

```

These kinds of refactoring tasks will become important as we start to understand how many files, or modules, reference each other in a package. 

We're also going to slightly modify wordcount to return a value instead of print so we don't double-print: 

```python

def word_count(filename):
	with open(filename, 'r') as file:
		file_contents = file.read()
		return('Total words:   ', len(file_contents.split()))

def sentence_count(filename):
	with open(filename, 'r') as file:
		file_contents = file.read()
		return('total sentences:    ', file_contents.count('.')+file_contents.count('!')+file_contents.count('?'))


if __name__ == '__main__':
	print(word_count('pool_of_tears.txt'))
	print(sentence_count('pool_of_tears.txt'))
```

Let's do one more thing, and now make this function a class so that it's really, really easy to reference from other modules. 

A class is part of [object-oriented programming](https://jeffknupp.com/blog/2014/06/18/improve-your-python-python-classes-and-object-oriented-programming/), and makes code more modular and easy to call. A class, in a way, is like an API. 

And, let's add a character count (we'll need that for some later work we're doing): 

```python
#!/usr/bin/env python

"""Takes a file as sys.arg[1] as input and returns a wordcount, 
character count, and sentence count.

Sysargs:  filename
"""

import re
import sys


class WC:
	"""Conducts character, word, and letter count of object"""

	def __init__(self, filename):

		self.filename = filename


	def open_file(self,filename):
		"""Opens a file object to be used across the class"""
		with open(filename,'r') as f:
			file_contents = f.read()
			return file_contents


	def word_count(self):
		"""Returns a file's word count"""

		wc_file = self.open_file(self.filename)
		self = len(wc_file.split())
		return("Words:",len(wc_file.split()))


	def sentence_count(self):
		"""Returns a file's word sentence count"""

		sc_file = self.open_file(self.filename)
		
		return("Sentences:", sc_file.count('.') + sc_file.count('!') + sc_file.count('?'))

	def letter_count(self):

		"""Returns a file's character count, excluding punctuation"""
		letter_counter = 0

		pattern = r'[\W]' # excluding all punctuation

		cc_file = self.open_file(self.filename)

		total_words = cc_file.split()

		for word in total_words: 
			for letter in word:
				if not re.search(pattern, letter):
					letter_counter += 1
		
		return("Letters:", letter_counter)

	def counts(self):
		print(self.word_count(),'\n',self.sentence_count(),'\n',self.letter_count())

if __name__ == '__main__':
	alice = WC(sys.argv[1])
	WC.counts(alice)

```

Now we're cooking. We've got a program that's easy to read, abstractable, and applicable across multiple files. We have [a complete Python module.](https://github.com/veekaybee/textedit/blob/master/textedit/review/wordcount.py) 

For bonus fun and to do some QA, you can paste the test into Word to do a sanity check. I got 274 words, which matches our program. 


## Combining programs into scripts

![alice](https://cdn-images-1.medium.com/max/1600/1*c8z_BEgJvo7ra4IKH28RfA.jpeg)

Ok, we've cleaned up a single file. We have a pretty simple, clean module to do a word count, a sentence count, or character count on any file that's input.  But what if we want to count the words in a file, and then replace some text in that file, save as a new file, and do a word count again?

I could do this in the same `wordcount.py` script, but for the sake of the case study and modulatrity, let's call these separate pieces of code, since they do separate things. The `wordcount` module is for analytics, the new module would actually change things in the program. 

So now I want one piece of code to do a word count and another to replace words in a file. Here's the word replacement. As always, we start with a single file and a single function.  

```
mbp-vboykis:python_packaging vboykis$ tree
├── alice.txt
├── pool_of_tears.txt
└── replace.py
└── wordcount.py

```

Now we have a couple of text samples and a couple of modules in the same file folder. Starting to get a little complicated, but it's not too bad, yet. 

```python
def replace_words(words):
	with open(words, 'r') as input:
		with open('new_alice.txt', 'w')as output:
			for line in input:
				line = line.rstrip()
				newline = line.replace("Alice", "Dora the Explorer")
				print(newline)
				output.write(newline)

if __name__ == '__main__':
	replace_words('alice.txt')
	
```

Let's say I want to call the wordcount on `alice.txt`, and then again on `new_alice.txt`, since I am adding more words. I could copy and paste the `word_count` method into the `replace_words` code. 

Or, I could just reference it from a different file in the same directory. Remember, Python always checks the directory you're currently working in first for files, and the top-level file is the one that has the `__main__` [module.](https://docs.python.org/3/library/__main__.html#module-__main__ ) 

Our directory [currently looks like this:](https://github.com/veekaybee/textedit/blob/master/textedit/edit/replace.py) 

```

mbp-vboykis:python_packaging vboykis$ tree
├── alice.txt
├── new_alice.txt
├── pool_of_tears.txt
├── replace.py
└── wordcount.py
```
Now, I'd like to call the  `wordcount` module from the `replace` module. Remember that all modules can be imported with their filename. So, if we are in the same directory, we should be able to do a simple `import wordcount`. Once we do that, we have access to all of wordcount's classes and methods in the `replace` module.

We also want to see what the name of the module in relationship to the module calling it is. Notice that wordcount's `__name __`is `wordcount`, while `replace's` is `main`, since that's now the main module we're referencing.

Program: 

```python
from wordcount import WC

def replace_words(words):
	with open(words, 'r') as input:
		with open('new_alice.txt', 'w')as output:
			for line in input:
				line = line.rstrip()
				newline = line.replace("Alice", "Dora the Explorer")
				output.write(newline)

if __name__ == '__main__':
	replace_words('alice.txt')
	alice = WC('alice.txt')
	new_alice = WC('texts/new_alice.txt')
	print("Old Wordcount",alice.word_count())
	print("WC __name__:", WC.__name__)
	print("New Wordcount",new_alice.word_count())
	print("replace __name__:", __name__)
```

Output: 

```
Old Wordcount ('Words:', 274)
WC __name__: WC
New Wordcount ('Words:', 281)
replace __name__: __main__
```

 
Once again: 


![pythonmodel](https://raw.githubusercontent.com/veekaybee/veekaybee.github.io/master/images/object_model.png)

All of the functions from the wordcount module are imported into the `replace` module, to be used. That module is no longer the main module when you are running `replace`, but a helper module, because of the way `__name__` now points. These two modules also have to be in the same directory. 

This distinction makes up the heart of the Python packaging system.  

Let's finish up by replace a callable object with an API, as well. 

## <a id="combining-scripts-into-a-module"></a>Combining scripts into a module

![tea party](https://raw.githubusercontent.com/veekaybee/veekaybee.github.io/master/images/tea_party.png)

Ok, now we have a word "editor", a word counter, and we're cranking away on _Alice in Wonderland_. What else might we want in a text editor?  

Let's brainstorm a couple modules of functionality: 

+ the ability to change from single to double space after a period? 
+ And, the ability to determine how readable our document is based on the [Colman-Liau](https://readable.io/content/the-coleman-liau-index/) index. There are lots of ways to gauge readability, but a lot of them involve counting word syllables, which is too complicated for what we're trying to do here, which is focus on Python package structure. (If you are interested in [wordcount](http://veekaybee.github.io/2017/03/20/hadoop-or-laptop/), check out my other posts. )
 
So, in addition to our two modules: 

+ [wordcount](https://github.com/veekaybee/textedit/blob/master/textedit/review/wordcount.py)
+ [replace](https://github.com/veekaybee/textedit/blob/master/textedit/edit/replace.py)

We'll be adding two more

+ [spacing](https://github.com/veekaybee/textedit/blob/master/textedit/edit/spacing.py)
+ [readability](https://github.com/veekaybee/textedit/blob/master/textedit/review/readability.py)

Based on PEP, the arbiter in Python style, the package name should be [short, lowercase, and meaningful.](https://www.python.org/dev/peps/pep-0008/#package-and-module-names).

Now that we have more than three modules, and also some text files, we'll want to create a package. Let's follow PEP again and call it something easy: `textedit`.  If we were publishing this package to [PyPI](https://pypi.python.org/pypi), the public repository of all available Python code and where we pull stuff from when we do `pip package`,  we would want to make sure it doesn't conflict with other names. Since we're just sharing it with ourselves and maybe a couple of other people on our team, it's fine for now. 

Note: there are IDEs (PyCharm) and [programs](https://github.com/audreyr/cookiecutter-pypackage) that set up a cookiecutter Python application for you, but I've found these to be too complicated when you're just starting out. 

It's important to understand what is actually being set up, and not to get too bogged down in the implementation/continuous integration details that you might not have to deal with at first. 

So we want something, at the very basic level, like this: 

```
 textedit
├── replace.py
├── wordcount.py
├── spacing.py
└── readability.py

```

Once we look at this, we might see that some modules belong together logically. For example, `wordcount` and `spellcheck` just take the text and analyze it. And, in fact, in the Word menu bar, they're grouped together: 


![wordmenu](https://raw.githubusercontent.com/veekaybee/veekaybee.github.io/master/images/wordcountmenu.png)

Which brings me to a (probably obvious) tip: Try to base your software on existing, successfully-tested patterns in the real world instead of trying to greenfield it. Chances are someone has already solved the exact problem you're trying to solve. 

`replace` and `spacing` actually edit the document and create new copies of files. So you might want something like the way these functions are organized in the Word menu. And then you have your text documents that you're actually testing the functionality on. Those should be separate from the code. 

```
 textedit
	├── edit
		├── replace.py
		├── spacing.py
	├── review
		├── wordcount.py
		└── readability.py
	├── texts
		├── alice.txt
		└── pool_of_tears.txt

```

At its very basic level, this is what our code structure looks like. 

## <a id="project-structure"></a>Project structure

But, there are signals we need to give to Python to read this correctly.  CPython also needs to know how these things relate to each other in order to compile a package for us. And the code needs to be built in a way such that other people can seamlessly download it and use it without a lot of explanation from you. 

Which is why, if you look at some popular Python project folder structures,like [Pandas](https://github.com/pandas-dev/pandas), [Requests](https://github.com/requests/requests), or [Flask](https://github.com/pallets/flask), you'll see they tend to be a bit more complicated.  

Generally, it's a good idea to look at other people's work, so here are some [smaller Python packages to take a look at.](https://news.ycombinator.com/item?id=9896369)  

So how do you go from two scripts to a fully-functional package? 

### <a id="modular-code"></a>Modular Code

First, Python needs to have all of the files of the [project in its same directory and subdirectory:](https://the-hitchhikers-guide-to-packaging.readthedocs.io/en/latest/creation.html#directory-layout). The top-level textedit is the distribution or package, and the lower-level one is the actual module.  

```
 textedit - package
 	textedit - module
		├── edit
			├── replace.py
			├── spacing.py
		├── review
			├── wordcount.py
			└── readability.py
		├── texts
			├── alice.txt
			└── pool_of_tears.txt

```

### <a id="unit-tests"></a>Unit Tests 

Then, you'll  want to add tests. Unit tests help you make sure that your code runs as expected, even as you change it. You usually want to write a test for each function or package. 

There is a whole art/science to unit testing, but for the sake of this example, I'm going to skip it and just write an example test, called `test_wordcount.py`.


Program: 

```python
import unittest
import sys
import os


sys.path.append(os.path.abspath("python_packaging/textedit/textedit/review"))
from wordcount import WC

WC_test = WC('../texts/alice.txt')

class Test(unittest.TestCase):

	wc = ('Words:', 274)
	sc = ('Sentences:', 7)
	cc = ('Letters:', 1120)


	def test_wc(self):
		self.assertCountEqual(WC_test.word_count(),self.wc)

	def test_sentences(self):
		self.assertEqual(WC_test.sentence_count(), self.sc)


	def test_characters(self):
		self.assertEqual(WC_test.character_count(), self.cc)

 
if __name__ == '__main__':
    unittest.main()
    
```
Output:

```
    mbp-vboykis:tests vboykis$ python test_wordcount.py 
...
----------------------------------------------------------------------
Ran 3 tests in 0.001s

OK
```

[When you run this piece of code,](https://github.com/veekaybee/textedit/tree/master/textedit/tests) it will check to make sure that wordcount is doing what you want it to do. Usually, we'll want to have the units broken down and more well-defined, but remember, here, we're focusing on building a package rather than the nuances of testing. 

But, whoa! What's going on up top here?  We have some important changes: 

1. Lots of imports
2. `sys.path.append`
3. `../texts/alice.txt` as a relative file reference. 

Here's where the fun of packaging starts.  Because we now have files in different directories, Python needs to understand how to link those directories together to execute the files. 

Before, we were executing everything in the same folder. But now, Python needs to know what you're talking about when you do `from wordcount import WC`, because that package is not in our current working directory, and it's not in the package path that we have when we did this: 

```python
>>> import os
>>> os.getcwd() #where wer're currently running code
'/python_packaging'

>>> print('\n'.join(sys.path)) # all the paths Python checks for packages
```

So we have to add this `sys.path.append(os.path.abspath("/python_packaging/textedit/textedit/review"))` to our sys path so Python knows where to refrence it from:

```python
>>>sys.path.append(os.path.abspath("/python_packaging/textedit/textedit/review"))
>>> print('\n'.join(sys.path))

python3/3.5.1/Frameworks/
Python.framework/Versions/3.5/lib/python35.zip
python3/3.5.1/Frameworks/Python.framework/Versions/3.5/lib/python3.5
python3/3.5.1/Frameworks/Python.framework/Versions/3.5/lib/python3.5/plat-darwin
python3/3.5.1/Frameworks/Python.framework/Versions/3.5/lib/python3.5/lib-dynload
/usr/local/lib/python3.5/site-packages
/python_packaging/textedit/textedit/review
```

And there it is, at the end. For now, we have to add `sys.path.append` to every file that we want to import. This is temporary, just to show how that works. We'll fix this later. 

And, speaking of imports, we now have external packages that we're calling: os, re, and sys, common modules. How does our package know how to call those? 


### <a id="requirements.txt-and-package-dependencies"></a>Requirements.txt and package dependencies


```
 textedit - package
 	textedit - module
		├── edit
			├── replace.py
			├── spacing.py
		├── review
			├── wordcount.py
			└── readability.py
		├── tests
			└── test_wordcount.py
	requirements.txt
			
```
We put a requirements.txt module at the top level of our module. This file that will tell people who install the package which Python packages are used in our programs and automatically install them when you run `pip install .` on the package. (We may have to run ``pip install -r requirements.txt` separately, but that's usually an edge case.)

For example, we've used `os`, `sys`, and `re` in building this module so far. These are standard library modules, and come default with every Python install.  It will also sync versions to make sure that they have the exact version you used to develop your code. 

To create a requirements file, you would normally do a  `pip freeze > requirements.txt`. You'll notice that this includes a lot of stuff, basically everything you have in your Python environment, since `pip freeze` doesn't know which specific packages you're using to build your package and there could be interactions.
 

There are several ways to build requirements on a per-project basis, but we're not going to use them here because we want to focus, again, on packaging rather than Python environments. A simpler way to do this is to install [pipreqs and take it from there](https://github.com/bndr/pipreqs). 

`pipreqs --force "/python_packaging/textedit/textedit"`

If you run that, you'll see that the file generated is blank, because the three modules we've included, `os`, `re`, and `sys`, are part of the standard Python library. If we included something like NumPy, we'd get:

```
mbp-vboykis:textedit vboykis$ cat requirements.txt 
nltk==3.2
```

### <a id="documentation"></a>Documentation

Let's add some documentation, as well.  Good documentation is really important, paricularly to someone just coming into your project. And, even more so, for yourself tomorrow morning. :) The easiest way to add documentation is to add a 	`README` to the top level of your module, the same place as `requirements.txt`. 

```
 textedit - package
 	textedit - module
		├── edit
			├── replace.py
			├── spacing.py
		├── review
			├── wordcount.py
			└── readability.py
		├── tests
			└── test_wordcount.py
	README.MD
	requirements.txt
		
		
``` 
Here are some aspirational examples of what [good documentation](https://github.com/PharkMillups/beautiful-docs) should look like, but remember, these are really complicated examples. Here's a [good, simple one.](https://gist.github.com/PurpleBooth/109311bb0361f32d87a2) 

The advanced next level is to generate your documentation with Sphinx or reST. Don't worry about that yet. 

Documentation also includes your code. All of your code should have headers, and your classes should have [docstrings](https://www.python.org/dev/peps/pep-0257/), as well. [Comments are good!](https://www.python.org/dev/peps/pep-0008/)

For headers, there are mixed opinions onto what one should contain, but I think headers help. 

```
"""This module contains x, y
Author: Vicki Boykis
"""
```

For classes, check out Scikit-learn; it's really good at [documentation](https://github.com/scikit-learn/scikit-learn/blob/master/sklearn/cluster/k_means_.py)


### <a id="scripts"></a>Scripts

If you have any shell scripts or additional helper methods associated with your project, you can create a `bin` directory for them.  We don't, so that folder will be empty. But scripts are usually present anywhere you have to deploy stuff, add it to cron, or generally put it in production in any way. 

This is also where continuous integration can come in. 

```
 textedit - package
 	__init__.py
 	bin/ # scripts
 	textedit - module
 		__init__.py
		├── edit
			├── replace.py
			├── spacing.py
		├── review
			__init__.py
			├── wordcount.py
			└── readability.py
		├── tests
		    __init__.py
			└── test_wordcount.py
	README.MD
	requirements.txt
			
```

## `__init__.py`

And, finally and most importantly, the `__init__.py`, which we'll want to add to every directory where you have runable Python modules. 

If you remember from the Python innards overview `__init__` is a special file that will make Python realize that you have a package working together instead of a [single file.](https://docs.python.org/3/tutorial/modules.html#packages)

We can leave it null. Or you can [add things to it](http://mikegrouchy.com/blog/2012/05/be-pythonic-__init__py.html) that will initialize when the module is run.  Let's leave it null for now. 

When Python imports the module for the first time, it checks the module registry for a list of modules that it can use. `Init` allows your module to be [put in that registry.](http://effbot.org/zone/import-confusion.htm#what-does-python-do). 

### <a id="__main__.py-driver"></a>`__main__.py` driver

There is this concept in Java of a driver program that you can run and have it call all the other programs in the package. 

In Python, we can also have a `__main__.py` file, which will execute everything in it first and pull in from other directories. This might be helpful if you're pre-executing things, like for example a script has to kick off right away to load the word document, for example. 

Here, since we're just performing different things to a document in a single state, we don't really need it.  

Additionally, there are mixed thoughts about having a driver. Google's Python code, for example, says that every [file should have a name/main pattern.](https://google.github.io/styleguide/pyguide.html#Main)
 
```
 textedit - package
 	__init__.py
 	__main__.py
 	bin/
 	textedit - module
 		__init__.py
		├── edit
			├── replace.py
			├── spacing.py
		├── review
			__init__.py
			├── wordcount.py
			└── readability.py
		├── tests
			└── test_wordcount.py
	README.MD
	requirements.txt
			
```

### <a id="setup.py"></a>`setup.py`


Now that we have the scaffolding in place, we can add things that will help us set up the module after we import it from pip or download it. 

This is `setup.py`, the heart of a project. 

It has metadata about the project, `import distutils`, which does the actual program setup whenever someone downloads it, as we specified in the `README.md`, and be at the [top level of the package.](https://the-hitchhikers-guide-to-packaging.readthedocs.io/en/latest/creation.html#setup-py-description). 

Here's more [really good background](https://github.com/kennethreitz/setup.py) on what goes into the file.  

Here's what ours will look like. The most important part is setting the `here` variable to the current filepath. 

Remember how we had to do crazy things to get our modules to reference each other in the context of the package? 
`sys.path.append(os.path.abspath("python_packaging/textedit/textedit/review"))`

This lets us avoid all of that and forces a root directory.
 
```python
import io
import os
import sys
from shutil import rmtree

from setuptools import find_packages, setup, Command

# Package meta-data.
NAME = 'textedit'
DESCRIPTION = 'A simple Python text editor for Victorian authors who just want to write. '
URL = 'https://github.com/veekaybee/textedit'
EMAIL = 'vickiboykis@gmail.com'
AUTHOR = 'Vicki Boykis'

# What packages are required for this module to be executed?
REQUIRED = [
    # 'requests', 'maya', 'records',
]

# The rest you shouldn't have to touch too much :)
# ------------------------------------------------
# Except, perhaps the License and Trove Classifiers!
# If you do change the License, remember to change the Trove Classifier for that!

here = os.path.abspath(os.path.dirname(__file__))



# Where the magic happens:
setup(
    name=NAME,
    #version=about['__version__'],
    description=DESCRIPTION,
    # long_description=long_description,
    author=AUTHOR,
    author_email=EMAIL,
    url=URL,
    packages=find_packages(exclude=('tests',)),
    # If your package is a single module, use this instead of 'packages':
    # py_modules=['mypackage'],

    # entry_points={
    #     'console_scripts': ['mycli=mymodule:cli'],
    # },
    install_requires=REQUIRED,
    include_package_data=True,
    license='MIT',
    classifiers=[
        # Trove classifiers
        # Full list: https://pypi.python.org/pypi?%3Aaction=list_classifiers
        'License :: OSI Approved :: MIT License',
        'Programming Language :: Python',
        'Programming Language :: Python :: 2.6',
        'Programming Language :: Python :: 2.7',
        'Programming Language :: Python :: 3',
        'Programming Language :: Python :: 3.3',
        'Programming Language :: Python :: 3.4',
        'Programming Language :: Python :: 3.5',
        'Programming Language :: Python :: 3.6',
        'Programming Language :: Python :: Implementation :: CPython',
        'Programming Language :: Python :: Implementation :: PyPy'
    ],

)

```

Now, once we run setup.py like the README specifices, we can follow the accepted module import naming structure, which is:

`import module.submodule`

For us, it would be: 

`import edit.replace`

The rest is metadata about who you are and what the package looks like. It's really top-loaded for use in PyPI, so I removed those components and kepts the ones necessary for use locally. 

One of the things included in here is the `install_requires` which lists the requirements that will be installed for the file to continue. But wait, you say. Don't we already have a `requirements.txt` file? Yes, we do. We'll be using that one, instead. 

The difference between the two is that `install_requires` is configured more if you're packaging for production-ready systems and putting your package in PyPi, the Python package repository. If you are targeting specific packages in [development or testing, use requirements.txt.](https://www.reddit.com/r/Python/comments/3uzl2a/setuppy_requirementstxt_or_a_combination/) 

Since we're not exactly focusing on targeting PyPi here to keep things simple, we'll use requirements. 

So now, finally we have: 

```
 textedit - package
 	__init__.py
 	setup.py
 	textedit - module
 		__init__.py
 		├── texts
 			├── alice.txt
			├── pool_of_tears.txt
		├── edit
			__init__.py
			├── replace.py
			├── spacing.py
		├── review
			__init__.py
			├── wordcount.py
			└── spellcheck.py 
		├── tests
	README.md
	requirements.txt
	
			
```

Now that we have everything set up, let's convert the rest of the code to more OOP-like practices, clean it up, and take a look. 

An extra step we can take here is to make sure that none of the modules actually have an `if __name__ == '__main__'` piece of code so that they don't run on their own, and create that driver module we were talking about so that there is only one entry point to the package. 

You can see that in action here, [for example](https://github.com/pallets/flask/blob/master/flask/cli.py#L721). that's a matter of style and might or might not make sense for your program. 

 

## <a id="sharing-and-using-our-package"></a>Sharing and using our package

![wordmenu](https://raw.githubusercontent.com/veekaybee/veekaybee.github.io/master/images/alice_08c-alice_flamingo.png)

Ok, we're done writing all of our code, our tests, making sure objects are accessible, and we have our structure all layed out. What do we do now? [Let's package it!](https://github.com/veekaybee/textedit)

Let's go to the module level of our package - where our `setup.py` file that will actually handle the install is located

`mbp-vboykis:textedit vboykis$ cd /python_packaging/textedit/textedit`

and run 

`pip install .`

You should see something like this: 

```
mbp-vboykis:textedit vboykis$ pip install .
Processing /python_packaging/textedit/textedit
Installing collected packages: textedit
  Running setup.py install for textedit ... done
Successfully installed textedit-0.0.0
```

✨ We've installed our package locally! ✨

Let's see that it's in our PYTHONPATH: 

```
python3/3.5.1/Frameworks/Python.framework/Versions/3.5/lib/python35.zip
python3/3.5.1/Frameworks/Python.framework/Versions/3.5/lib/python3.5
python3/3.5.1/Frameworks/Python.framework/Versions/3.5/lib/python3.5/plat-darwin
python3/3.5.1/Frameworks/Python.framework/Versions/3.5/lib/python3.5/lib-dynload
/usr/local/lib/python3.5/site-packages
/usr/local/lib/python3.5/site-packages/textedit-0.0.0-py3.5.egg
```

And now, let's write some code with it! We can either use the command line to run individual modules:

```bash
#Replace 

python replace.py ../texts/alice.txt "Alice" "Dora the Explorer"
Old Wordcount ('Words:', 274)
WC __name__: WC
New Wordcount ('Words:', 281)
replace __name__: __main__

#WordCount

mbp-vboykis:review vboykis$ python wordcount.py "../texts/alice.txt"
('Words:', 274)
('Sentences:', 7)
('Letters:', 1120)

# Readability

mbp-vboykis:review vboykis$ python readability.py "../texts/alice.txt"
('Reading Grade Level of Text:', 7)
```

Or we can now call the package from a completely new program, since we have it installed. Note the import structure. Each folder and subfolder is `folder.subfolder`, and each module name is after the import. Classes are referenced later down. 

```python
"""Tests textedit functionality on the pool_of_tears.txt file"""


from textedit.review import readability 
from textedit.review import wordcount 
from textedit.edit import spacing
from textedit.edit import replace


test_file = '/python_packaging/textedit/textedit/texts/pool_of_tears.txt'

# Count words
alice = wordcount.WC(test_file)

print(wordcount.WC.word_count(alice))
print(wordcount.WC.character_count(alice))
print(wordcount.WC.sentence_count(alice))

# Readability Index

print(readability.colman_liau(test_file))

# Change Spacing

sp = spacing.Spacing(test_file)
spacing.Spacing.spacing_check(sp)
print('Spaces replaced')

# Replace Words

replace.Replace(test_file, "Alice", "Dora the Explorer")
print('Words replaces')
```

The next step would be to use [`argparse`](https://docs.python.org/3/library/argparse.html) to pass arguments a bit more cleanly. 

And that's it! We've imported a package that we can now use to write other software. 


## <a id="next-steps"></a>Next steps

![wordmenu](https://raw.githubusercontent.com/veekaybee/veekaybee.github.io/master/images/alice_dodo.png)


🎉 Congratulations! We've built a Python package!  🎉

What's next? A lot. But we're not going to cover any of it in this post, because the goal, again, was to understand the basics of packaging. 

But, now that we've taken our first step into a much larger and more insane world, there are a lot more next steps we can take to make sure our code is clean, secure, and easy to use. 

Here are some great places to start exploring once you've gotten the hang of the above: 

### <a id="testing-environments"></a>Testing environments

I mentioned before that there are several ways to create specific environments to build your applications so you're isolated from whatever else is going on in your Python ecosystem. 

1) Use [virtualenv](https://virtualenv.pypa.io/en/stable/), which acts like a container and isolates the application you're developing to a single environment that just has the packages you're interested in for that specific application. 

2) Use [pipenv](http://docs.python-guide.org/en/latest/dev/virtualenvs/), in which you install packages through `pipenv install requests` instead of pip. It will create a Pipfile, which keeps track of all your libraries. 

Both of these are involved in modifying the `sys.path` that we looked at earlier so that Python knows where to find other files. 

Your next step would be to use either of these instead of your local Python environment. 

### <a id="more-advanced-testing"></a>More advanced testing

Unittest is a great starting point, but there's also `pytest`, `nose`, mock testing (when you have complex object dependencies), and [much, much more.](https://wiki.python.org/moin/PythonTestingToolsTaxonomy) All of these work slightly differently.  


### <a id="continuous-integration"></a>Continuous Integration

Once you build a package, you'll probably want to make changes to it. And push those changes to some remote  version-controlled repository so that others can use them. And you'll want to automate this process so that you're not manually doing `pip install .`.  This process is known as [continuous integration](https://www.thoughtworks.com/continuous-integration). 

There's a number of fantastic tools for Python CI. The most popular one these days is [Travis](http://docs.python-guide.org/en/latest/scenarios/ci/), which involves adding a `travis.yml` file that will test your code against your tests and known Python versions. 

### <a id="git-hooks-and-version-control"></a>Git Hooks and Version Control

We haven't touched the subject of version control for your package, but you'll want to add it to specific repositories.  We can also add [pre-commit and post-commit hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) to our code, which mean that the code gets checked, or some specific action is triggered, when you try to check in your code. 

### <a id="setup.cfg"></a>Setup.cfg

In our module, there are no passwords or usernames. But if you're working with databases, webforms, or any number of software options that require you to log in, you'll need a separate `setup.cfg` file that stores your passwords and keeps them safe on your local or testing machine while not sharing them with others. 
	

### <a id="Sphinx/reST"></a>Sphinx/reST

We've already written README.md. But what if you have multiple files that rely on each other? Or you want to use your docstrings to build documentation? [Sphinx and reST](https://thomas-cokelaer.info/tutorials/sphinx/introduction.html) are some ways popular Python packages are documented. 

### <a id="wheels"></a>Wheels

Once you're done refining all of that, and you're ready to go to production, you should build a wheel. Python [wheels](https://packaging.python.org/tutorials/distributing-packages/#wheels) are similar to `JAR` packages in Java, and are a much faster and lightweight process to use in production environments.  You can use `setup.py` to build wheels (which is why it's so important to get it right initially), `python setup.py bdist_wheel --universal`.

For much, much more info on wheels, see [here](http://pythonwheels.com/.)


### <a id="PyPi"></a>PyPi

This is the big one. If your module is stable enough, you [can release it to PyPi](https://glyph.twistedmatrix.com/2016/08/python-packaging.html), which means anyone in the world can download it through `pip`. There are some [extra hoops you have to jump through here](https://hynek.me/articles/sharing-your-labor-of-love-pypi-quick-and-dirty/), namely in how you configure your setup.py file. 
For an easier way to do this, [Flit](http://flit.readthedocs.io/en/latest/) is a potential option. 

Once you're ready, the whole world can see and use your text editor. 


## <a id="conclusion"></a> Conclusion

Python project structure and packaging can be intimidating, but, if you take it step by step, it doesn't have to be. 

Look at other people's code, particularly smaller, modular projects, break the work up into pieces, and work through it piece by piece, until you're all the way down the rabbit hole. 

Good luck!
