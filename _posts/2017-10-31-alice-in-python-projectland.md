---
layout: post
title: "[번역] 파이썬 나라의 앨리스"
tags: [파이썬, python, 번역]
---

이 글은 파이썬 프로젝트 패키징에 관한 [Vicki Boykis](https://veekaybee.github.io/)의 [Alice in Python projectland](https://veekaybee.github.io/2017/09/26/python-packaging/)를 번역한 글입니다. 오역 혹은 그 외 잘못된 부분이 있다면 역자의 [트위터](https://twitter.com/res_tin), [메일](mailto:wintermy201@gmail.com), [PR](https://github.com/JungWinter/JungWinter.github.io)로 알려주시면 수정하겠습니다. 모든 저작권과 권리는 원작자인 [Vicki Boykis](https://veekaybee.github.io/)에게 있습니다.

![](https://raw.githubusercontent.com/veekaybee/veekaybee.github.io/master/images/alice_cards.jpg)


## 서문
Python project structure and packaging standardization is still not a solved problem, something that became even more apparent to me when I recently worked on packaging a machine learning natural language app.

파이썬 프로젝트 구조와 패키징 표준화는 아직 해결되지 않은 문제다. 이는 최근에 내가 기계학습 자연어 응용 프로그램을 패키징할 때 더욱 분명해졌다.


In the JVM, as long as you have your path structured correctly, build tools will understand it and create a package for you into an executable JAR.

JVM에서 [경로가 올바르게 구조화 되어 있다면](https://maven.apache.org/guides/introduction/introduction-to-the-standard-directory-layout.html), 빌드 도구는 이를 이해하고 실행 가능한 JAR로 패키지를 생성한다.


But, when I started looking for the same standardization in Python, it wasn’t as straightforward. Some questions I had as I worked: Should I be using virtualenvs? Pipenvs? Setuptools? Should I have a setup.cfg? What are wheels, or eggs, for that matter? Does each folder need an __init__.py? What does that file even do? How do I reference modules along the same PYTHONPATH?

그러나 파이썬에서 동일한 표준화를 찾기 시작했을 때, 그다지 간단하지 않았다. `virtualenv`, `pipenv`, `setuptools`를 사용해야 하나? `setup.cfg`가 있어야하나? `wheel`이랑 `egg`는 또 뭐고? 각 폴더에 `__init__.py`가 있어야 하나? 그 파일은 대체 뭐야? 동일한 `PYTHONPATH`에서 모듈을 어떻게 참조하지? 같은 여러 의문들이 들었다.


It became apparent that Python’s flexbility, which I really appreciate when I’m buckled down and writing code, makes it a huge pain to operationalize.

내가 코드를 작성할 때 정말 고맙게 생각하는 파이썬의 유연성이 운영에 큰 고통을 준다는게 분명해졌다.


As I worked, I thought I’d write down everything I learned and built up to a complete Python project, from first principles, through a simple example, and thought it might help other people working through the same issues, as well.

첫 번째 원칙으로부터 간단한 예제를 통해 완전한 파이썬 프로젝트를 만들면서 배웠던 모든 것을 적었고, 같은 문제를 겪는 사람들에게 도움이 될 것 이라고 생각한다.


Come with me on a voyage of magic, adventure, and really annoying relative path references to find out how and why Python packaging works the way it does.

파이썬 패키지가 어떻게 그리고 왜 그렇게 작동하는지 찾기 위한 마법과 모험 그리고 정말 성가신 상대 경로 참조의 모험을 함께하자.


This post goes through:

이 글은 아래의 순서로 이루어져있다.


![](https://raw.githubusercontent.com/veekaybee/veekaybee.github.io/master/images/package_flow.png)


To comfortably go through the content, you should be reasonably comfortable with Python (aka if you know what a list comprehension is and how it works you should probably be good), and have some understanding of object-oriented programming basics.

이 글을 잘 이해하기 위해선 파이썬에 익숙해야하고([리스트 컴프리헨션](http://effbot.org/zone/python-list.htm)이 무엇인지 알고 어떻게 동작하는지 안다면 더 좋다), 객체 지향 프로그래밍 기초에 대한 이해가 있어야한다.


All of the code is here.

모든 코드는 [여기](https://github.com/veekaybee/textedit/tree/master/textedit)에 있다.


My hope is that this post becomes a living document, so if you see something egregiously wrong, or something I missed, feel free to submit a pull request.

이 글이 살아있는 글이 되길 원하므로 뭔가 심각하게 잘못되거나 내가 놓친 부분을 발견한다면 [PR](https://github.com/veekaybee/textedit/pulls)을 보내주길 바란다.


## Python hides the hurt 파이썬이 숨긴 상처
I’m going to start this Python post with a little Java. Sorry in advance.

파이썬 글을 조금의 자바 코드로 시작할텐데, 미리 양해를 바란다.


The reasoning behind this is that having another language for comparison really helps sometimes. I’m currently working through a certificate in computer science, and most of my classes so far have been in Java. I’ve previously done mostly dynamically-typed Python, and R, which is way out there, and Scala, but for Spark, which is a separate beast altogether, so I was a little intimidated by the shackles of Java syntax.

왜냐면 다른 언어와 비교하는 것은 때때로 도움이 되기 때문이다. 나는 현재 컴퓨터 과학쪽에서 일하고 있으며 지금까지 대부분의 강의를 자바로 진행했다. 이전에는 스파크를 예외로 치면 보통 동적 타입 언어인 파이썬과 R, 스칼라로 진행했었기에 자바 문법의 족쇄때문에 조금 자신이 없다.


For instance, if I want to read a text file, change some text in it, and output to a new file, Java requires a lot of scaffolding:

예를 들어 텍스트 파일을 읽고, 텍스트를 변경한 후 새로운 파일로 내보내려면 자바는 준비 단계가 많이 필요하다.

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

But Python makes it relatively painless:

파이썬에선 한결 덜 고통스럽다.

```python
# Replaces all instances in a file from "Alice" to "Dora the Explorer"
# 파일에 있는 모든 "Alice"를 "Dora the Explorer"로 바꾼다

with open('alice.txt', 'r') as input:
		with open('new_alice.txt', 'w') as output:
				for line in input:
						line = line.rstrip()
						newline = line.replace("Alice", "Dora the Explorer")
						output.write(newline)
```

For a program of a couple lines, you might not want to use Java. But a benefit all of Java’s scaffolding and type safety, and, particularly its build tools, is that it makes large programs easier to package.

몇 줄짜리 프로그램이라면 자바를 사용하지 싫을 수 있다. 그렇지만 자바에 있는 안전한 타입과 여러 준비 단계, 특히 빌드 도구들은 큰 프로그램을 패키징 하기 쉽다는 이점이 있다.


In Java, you could have three classes, put them in a project structure called project/src/main/java/program and call package on them, and they’ll automatically reference each other. Each program is automatically split logically into a single class. You can compile the separate programs in Maven. And you’re done.

3개의 클래스가 있다면 클래스들을 `project/src/main/java/program` 이라는 프로젝트 구조에 넣고, `package` 를 호출 하면 서로를 자동으로 참조한다. 각 프로그램은 단일 클래스로 자동으로 분리 되고, Maven에서 분리된 프로그램을 컴파일 하면 끝난다.


But, since Python abstracts types, objects, and paths away from the user, and there’s no centralized build system, it becomes a bit more complicated internally.

그러나 파이썬은 타입, 오브젝트, 경로들을 추상화하고 빌드 시스템이 없기 때문에 패키징이 내부적으로 조금 더 복잡해진다.


To understand what Python abstracts away and why this leads to different architecture choices, let’s start at The Beginning.

파이썬이 추상화하는 것과 왜 다른 아키텍처를 가지게 됐는지 이해하기 위해 `처음`부터 시작해보자.

## 간단한 워드 프로세서 만들기
![](https://raw.githubusercontent.com/veekaybee/veekaybee.github.io/master/images/rabbit.jpg)

Let’s say I am Lewis Carroll, and I’m writing Alice in Wonderland. But instead of using ink and paper, I’d like to use some 21st century technology.

내가 이상한 나라의 앨리스를 쓰고 있는 루이스 캐롤이라 하자. 그러나 잉크와 종이 대신 21세기 기술을 사용하려 한다.


What kinds of stuff do authors usually like to do to books, that can be easily automated with a program? Fixing spacing after a period from single to double, spellcheck, replacing words in entire texts, and word count are some common text editing tasks. Authors like word editors.

저자가 책을 쓰면서 자주 하는 일 중에 프로그램을 이용해 쉽게 자동화할 수 있는 것이 무엇이 있을까? 마침표 뒤에 공백 넣기, 맞춤법 검사, 글자 바꾸기, 글자 수 세기가 빈번하기 때문에 저자는 워드 프로세서를 애용한다.


We’re going to create a really, really (really) simple version of Word to demonstrate how Python packaging works, drilling down through internals and hopefully having some fun along the way.

우리는 이제 [정말 정말 단순한(정말로) Word](https://github.com/veekaybee/textedit)를 만들면서 파이썬 패키징이 어떻게 작동하는지 내부적으로 파헤칠 것 이다. 부디 즐겁기 바란다.

## Creating a single object 단일 객체 만들기
![](https://raw.githubusercontent.com/veekaybee/veekaybee.github.io/master/images/alice_door.gif)

![](https://raw.githubusercontent.com/veekaybee/veekaybee.github.io/master/images/model.png)

![](https://raw.githubusercontent.com/veekaybee/veekaybee.github.io/master/images/object_props.png)

```java
public class Alice {

    public static void main(String[] args) {
        String x = "Alice";
    }

}
```

```python
>>> id(x)
4409066472
```

```python
>>> x = "Alice"
>>> type(x)
<class 'str'>
>>> type(str)
<class 'type'>
```

```python
>>> x.__doc__
"str(object='') -> str\nstr(bytes_or_buffer[, encoding[, errors]]) -> str\n\nCreate a new string object from the given object. If encoding or\nerrors is specified, then the object must expose a data buffer\nthat will be decoded using the given encoding and error handler.\nOtherwise, returns the result of object.__str__() (if defined)\nor repr(object).\nencoding defaults to sys.getdefaultencoding().\nerrors defaults to 'strict'."
```
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
