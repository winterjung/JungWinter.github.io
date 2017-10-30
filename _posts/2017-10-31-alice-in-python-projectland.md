---
layout: post
title: \[번역\] 파이썬 나라의 앨리스
tag: python, translation
---

이 글은 파이썬 프로젝트 패키징에 관한 [Vicki Boykis](https://veekaybee.github.io/)의 [Alice in Python projectland](https://veekaybee.github.io/2017/09/26/python-packaging/)를 번역한 글 입니다. 오역 혹은 그 외 잘못된 부분이 있다면 [트위터](https://twitter.com/res_tin), [메일](mailto:wintermy201@gmail.com), [PR](https://github.com/JungWinter/JungWinter.github.io)로 알려주시면 수정하겠습니다.

![](https://raw.githubusercontent.com/veekaybee/veekaybee.github.io/master/images/alice_cards.jpg)

{:toc}

## 서문
Python project structure and packaging standardization is still not a solved problem, something that became even more apparent to me when I recently worked on packaging a machine learning natural language app.

![](https://raw.githubusercontent.com/veekaybee/veekaybee.github.io/master/images/package_flow.png)

## Python hides the hurt
I’m going to start this Python post with a little Java. Sorry in advance.

```java
import java.io.*;

public class Replace {

    public static void main(String[] args) throws IOException {
        File file = new File("alice.txt");

        try (FileReader freader = new FileReader(file);
             BufferedReader reader = new BufferedReader(freader);
             FileWriter writer = new FileWriter("new_alice.txt")) {
            String line;

            while ((line = reader.readLine()) != null) {
                String newLine = line.replaceAll("\\s+$", "").replaceAll("Alice", "Vicki");
                writer.write(newLine);
            }
        }
    }
}
```

```python
# Replaces all instances in a file from "Alice" to "Dora the Explorer"

with open('alice.txt', 'r') as input:
	with open('new_alice.txt', 'w') as output:
		for line in input:
			line = line.rstrip()
			newline = line.replace("Alice", "Dora the Explorer")
			output.write(newline)
```
## 간단한 단어 처리기 만들기
![](https://raw.githubusercontent.com/veekaybee/veekaybee.github.io/master/images/rabbit.jpg)

## Creating a single object
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
