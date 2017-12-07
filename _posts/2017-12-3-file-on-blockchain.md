---
layout: post
title: File on blockchain 개발기
tags: [blockchain, smart contract, python, flask]
---

## 이번 글은
파일의 해시값을 블록체인에 올리고, 파일이 원본임을 확인하고, 파일의 정보를 조회할 수 있는 기능을 개념 증명(Proof of concept) 수준에서 구현한 [File on blockchain](https://github.com/JungWinter/file-on-blockchain)이라는 서비스를 개발했다. 이번 글을 통해 **solidity 코드를 작성하면서 해결한 문제** 와 **파이썬 서버와 어떻게 결합했는지** 돌아보고자 한다.

## 복습
앞서 [파이썬으로 스마트 컨트랙트 개발하기](https://winterj.me/smart-contract-with-python/) 글을 통해서 파이썬 서버를 운영하면서 동시에 `geth`, `solc`, `web3.py`를 이용해 javascript가 아닌 파이썬으로 간단한 스마트 컨트랙트를 배포하고 사용하는 법에 대해 알아봤다.

<figure>
	<a href="https://winterj.me/images/20171127/finish.png"><img src="https://winterj.me/images/20171127/finish.png" alt=""></a>
	<figcaption>저번에 한 걸 요약한 사진. 파이썬으로 스마트 컨트랙트를 배포하고 호출했다.</figcaption>
</figure>

> 작성한 코드는 [여기서](https://github.com/JungWinter/file-on-blockchain/blob/master/example/deploy_contract_and_test.py) 볼 수 있다.

## File on blockchain 이란?
PoC 수준에서 간단한 원본 증명 서비스를 제공한다. 사용자는 파일을 업로드 할 수 있으며 파일의 데이터는 서버에, 파일의 메타 정보는 블록체인에 기록된다. 다른 사용자가 어떤 파일이 블록체인에 올라가있는지 확인(원본 증명)하기 위해 파일을 업로드하면 해당 파일의 해시값과 블록체인에 기록된 해시값을 비교하여 등재 여부를 반환해준다. 진정한 원본 증명을 제공하기엔 개선되어야 할 부분이 있지만 블록체인과 서버의 상호작용 부분을 중점으로 뒀기에 크게 고려하지 않았다.

### 참고 사진 (클릭 시 커짐)

<figure class="third">
	<a href="https://winterj.me/images/20171203/02_upload_file.png"><img src="https://winterj.me/images/20171203/02_upload_file.png" alt=""></a>
	<a href="https://winterj.me/images/20171203/03_upload_result.png"><img src="https://winterj.me/images/20171203/03_upload_result.png" alt=""></a>
	<a href="https://winterj.me/images/20171203/04_info.png"><img src="https://winterj.me/images/20171203/04_info.png" alt=""></a>
	<figcaption style="text-align: center;">파일 업로드와 파일 메타 정보 조회 화면</figcaption>
</figure>

<figure class="third">
	<a href="https://winterj.me/images/20171203/05_check.png"><img src="https://winterj.me/images/20171203/05_check.png" alt=""></a>
	<a href="https://winterj.me/images/20171203/06_true.png"><img src="https://winterj.me/images/20171203/06_true.png" alt=""></a>
	<a href="https://winterj.me/images/20171203/08_false.png"><img src="https://winterj.me/images/20171203/08_false.png" alt=""></a>
	<figcaption style="text-align: center;">블록체인에 있는 파일인지 확인한 결과 화면</figcaption>
</figure>

### 기능
사용자 입장에서 고려한 기능은 윗 단락에서 언급했으니 [서버의 endpoint](https://github.com/JungWinter/file-on-blockchain/blob/master/app/server.py)를 기준으로 기능을 사펴보자.

#### `/` - 파일 업로드
인덱스 페이지, 파일을 업로드 하는 사용자의 이름을 적고 파일을 업로드 해 결과를 얻을 수 있다. 이 때 `upload_on_blockchain`함수가 실행돼 `filehash`, `filename`, `filesize`, `owner`가 [`FileHashStorage`라는 스마트 컨트랙트](https://github.com/JungWinter/file-on-blockchain/blob/master/app/storage.sol)에 기록된다. 사용자에겐 해당 파일의 해시값이 반환되고 이를 토대로 블록체인에 기록된 파일의 메타 정보를 조회할 수 있다.

업로드된 파일은 Flask 설정에 따라 지정된 폴더에 저장된다. `/uploads/<filename>`로 파일에 접근할 수 있는데 이 부분을 더 분산화 한다면 [ipfs](https://ipfs.io/)를 사용할 수 있겠으나 본래 목적은 아니었기에 개선 사항으로 남겨두었다.

#### `/info/<filehash>` - 파일 메타 정보 조회
파일 해시에 해당하는 파일의 정보를 컨트랙트에서 조회해 메타 정보를 반환한다. 스마트 컨트랙트의 특성상 해시가 `mapping`의 키로 존재하지 않는다면 value의 기본값들이 반환된다. 여기선 간단하게 `filename`, `uploadDate`, `filesize`가 반환되는데 존재하지 않는 파일 해시값이라면 `""`, `1970-01-01`, `0`이 반환된다.

사용자는 파일이 블록체인에 존재함을 알고 파일의 메타 정보를 조회할 때 업로더가 파일을 언제 올렸는지 비교함으로써 원본인지 확인할 수 있다.

- [존재하지 않는 key의 기본 값](https://ethereum.stackexchange.com/questions/10831/how-to-judge-if-a-struct-var-exists-in-solidity)

#### `/check` - 파일 등재 여부 조회
인덱스 페이지에서 파일을 업로드 해 그 파일이 블록체인에 존재하는지 조회한다. 존재한다면 파일의 메타정보 조회, 다운로드가 가능하다.

## 솔리디티로 구현한 기능들
저번에는 홈페이지에서 제공되는 예제로만 테스트해봤는데 이번에는 실제로 서비스에 사용될 기능을 설계하고, solidity를 사용해 [별도의 파일](https://github.com/JungWinter/file-on-blockchain/blob/master/app/storage.sol)로 작성했다. 컨트랙트는 2개가 존재하는데 `Owned`라는 컨트랙트는 접근 권한을 제한하는 Abstract Contract에 가깝다. 실제 기능은 `FileHashStorage`컨트랙트에 기술되어 있다.

- [유용한 Storage 컨트랙트 패턴](https://ethereum.stackexchange.com/questions/13167/are-there-well-solved-and-simple-storage-patterns-for-solidity)

### 변수들
> 보다보면 왜 굳이 이렇게 짰을까스러운 부분이 있지만 첫째로 간단한 구조를 유지하기 위해, 둘째로 이렇게 할 수 밖에 없었기에 어쩔 수 없었다. Solidity에선 mapping의 key를 반환하는 기능(파이썬의 `dict.keys()`)도 없고, 중첩 mapping을 사용할 수도 없다.

#### File
```go
struct File {
    string name;
    uint uploadDate;
    uint size;
}
```
파일의 메타 정보를 저장하는 구조체다. 여기서는 간단하게 파일의 이름, 업로드 된 날짜, 파일의 사이즈만 기록하고 있으며 `uploadDate`는 유닉스 타임스탬프기 때문에 `unit`자료형을 사용한다. 또 `bytes32`가 아닌 `string`을 사용했는데 [Solidity 공식문서에 따르면](http://solidity.readthedocs.io/en/develop/types.html#dynamically-sized-byte-array) `bytes`는 raw byte data, `string`은 UTF-8로 인코딩된 문자열을 저장하는 타입이라고 설명하고있기에 `string`을 사용하였다.

- [bytes와 string 중 무엇을 사용해야 할까](https://ethereum.stackexchange.com/questions/11556/use-string-type-or-bytes32)
- [Solidity 공식문서 - array](http://solidity.readthedocs.io/en/develop/types.html#dynamically-sized-byte-array)

#### files
```go
mapping(string => File) private files;
```
`mapping` 자료형은 파이썬의 딕셔너리와 비슷한 자료구조이며 여기선 `string`이 key, `File`이 value로 지정됐다. 여기서 파일의 해시값이 key로 사용되는 `string`이다. 아래와 같은 형태라고 생각하면 편하다.

```js
files["0xABCD1234"] = {
  name: "test_file.pdf",
  registerDate: 17203124, // Unix timestamp
  size: 154000 // Bytes
}
```

#### fileOwners
```go
mapping(string => string[]) private fileOwners;
```
특정 사용자가 여러 파일을 업로드 했을 때, 그 사용자가 올린 모든 파일의 해시값을 조회하기 위해 선언해둔 변수다. 스마트 컨트랙트의 별도로 getter를 구현해두지 않았기에 현재는 사용할 수 없는 변수지만 향후 기능을 업데이트할 때 참고하기 위해 남겨두었다. `fileOwners["Jung"] = ["0xABCD1234", "0xDEAD4321"]`의 형태를 가지고 있다.

```java
string[] public owners;
```
파일을 업로드한 사용자의 이름을 가지고 있는 리스트다. 서버 기능으로 구현해두진 않았지만 `getOwnerName` 함수에 인덱스 번호를 전달해 사용자의 이름을 반환받을 수 있다. 모든 사용자의 이름을 알고 싶다면 `ownerID` 혹은 `owners.length`로 길이를 가져와 해당하는 만큼 `getOwnerName(i)`를 호출하면 된다. `owners = ["Jung", "Park", ...]`의 형태를 가지고 있다.

```go
uint public ownerID = 0;
```
현재 서비스를 사용한 사람의 수를 알기 위해 선언해 둔 변수다. 이 부분은 `owners.length`가 있으니 굳이 있을 필요가 없지만 차후 개선 사항으로 남겨두었다.


### `upload` - 업로드
```js
function upload(string personName, string fileHash, string fileName, uint fileSize) onlyOwner public {
    ownerID++;
    owners.push(personName);
    File memory f = File(fileName, now, fileSize);
    files[fileHash] = f;
}
```

### `checkExist` - 존재 확인
```java
function checkExist(string fileHash) onlyOwner public view returns (bool) {
    if (files[fileHash].size > 0) {
        return true;
    }
    return false;
}
```

### `getFileInfo` - 파일 메타 정보 반환
```java
function getFileInfo(string fileHash) onlyOwner public view returns (string, uint, uint) {
    return (files[fileHash].name, files[fileHash].uploadDate, files[fileHash].size);
}
```

## 의사 결정 과정

## 문제가 됐던 부분
Solidity로 스마트 컨트랙트 코드를 작성하며 매우 많은 오류를 만나고 디버깅 과정을 거쳤다. [공식 예제를 분석했을 때](https://www.slideshare.net/wintermy201/smart-contract-and-solidity)나 `Greeter`같은 예제로 사용했을 때는 별다른 문제가 없었지만 `mapping`, `struct`, `view`등의 개념을 실제로 사용될 코드와 결합하는 것은 다른 영역이었다.


## 개선해야할 점
- [ ] async upload
- [ ] 특정 사용자가 올린 모든 파일의 해시 리스트 반환
- [ ] 컨트랙트 초기 배포

## 마치며
혹시 잘못된 점이나 궁금한 점이 있다면 언제든지 **wintermy201@gmail.com** 로 메일을 보내주기 바랍니다.
