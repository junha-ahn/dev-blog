---
title: Geth Sync Mode 설명 및 차이점
date: "2021-07-21T08:35:28.062Z"
description: "Geth Sync Mode를 비교해보자"
category: blockchain
draft: false
---

Geth Document와 stackoverflow 등을 번역 후 설명하는 글입니다.

목표는 각 Sync Mode 차이점을 Document 수준에서 이해하는 것입니다.

**혹시 오류 있다면, 댓글 부탁드립니다.**

## [Official Document](https://geth.ethereum.org/docs/getting-started)

**Full 과 Fast Mode 의 공통점은?**
- 모든 블록(headers, transactions, and receipts 포함) 데이터를 다운로드한다.

<br>

**그럼 차이점은?**
- `Full Mode`는 모든 Transaction을 실행하여 state(of the blockchain)을 만들어 나간다

- `Fast Mode`는 모든 Transaction을 실행하지 않고 state를 네트워크에서 직접 다운로드합니다. (다만 최신  몇 block의 transaction은 지속 excute합니다.)

<br>

**그 외 Mode**

- `Snap Mode`: Fast와 동일하지만, 알고리즘이 더 빠르다.
- `Light Mode`: 모든 블록 헤더, 블록 데이터를 다운로드하고 랜덤으로 일부를 확인합니다. (state를 다운로드 하지 않습니다.)

<br>

**사실 아직 이해하기 어렵다**. 더 자세히 설명된 글을 읽어보자

`Geth Config`를 보면 `NoPruning` 라는 옵션이 존재하는데 이에 대한 설명이다

## [Difference between a pruned and unpruned blockchain](https://ethereum.stackexchange.com/questions/1229/difference-between-a-pruned-and-unpruned-blockchain/1234#1234)


과거부터 모든 트랜잭션을 실행시키며 한 블럭, 한 블럭 state를 쌓아가는 것은 **unpruned state/blockchain**라고 합니다

중요한것은 3년전 `account balance`를 신경쓰지 않고, 심지어 현재에만 신경쓰는게 대부분입니다.
- 따라서 이러한 데이터는 필요가 없습니다. (심지어 몇백 기가라는 매우 큰 용량)

<br>

**State pruning**은 결국 이러한 필요없는 `intermediate state`를 버리는것을 의미합니다.
- 따라서 당신은 `past balance`를 알 수 없게 됬지만, 데이터 양을 매우 줄일 수 있습니다.
<br>
<br>

### 그렇다면 Fast Sync는?

만약 당신이 3년전 `balance`를 신경쓰지 않는다면, 왜 블록체인의 전체 `transaction history`를 replay합니까?  (`full mode`) 


그냥 `current state`만 가져오면 되는데? (`Fast Sync`)

`Fast Sync`는 전체 체인이 다운로드 완료됬을때 **state root**(현재 `World View`를 정의하는 해시)를 확인하고 네트워크에서 직접 다운로드합니다.
> World View는 그 시점의 Blockchain 상태라고 생각해도 좋을거 같다. unpruned blockchain은 과거 모든 블록에 대해 각각의 World View가 있었을것이다.

> It does not execute the transactions, generating the world view one block at a time.


즉 `Fast Sync`는 방법이 좀 다른 **puruned database**입니다


## [Geth FAQ](https://geth.ethereum.org/docs/faq)

### Q. How Do Ethereum syncing work?

현재 기본 동기화 모드는 `Fast sync`이다. 
- 처음 블록부터 지금까지 발생한 모든 `transaction`을 재처리하는 대신, 블록을 다운로드하고 `PoW`에 관련된것만 확인 합니다. 
- `Fast Sync`는 `Full Sync`에 비해 빠르게 전체 체인을 재조립할 수 있습니다.

많은 사람들은 블록 데이터가 존재하기 때문에, 동기화 진행중이며 **곧 완료될 것이라고 착각**합니다.
<br>

#### 아직 멀었다고?

왜냐하면 `transaction`이 실행되지 않았기 때문에, 아직 `account state`가 유효하지 않기때문이다

예를들어 특정 `account에 balance`를 알 수 없고 `Node`는 실행 불가능 상태이다.
- 이 `state`는 **별도로 다운로드**해서 최신블록과 교차확인해야 합니다. 

이 단계를 우리는 `state trie download`라고 부른다.
- 사실 블록 다운로드와 동시에 실행되지만, 블록을 다운로드하는 것보다 훨씬 오래 걸린다.

<br>

### 그래서 state trie가 무엇인가? 

- Node가 accout들이 변조되지 않았는지 실제로 **검증**할 수 있도록 각 블록에 암호화적으로 연결된 데이터 구조
- 그 외 설명 생략 

<br>

#### 무엇이 문제인가?

실제로 동기화된 node를 가지려면 모든 계정 데이터와 모든 암호화 state trie을 다운로드해 네트워크를 검증해야 한다.
- 그런데 이것은 이미 **엄청난 양의 데이터**이다. 심지어 실시간 끊임없이 변경되어 당신의 Node는 계속 이 변화에 동기화되어야 한다.
- 더 최악은, 당신이 동기화하는 동안 네트워크는 끊임없이 변경되어 간다는 점이다. 

그런데 실제로 모든 데이터를 수집하기 전까지는 로컬 노드를 사용할 수 없습니다. 
- 동기화되지않은 로컬 노드는 어떤 account도 암호화된 방식으로 증명할 수 없기 때문입니다.

Node가 최신으로부터 `64 block` 뒤에 있다면, 당신은 아직 동기화완료되지않았다. 
- 심지어 금방 끝나지도 않는다. 
- 당신은 그저 블록 데이터 다운로드 단계를 끝낸것이고 **아직 state를 다운로드하는 중**이다. 
- 당신은 이것을 `Imported state entried [...]` 라는 로그로 확인할 수 있다. 

<br>

#### Q. Why does downloading the state take so long, I have good bandwidth?

`state tire` 동기화는 대부분 대역폭(`network bandwidth`)가 아닌 `disk I/O`에 의해 결정된다.
- 디스크에 데이터를 저장하기에 끔찍한 구조이다

그래서 SDD를 추천한다. HDD는 따라잡지 못할 것이다. (다만 light mode는 가능하다)
