---
layout: post
title: File on blockchain 개발기
tags: [blockchain, smart contract, python, flask]
---

## 이번 글은
파일의 해시값을 블록체인에 올리고, 파일이 원본임을 확인하고, 파일의 정보를 조회할 수 있는 기능을 개념 증명(Proof of concept) 수준에서 구현한 [File on blockchain](https://github.com/JungWinter/file-on-blockchain)라는 서비스를 개발했다. 이번 글을 통해 **solidity 코드를 작성하면서 해결한 문제** 와 **파이썬 서버와 어떻게 결합했는지** 돌아보고자 한다.

## 복습
[파이썬으로 스마트 컨트랙트 개발하기](https://winterj.me/smart-contract-with-python/)글을 통해서 파이썬 서버를 운영하면서 동시에 javascript가 아닌 파이썬으로 스마트 컨트랙트를 배포하고 사용하는 법에 대해 알아봤다.
11월 21일 화요일, 블록체인 스터디에서 [web3.py](https://github.com/pipermerriam/web3.py)를 이용한 스마트 컨트랙트 배포와 활용에 대해 발표했다. 본래는 PDF파일을 블록체인에 올린 원본 증명 서비스의 간단한 개념 증명을 구현하려 했지만, 초기 계획보다 작업을 많이 하지 못해 2주에 걸쳐 작업하기로 결정했다. 이 글을 통해 **이더리움 노드를 설치해 구동시키고 파이썬을 사용해 스마트 컨트랙트를 컴파일 한 후 배포하기까지의 과정** 과 그러면서 있었던 **여러 문제 상황들을 어떻게 해결했는지** 회고하고자 한다.


![]({{ site.baseurl }}/images/20171127/study.jpeg)

서비스는 파이썬 Flask 서버에서 돌아가며 블록체인 네트워크는 Geth 노드로, py-solc로 스마트 컨트랙트 코드를 컴파일하며, Flask와 Geth 사이의 상호작용에는 web3.py가 사용됐다.

위의 코드와 `json` 파일은 [Github](https://github.com/JungWinter/file-on-blockchain)에 올라가 있다. 혹시 잘못된 점이나 궁금한 점이 있다면 언제든지 **wintermy201@gmail.com** 로 메일을 보내주기 바랍니다.

## 이제 해야할 것
- 제대로된 컨트랙트 코딩
- Flask 서버 제작
- 간단한 프론트 페이지 제작
- 최소한으로 동작하는 모델 만들기
