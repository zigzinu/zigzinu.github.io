---
published: true
title: "이더리움 개념 정리"
category: ethereum
tags:

# toc: true
# excerpt: "hello, i'm a blockchain developer at an IT company in South Korea.
# i like making clear the confusing concepts within blockchain ecosystem.
# also, i am developing my hands-on skills on any type of blockchain service."
# sidebar:
#   title: "Sample Title"
#   nav: sidebar-sample
---

# Ethereum Concepts

## Smart Contract

### 변조 불가능성

* 배포된 스마트 컨트랙트는 변조 불가능 하다. 배포하기 전에 철저한 테스트를 거쳐야 한다.
* 변조 불가능한 장부는 각 tx를 누가, 언제, 어떤 내용으로 실행했는지에 대한 추적이 가능하다.
* 변조 불가능한 장부는 탈중앙화된 노드 간에 신뢰를 구축해준다.

### Solidity 

* 정적 타입 언어 (statically typed): 컴파일 시 변수의 타입이 결정되어 에러를 잡을 수 있다.
* 읽기 전용 함수를 정의할 수 있다. 이 함수는 원장에 기록을 남기지 않는다.
* 상태(state)를 변경하는 명령어들은 다음과 같다.
  1. state variable 에 쓰기
  2. event 발생
  3. 또다른 contract 생성
  4. `selfdestruct` 사용
  5. 호출(call)을 이용한 Ether 정송
  6. `view` 또는 `pure` 키워드가 명시되지 않은 함수 호출
  7. low-level call 사용
  8. 특정 opcode 가 포함도니 line assembly 사용 

## Accounts

* 이더리움에는 두 종류의 어카운트가 있다: EOA, CA
* 주소의 크기는 20 bytes (160 bits)
* EOA, CA 모두 이더 밸런스르 가질 수 있다.
  * `address`, `balance`라는 implicit 속성을 가진다.
  * 명시적으로 속성이 없어도 `address(this).balance` 처럼 사용할 수 있다.

### EOA (Externally Owned Accounts)

* CA에 메시지를 보냄으로써 함수를 호출한다.
* 메시지는 `msg.sender`와 `msg.value`라는 implied 속성을 가지고 있다.
* 메시지로 전달한 value 는 CA의 balance에 더해진다.
  
### CA (Contract Accounts)

* 금액을 전송받기 위해서는 함수에 `payable` 수정자를 선언해야 한다.
* 개인키가 존재하지 않아서 tx를 실행할 수 없다. Tx의 목적지가 CA인 경우에만 실행 된다.
* `address`를 통해 EOA를 가진 사용자들이 CA를 호출할 수 있다.

## Ethereum Stack

### Level 1: EVM (Ethereum Virtual Machine)

* 이더리움 스마트 컨트랙트의 실행환경