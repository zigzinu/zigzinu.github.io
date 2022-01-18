---
published: true
title: "Truffle을 이용한 이더리움 개발 Hands-on"
category: ethereum
tags:
  - truffle

# toc: true
# excerpt: "hello, i'm a blockchain developer at an IT company in South Korea.
# i like making clear the confusing concepts within blockchain ecosystem.
# also, i am developing my hands-on skills on any type of blockchain service."
# sidebar:
#   title: "Sample Title"
#   nav: sidebar-sample
---

# Truffle을 이용한 이더리움 개발 Hands-on

[Truffle 공식 사이트](https://trufflesuite.com/tutorial/index.html)에 제시된 튜토리얼을 따라 실습한다.

## Set up

### nodejs

```command
# curl
sudo apt-get update
sudo apt-get install curl

# nvm latest LTS 버젼 설치
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
# restart terminal
nvm --version

# nodejs latest LTS 버젼 설치
nvm ls-remote --lts # nodejs latest LTS 버젼 확인 (v16.13.2)
nvm install 16.13.2
nvm ls
node -v
```
- [nvm LTS](https://github.com/nvm-sh/nvm#long-term-support) 최신버젼 출처
- `nvm ls-remote` 명령어 실행시 `N/A` 결과가 나올 경우: [stackoverflow](https://stackoverflow.com/questions/26476744/nvm-ls-remote-command-results-in-n-a)

### truffle

```command
npm install -g truffle
truffle version 
# Truffle v5.4.29 (core: 5.4.29)
# Solidity v0.5.16 (solc-js)
# Node v16.13.2
# Web3.js v1.5.3
```
- `npm ls -g` 명령어로 설치된 패키지들을 조회해보면 solidity나 web3 라이브러리는 설치되어있지 않다.
다만 truffle 에서 사용하기 위해 내부적으로 라이브러리가 포함된 것으로 추정된다.
truffle 튜토리얼 중에는 `solidity ^0.5.0`버젼을 활용하는 것으로 보아 지금은 새로 설치하지 않고 
추후에 스마트 컨트랙트를 개발할 시점에 최신 버젼의 solidity를 설치하여 dependency 에 추가해서 사용하겠다.

<details>
<summary>solidity 업그레이드 관련 노트</summary>
<div markdown="1">

1. Path에 등록된 truffle 명령어 설치 위치 확인
```
which truffle # /home/gus/.nvm/versions/node/v16.13.2/bin/truffle
```

2. 설치 가능한 solidity 버젼 확인
```
npm view solc versions
```

3. solc 패키지 설치
```
npm install -g solc@0.8.11
```

4. truffle 패키지 dependency 수정
```
vi /home/gus/.nvm/versions/node/v16.13.2/lib/node_modules/truffle/package.json
```

```
...
"bundleDependencies": false,
  "dependencies": {
    ...
    "solc": "^0.4.15"
  },
```

- [stackexchange](https://ethereum.stackexchange.com/questions/17551/how-to-upgrade-solidity-compiler-in-truffle/47244)

</div>
</details>

### Ganache

필자는 nodejs 및 truffle 라이브러리는 가상머신 위의 우분투 서버 설치했고, [Ganache](https://trufflesuite.com/ganache/)는 호스트 윈도우에 설치했다.

## Tutorial

### 1. 프로젝트 생성 

```
mkdir ~/pet-shop-tutorial
cd ~/pet-shop-tutorial
truffle unbox pet-shop
```
- 튜토리얼을 위해 제공되는 pet-shop 프로젝트를 `unbox` 명령어로 받아온다.
- 새로운 프로젝트를 생성하고 싶다면 `truffle init`명령어를 실행한다.

### 2. 디렉토리 구조

- `contracts/`: 솔리티디 코드 목록 (중요한 컨트랙트: `Migrations.sol`)
- `migrations/`: 트러플은 스마트 컨트랙트 배포를 위해 migration system을 사용한다. 변경 사항을 기록하는 특수한 스마트 컨트랙트라고 생각하면 된다.
- `test/`: 자바스크립트와 솔리디티 테스트 
- `truffle-config.js`: 트러플 설정 파일

### 3. 스마트 컨트랙트 작성

```js
// Adoption.sol
pragma solidity ^0.5.0;

contract Adoption {
  address[16] public adopters;

  // Adopting a pet
  function adopt(uint petId) public returns (uint) {
    require(petId >= 0 && petId <= 15);

    adopters[petId] = msg.sender;

    return petId;
  }

  // Retrieving the adopters
  function getAdopters() public view returns (address[16] memory) {
    return adopters;
  }

}
```

- `pragma`: 컴파일러만 참고하는 정보
- `^0.5.0`: `0.5.0`버젼 이상
- `;`: 문장 종료 syntax
- **statically-typed language**: 솔리디티는 타입을 선언해야하는 언어이다. 컴파일시 타입 체크.
  - 함수 파라미터와 리턴 값 모두 타입이 선언되어야 한다.
- `address`: 어카운트, 또는 스마트 컨트랙트의 주소를 의미하는 솔리디티만의 타입 (20 bytes).
- array: fixed 또는 variable 길이 배열 선언 가능
- `public`: 자동으로 **getter** 함수가 선언된다. Array의 경우 key를 이용해 하나의 값만 리턴 받을 수 있다.
- `require()`: 조건을 확인하는 함수
- `msg.sender`: 해당 함수를 호출한 EOA 또는 CA 주소
- `getAdopters()`: Array의 getter 함수는 1개의 값만 리턴할 수 있기 때문에, 전체 array를 리턴하는 함수를 만들었다.
- `memory`: 변수의 저장 위치를 반환한다.
- `view`: 함수가 컨트랙트의 상태(state)를 바꾸지 않을 것을 명시하는 키워드.
  - `getter` 함수는 기본적으로 `view`로 선언된다.

### 4. Compilation

솔리디티 코드는 bytecode로 컴파일해야 EVM 에서 실행할 수 있다.

```
truffle compile
```
- 프로젝트의 root 디렉토리에서 실행

![](../assets/images/2022-01-18-16-04-01.png)

### 4. Migration

