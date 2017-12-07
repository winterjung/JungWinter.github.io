---
layout: post
title: File on blockchain 개발기
tags: [blockchain, smart contract, python, flask]
---

## 이번 글은
파일의 해시값을 블록체인에 올리고, 파일이 원본임을 확인하고, 파일의 정보를 조회할 수 있는 기능을 개념 증명(Proof of concept) 수준에서 구현한 [File on blockchain](https://github.com/JungWinter/file-on-blockchain)이라는 서비스를 개발했다. 이번 글을 통해 **solidity 코드를 작성하면서 해결한 문제** 와 **파이썬 서버와 어떻게 결합했는지** 돌아보고자 한다.

## 복습
앞서 [파이썬으로 스마트 컨트랙트 개발하기](https://winterj.me/smart-contract-with-python/) 글을 통해서 파이썬 서버를 운영하면서 동시에 `geth`, `solc`, `web3.py`를 이용해 javascript가 아닌 파이썬으로 간단한 스마트 컨트랙트를 배포하고 사용하는 법에 대해 알아봤다.

![저번에 한 걸 요약하면]({{ site.baseurl }}/images/20171127/finish.png)

> 작성한 코드는 [여기서](https://github.com/JungWinter/file-on-blockchain/blob/master/example/deploy_contract_and_test.py) 볼 수 있다.

## File on blockchain 이란?
PoC 수준에서 간단한 원본 증명 서비스를 제공한다. 사용자는 파일을 업로드 할 수 있으며 파일의 데이터는 서버에, 파일의 메타 정보는 블록체인에 기록된다. 다른 사용자가 어떤 파일이 블록체인에 올라가있는지 확인(원본 증명)하기 위해 파일을 업로드하면 해당 파일의 해시값과 블록체인에 기록된 해시값을 비교하여 등재 여부를 반환해준다. 진정한 원본 증명을 제공하기엔 개선되어야 할 부분이 있지만 블록체인과 서버의 상호작용 부분을 중점으로 뒀기에 크게 고려하지 않았다.

### 참고 사진

<figure class="third">
	<a href="https://winterj.me/images/20171203/02_upload_file.png"><img src="https://winterj.me/images/20171203/02_upload_file.png" alt=""></a>
	<a href="https://winterj.me/images/20171203/03_upload_result.png"><img src="https://winterj.me/images/20171203/03_upload_result.png" alt=""></a>
	<a href="https://winterj.me/images/20171203/04_info.png"><img src="https://winterj.me/images/20171203/04_info.png" alt=""></a>
	<figcaption>파일 업로드와 결과 조회 화면</figcaption>
</figure>


사용자 시나리오
기능
블록체인이 사용된 부분

## 솔리디티로 구현한 기능들
업로드
존재 확인
메타 정보 반환

## 의사 결정 과정

## 문제가 됐던 부분

## 개선해야할 점

## Solidity 코드를 작성하며
Solidity로 스마트 컨트랙트 코드를 작성하며 매우 많은 오류를 만나고 디버깅 과정을 거쳤다. [공식 예제를 분석했을 때](https://www.slideshare.net/wintermy201/smart-contract-and-solidity)나 `Greeter`같은 예제로 사용했을 때는 별다른 문제가 없었지만 `mapping`, `struct`, `view`등의 개념을 실제로 사용될 코드와 결합하는 것은 다른 영역이었다.


File on blockchain은 원본 증명 서비스를 제공하는데 사용자가 파일을 업로드하면 블록체인에 파일의 메타 정보를 저장한다. 후에
