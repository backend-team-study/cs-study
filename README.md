# CS Study
> 백엔드 개발 기술면접을 대비한 CS 스터디를 진행합니다.
<br>

## 👪 스터디 멤버

|                     [김영주](https://github.com/kylekim2123)                     |                      [고범준](https://github.com/K-jun98)                      |                       [원건희](https://github.com/weonest)                        |                       [문종운](https://github.com/bombo-dev)                        |
|:-----------------------------------------------------------------------------:|:-----------------------------------------------------------------------------:|:-----------------------------------------------------------------------------:|:-----------------------------------------------------------------------------:|
| <img src="https://avatars.githubusercontent.com/u/49775540?v=4" width="150">  | <img src="https://avatars.githubusercontent.com/u/101342145?v=4" width="150">  | <img src="https://avatars.githubusercontent.com/u/98159941?v=4" width="150">  | <img src="https://avatars.githubusercontent.com/u/74203371?v=4" width="150">  |

<br>

## 📌 스터디 진행 방식

| 항목             | 내용                                                         |
| ---------------- | ------------------------------------------------------------ |
| **기간**         | 2023년 10월 23일 ~                                           |
| **장소**         | 온라인 게더타운                             |
| **날짜** | 매주 화요일, 금요일          |
| **방식**         | 각 CS 과목의 키워드에 해당하는 지식을 학습하고, 자주 나오는 면접 질문의 답변을 정리한다.<br />매 스터디 마다 서로가 면접관과 면접자가 되어 모의 면접을 진행한다.|

<br>

## 📝 각종 컨벤션

### 📁 패키지 구조

```
Cs-study
    ├── database
    |   └── 01_관계형_데이터베이스
    |       ├── README.md
    |       ├── 건희.md
    |       ├── 범준.md
    |       ├── 영주.md
    |       └── 종운.md
    ├── network
    |   └── ...
    ├── os
    |   └── ...
    ...
```



### 📍 커밋 및 PR

1. **커밋 메시지**

   ```
   {과목} {내용} 문서 작성(수정) [이름]
   ```

   - 작성 예시) `DB 01_관계형_데이터베이스 작성 [김영주]`
   - 수정 예시) `DB 01_관계형_데이터베이스 수정 [김영주]`

2. **PR**

   ```
   {과목} {내용} 제출 [이름]
   ```

   - PR 예시) `DB 01_관계형_데이터베이스 제출 [김영주]`

<br>

## 🔑 키워드

### 1️⃣ 데이터베이스(DB)

- 관계형 데이터베이스
  - 데이터베이스와 파일시스템의 차이
  - 관계형 데이터베이스의 개념과 장단점
  - DDL, DML, DCL, TCL
  - Key
- MySQL 아키텍처
  - innodb
  - 쿼리동작 방식
- Join
- 이상 현상과 정규화
- 트랜잭션
  - 트랜잭션 개념
  - ACID
  - Commit, Rollback
  - 트랜잭션 격리수준
  - LOCK, 교착상태
- 인덱스
  - 인덱스 개념
  - 인덱스 종류
  - Clustered index, Non-Clustered index
  - 인덱스 자료구조
- Master/Slave
- Sharding
- NoSQL
  - NoSQL의 개념
  - RDB VS NoSQL
  - Redis 동작원리

<br>
 
### 2️⃣ 네트워크(Network)
- 네트워크 레이어
    - OSI 7계층
    - TCP/IP 4계층
    - IP
        - IPv4 vs IPv6
        - subnet
        - CIDR
- 통신
    - TCP
        - 흐름제어, 혼잡제어, 오류제어
        - 3-way-handshake, 4-way-handshake
    - UDP
    - HTTP
        - HTTP status code
        - HTTP method
        - HTTP 1.1, 2.0, 3.0
    - HTTPS, SSL/TLS
    - DNS
    - 기타 : socket, STOMP, SMTP (프로젝트에서 사용한 경우)
- Web
    - Web Server vs WAS
    - Web Server
        - apache vs nginx (동작원리)
        - SSL offloading
        - reverse proxy
        - load balancing
            - L7 vs L4
            - 알고리즘
    - Web cache
    - URI, URN, URL
    - Rest API
- 보안
    - CORS
    - XSS
    - SQL Injection
- 인증
    - cookie
    - session
    - JWT
- 총정리 : [www.example.com을](http://www.example.xn--com-of0o/) 입력했을 때 일어나는 일
