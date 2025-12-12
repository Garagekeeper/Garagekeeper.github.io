---
title: "인증, Hash Function"
date: 2025-12-12 17:27:49 +0900
description: 인증과 해쉬함수에 대해서 알아보자
categories: [Computer Science, Security] #[upper, lower]
tags: [cs] #must be lower case
math: true
mermaid: true
---

## 정보보호의 목표
 * 기밀성: 인가된 사용자만 접근가능(수동적인 공격으로부터 보호)
   * 위협요소: 키를 보유하지 않은 사람이나 프로세스에게 노출, 트래픽 분석
<br>
 * 무결성: 전달되는 메시지가 임의로 변경되면 안됨, 인가된 사용자에의해 인가된 방법으로 진행되어야함(by 메시지 인증)
<br>
 * 인증: 송신자 인증
    * 위협요소
      * 위장(송신자와 다른 사람이 송신자인 척)
      * 내용 수정(메시지 내용의 변경)
      * 순서 수정(메시지 순서의 변경)
      * 시간 수정(메시지 지연, 재전송)
 * 부인봉쇄: 수신자가 수신 부인, 발신자가 발신 부인하지 못하도록 하는 것
<br>

## 메시지 인증
 <br>
 메시지의 무결성을 기술적으로 증명하는 기법으로 다음과 같은 함수들이 존재한다
 (1) 메시지 암호화: 메시지 전체를 암호화
 (2) 메시지 인증코드(MAC): 고정된 길이의 값을 만드는 메시지 및 비밀키의 공개함수
 (3) 해쉬함수: 임이의 길이를 고정된 길이의 해쉬로 대응 시키는 공개함수
 <br>

### 메시지 암호화
 * 비밀키 암호화 (관용 암호화)
   * 기밀성 제공
     * 공유한 키를 바탕으로 암/복호화
   * 단순한 인증
     * 키를 공유한 사람만 암호문 생성 가능
   * 서명 미제공
     + 수신자의 위조, 송신자의 부인 가능
 
<br>
 * 공개키 암호화 (관용 암호화)
   * 기밀성 제공
     * 공유한 키를 바탕으로 암/복호화
   * 인증 미제공
     * 누구나 공개키에 접근 가능
 
 
### MAC (Message Authentication Code)
 * 메시지에 암호학적 점검감 MAC 추가
   * MAC
     * 비밀키를 사용해 생성된 작은 크기의 data block
     * MAC = $C_k(M)$
  </br>
 * 수신측에서 계산한 MAC이 송신측에서 보낸 MAC과 비교하여 인증
 ![image1](https://velog.velcdn.com/images/garage_keeper/post/14d182e6-a2ee-4ac8-a753-c52796a38408/image.png)
   * 비밀키를 송/수신자만 알고 있다고 가정하면 메시지 인증 가능 (기밀성 가정)
     * 메시지 내용 인증
     * 사용자 인증 (출처)
     * 메시지 순서 인증 (일련번호 사용시)
     
  ![image2](https://velog.velcdn.com/images/garage_keeper/post/24a7fe8b-097d-4a24-85ad-72833896a942/image.png)
   * 메시지 인증
     * A,B만 아는 $K_1$을 통해서 MAC
   * 기밀성 제공
     * A,B만 아는 $K_2$을 통해서 메시지+MAC을 암/복호
 
### Hash Function
MAC의 변형중 하나인 'one-way hash function' $H$ (임의의 길이를 고정된 길이의 해쉬값으로)
* One-way hash function $H$
  * $H$ 자체는 쉽게 계산이 가능해야하고, 그 역은 어려워야한다
  * 동일한 해쉬값을 가지는 입력을 찾기 힘들어야한다.
    * 뒤에서 2nd-preimage resistance, coillision resistance 
  * $H$는 MD(Message Digest)라 불리기도 한다.
* MDC(Modification Detection Code)
``교재에서는 Manipulation Detection Code라 되어있는 걸 발견했는데 비슷한 의미인듯``
![image3](https://velog.velcdn.com/images/garage_keeper/post/da22cd4a-a07a-4a16-8674-1ea588da63db/image.png)
  
  * 키값이 없는 함수
  * 일방향 해쉬 함수, 충돌 회피 해쉬 함수의 특징을 가진다
    * 일방향:
      * compression
      * ease of compute
      * preimage resistance
      * 2nd-preimage resistance
    
    * 충돌 회피 함수:
      * compression
      * ease of compute
      * 2nd-preimage resistance
      * collision resistance


### Construction of Hash Function
![image4](https://velog.velcdn.com/images/garage_keeper/post/2706d4a2-968b-44e3-9eb2-76a0a8a9de61/image.png)
* CBC 모드와 비슷함
* 메시지를 잘라서, 함수에 넣은 결과를 누적시키는 방식
* 임의의 길이 $\rightarrow$ 고정길이(+ 고유특성)
* 1bit 변화가 생기면 결과는 전혀 다른 값이 됨



### Hash Function
* Bitwise-XOR
  * 2개 이상의 변환에 취약/ 보안에 취약 (2개의 비트의 변화를 찾아내지 못함)
  * 같은 출력값을 찾기 쉬움(collison)
  * Can be improved by rotating the hash code after each block is XORed into it (???)
</br>
* MD5
  * 128bit 알고리즘
  * UNIX 등에서 패스워드를 저장하는데 사용
</br>
* SHA,SHA-1
  * Secure Hash Algorithm
  * 160bit 알고리즘
  * MD5에서 다이제스트의 길이가 더 길어지게 확장
</br>
* 해쉬 함수의 안정성은 hash value의 길이에 좌우 (길이가 길면 더 다양한 값이 나옴, 그렇다고 너무 길면 안됨)

### MD5
![MD5_DISCRIPTION](https://velog.velcdn.com/images/garage_keeper/post/738327b3-8d88-4e68-a39c-f954b2b4c5d9/image.png)
* 입력: 임의의 길이 메시지
* 출력: 128비트 메시지 다이제스트
* 입력처리: 512비트 블록단위 처리

#### Process
1. 패딩 비트의 부가 (메시지의 길이를 512의 배수로 맞춤 단 64비트는 제외하고, 1000000...의 형식)
   ex) 480bit message $\rightarrow$ 960 message(padded) + 64 = 1024
2. 메시지 길이의 부가: 위에서 남겨둔 64bit에 메시지 길이를 기록 (여기까지가 전처리 단계)
3. MD 버퍼의 초기화: Little endian 형태로 4개의 레지스터에 값을 초기화 (4*32)
   ex) $A=67452301\rightarrow A:01\space23\space45\space67$
4. 512bit(16 word) 블록의 메시지 처리: 4개 round, $T[i]$= 행렬 T에서 [i]번째 32비트 단어
5. 출력
![total](https://velog.velcdn.com/images/garage_keeper/post/8ff30ba7-f275-4cc1-b65c-49c7d49659ed/image.png)
<center>전체 과정</center>
  
![1round](https://velog.velcdn.com/images/garage_keeper/post/9624ca71-f419-409b-b0ad-2b559e84e972/image.png)
  <center>1 round</center>
 X[k]: q번째 512비트 블록 중에서 K번째 32비트 단어
 
### SHA
![SHA](https://velog.velcdn.com/images/garage_keeper/post/73dad846-1d2f-44c7-b8aa-4e31dedcfaa7/image.png)
* 입력: 임의의 길이 메시지
* 출력: 160비트 메시지 다이제스트
* 입력처리: 512비트 블록단위 처리

#### Process
* MD5 방식과 동일(step수의 차이, 20step per round)
![total](https://velog.velcdn.com/images/garage_keeper/post/360cb2cf-936e-45ce-ac33-2ccd5719897c/image.png)
<center>전체 과정</center>
![round](https://velog.velcdn.com/images/garage_keeper/post/92c90c26-cd4f-4d03-ad48-bd48c25221a4/image.png)
<center>1 round</center>
$W_t=S^1(W_{t-16}\oplus W_{t-14}\oplus W_{t-8}\oplus W_{t-3})$


## 사용자 인증
### 단방향 함수를 이용한 인증
* 사용자가 ID,Password를 입력,
* 서버에 저장된 해쉬값과 사용자가 입력한 password의 해쉬값을 비교
### 대칭키 암호화 방식을 사용한 인증
* 서버에서 난수 R을 선택하여 사용자 A에 전송
* A는 난수 R을 비밀키로 암호화한 C를 서버에 전송
* 서버는 사용자의 비밀키로 C를 복호화하여 R을 구해서 R을 비교
* **안전한 키를 가진다는 전제!**
### 공개키 암호화 방식을 사용한 사용자 인증
* 기존 공개키 암호화의 변형
* 서버가 난수 R을 선택하여 A에게 전송
* A는 자신의 개인키를 통해 C 생성
* 서버는 사용자의 공개키를 통해 복호화
* 기밀성은 보장하지 않는다(누구나 공개키를 통해서 확인 가능)
