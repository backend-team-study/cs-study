# 02 HTTP, 인증, 보안

## HTTP 1.1 & HTTP 2 & HTTP 3

![Untitled](02%20HTTP,%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC,%20%E1%84%87%E1%85%A9%E1%84%8B%E1%85%A1%E1%86%AB%20810e9edea62a4e37b58a00f169b29a3b/Untitled.png)

### HTTP 1.1 의 등장

HTTP/1.1은 HTTP의 첫 번째 공식 표준 버전으로 GET, POST 외에도 PUT과 DELETE 가 생겼다. 또한, 하나의 TCP 연결을 재사용해 많은 콘텐츠를 전달할 수 있는 지속적 연결 기술 (keep alive) 이 추가되었다.

HTTP/1.1 의 또 다른 특징 중 하나는 `파이프라이닝` 기술이다. 파이프라이닝은 브라우저가 웹 서버에 여러 개의 콘텐츠를 요청했을 때, 이전 요청에 대한 응답을 완전하게 받지 않더라도 지속적 연결로 확보한 하나의 TCP 연결 내에서 미리 다음 요청에 대한 처리를 시작하면서 전체적인 전달 시간을 줄이는 방식이다.

![Untitled](02%20HTTP,%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC,%20%E1%84%87%E1%85%A9%E1%84%8B%E1%85%A1%E1%86%AB%20810e9edea62a4e37b58a00f169b29a3b/Untitled%201.png)

Short-lived connections는 HTTP/1.0, 오른쪽 두 개는 HTTP/1.1에 해당한다.

**Persistent Connection (Keep alive connetcion) 영속적인 커넥션**

영속적인 커넥션은 얼마간 연결을 열어놓고 여러 요청에 재사용함으로써, 새로운 TCP 핸드셰이크를 하는 비용을 아끼고, 성능 향상에 기여할 수 있다. 커넥션이 영원히 열려있는 것은 아니며, 특정한 시간이 지나면 닫히게 된다. (서버는 Keep-alive 헤더를 사용해서 연결이 최소한 얼마나 열려있어야 할 지를 설정할 수 있다.

물론, 영속적인 커넥션도 단점을 가질 수 있다. 실제로 데이터를 보내지 않는 상황에서도 통신을 열어서 유지해야 하기 때문에 서버 리소스를 소비하게 된다.

HTTP/1.0 커넥션은 기본적으로는 영속적이지 않지만, Connection을 close가 아닌 다른 것으로, 일반적으로 `retry-after`로 설정하면 영속적으로 동작한다.

반면, HTTP/1.1은 기본적으로 영속적이다.

**HTTP 파이프라이닝**

HTTP 파이프라이닝은 모던 브라우저에서 기본적으로 활성화 되어 있지 않다 이는 모질라의 문서에 잘 나와 있다.

- 파이프라이닝은 정확히 구현해내기 복잡하다. 전송 중인 리소스의 크기, 사용될 효과적인 RTT (Round Trip Time - 왕복 시간), 그리고 효과적인 대역폭은 파이프라인이 제공하는 성능 향상에 직접적으로 영향을 미친다. 이런 내용을 모른다면, 중요한 메시지가 덜 중요한 메시지에 밀려 지연될 수 있다. 중요성에 대한 생각은 페이지 레이아웃 중에도 진전된다. 그러므로 파이프라이닝은 대부분의 경우 미미한 수준의 향상만을 가져다 준다.
- 파이프라이닝은 `HOL` 문제에 영향을 받는다. (Head Of Line Blocking)
    - HTTP/1.1의 요청-응답 쌍은 항상 순서를 유지하고 동기적으로 수행되어야 한다.
    - 기본적으로 1개의 TCP 커넥션 상에서 3개의 데이터를 받는 경우 요청은 다음과 같이 된다.
    
    ```json
    |---data 1---|
                 |---data 2---|
                              |---data 3---|
    ```
    
    - 하나의 요청이 처리되고 응답을 받은 후에 다음 요청을 보낸다. 이전의 요청이 처리되지 않았다면 그 다음 요청은 보낼 수 없다는 것.
    - 만약 data1에서 시간이 오래 걸린다면 data2, data3 이 아무리 빨리 처리될 수 있더라도 전체적으로 느려지게 된다.
    - 파이프라이닝이라는 사양은 요청만 먼저 보내버리는 것으로, 이 문제를 회피하는 것처럼 보이지만 응답은 항상 보낸 순서로 받아야 하기 때문에 data1 에서 막혔을 경우 생각보다 큰 효과를 보기 어렵다.
- 이러한 이유들로, 파이프라이닝은 더 나은 알고리즘인 멀티 플렉싱으로 대체되었는데, 이는 HTTP/2에서 사용된다.

**도메인 샤딩 ( 도메인 분할 기법)**

도메인 분할 기법은 여러 도메인을 소유한 경우 웹 콘텐츠를 병렬적으로 동시에 다운로드할 수 있도록 하는 방법이다.

브라우저는 동일 호스트 명의 동시 연결 개수를 제한하는데, 한 도메인당 6~13 개의 TCP 연결들을 동시 생성해 여러 리소스를 한 번에 다운로드 할 수 있도록 허용한다. 

| 브라우저 종류 | 최대 동시 연결 개수 |
| --- | --- |
| IE 11.0 | 13 |
| 파이어폭스 | 6 |
| 크롬 | 6 |
| 사파리 | 6 |
| 오페라 | 6 |
| iOS | 6 |
| Android | 6 |

예를 들어 6개의 동시 연결을 지원하는 브라우저에서 도메인 샤딩을 이용해 2개의 도메인을 사용하게 된다면 이론적으로 12개의 연결이 가능해진다.

우리가 [www.steadies.kr](http://www.steady.kr) 이라는 사이트를 운영한다면 추가 도메인을 다음과 같이 디자인 할 수 있다.

- [img.steadies.kr](http://img.steadies.kr) - 이미지 호출 도메인
- [script.steadies.kr](http://script.steadies.kr) - 정적 콘텐츠 도메인
- [api.steadies.kr](http://api.steadies.kr) - API 서비스 도메인

이러한 도메인 샤딩은 사이트 전체의 쿠키 사이즈를 축소 시킬 수도 있다는 장점이 있다.

도메인을 여러 개로 분할하는 방법은 기술적으로 크게 어렵지 않으나, 도메인을 몇 개로 운용하는 것이 최적인지 결정하는 것은 쉽지 않은 문제이다.

> 도메인 샤딩이 고안된 이유는 HTTP/1.1의 가장 큰 문제점으로 지적되어온 HOL 현상 때문이다. HTTP/1.1 에서 클라이언트와 서버 간의 연결은 마치 하나의 차선만 있는 도로와 같다. 클라이언트는 하나의 요청을 서버에 보내고 그에 대한 정상적 응답을 받은 후에야 다음 요청을 보낼 수 있기 때문이다.
> 
> 
> 하지만, 이러한 문제점들은 HTTP/2의 멀티 플렉싱 기술로 해결되었고, 때문에 명확한 이유가 있는 것이 아니라면 도메인 샤딩을 쓰면 안 된다. 도메인 샤딩을 사용하면 오히려 HTTP/2만의 특징인 헤더 압축, 우선순위 전송, 서버 푸시 기능을 방해하기 때문이다. 
> 

Q. 그렇다면 무작정 많은 도메인을 사용하면 더 빨라지는가?

그렇지 않다. 브라우저는 해당 도메인의 IP 주소를 찾기 위해 DNS Lookup 과정을 거치게 된다. 때문에 도메인이 많아지면 많아질 수록 이들에 대한 DNS Lookup 시간, 커넥션을 유지하는데 드는 비용들이 필요해진다.

### HTTP/2 의 등장 배경

HTTP/1.1 의 메세지 포맷은 구현의 단순성과 접근성에 주안점을 두고 최적화되었다. 그러다 보니 성능은 어느 정도 희생시키지 않을 수 없었다. 커넥션 하나를 통해 요청 하나를 보내고 그에 대해 응답 하나만을 받는 HTTP의 메시지 교환 방식은 단순했지만, 응답을 받아야만 그 다음 요청을 보낼 수 있기 때문에 심각한 회전 지연(latency)을 피할 수 없었다.

때문에 더 효육적이고 빠른 HTTP 가 필요했고, 이러한 요구에 만들어진 것이 구글의 SPDY 프로토콜이다. 텍스트 방식의 프로토콜 메시지를 과감히 버리고 이진 포맷을 사용하면서 프로토콜 자체를 경량화하고자 했던 첫 시도이다. 이후 HTTP 워킹 그룹은 SPDY 프로토콜에 몇 가지를 보완해 새로운 정식 HTTP 버전을 제안했고, 이것이 HTTP/2다.

HTTP/2 요청과 응답은 길이가 정의된 (최대 16383 (2^14 - 1) 바이트) 한 개 이상의 프레임에 담긴다. 프레임들에 담긴 요청과 음답은 스트림을 통해 보내진다. 한 개의 스트림이 한 쌍의 요청과 응답을 처리한다. 하나의 커넥션 위에 여러 개의 스트림이 동시에 만들어질 수 있으므로, 여러 개의 요청과 응답을 처리하는 것 역시 가능하다.

HTTP/2는 기존의 요청-응답과는 약간 다른 새로운 상호작용 모델인 서버 푸시를 도입했다. 이를 통해 서버는 클라이언트에게 필요하다고 생각하는 리소스라면 그에 대한 요청을 명시적으로 받지 않더라도 능동적으로 클라이언트에게 보내줄 수 있다.

### HTTP/1.1 과의 차이점

1. 프레임

HTTP/2 에서 모든 메시지는 프레임에 담겨 전송된다. 모든 프레임은 8바이트 크기의 헤더로 시작하여, 최대 16383 바이트 크기의 페이로드가 온다.

![Untitled](02%20HTTP,%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC,%20%E1%84%87%E1%85%A9%E1%84%8B%E1%85%A1%E1%86%AB%20810e9edea62a4e37b58a00f169b29a3b/Untitled%202.png)

1. 스트림과 멀티플렉싱

![Untitled](02%20HTTP,%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC,%20%E1%84%87%E1%85%A9%E1%84%8B%E1%85%A1%E1%86%AB%20810e9edea62a4e37b58a00f169b29a3b/Untitled%203.png)

한 쌍의 HTTP 요청과 응답은 하나의 스트림을 통해 이루어진다. 클라이언트는 새 스트림을 만들어 그를 통해 HTTP 요청을 보낸다. 요청을 받은 서버는 그 요청과 같은 스트림으로 응답을 보낸다. 그러고 나면 스트림이 닫히게 된다.

HTTP/1.1 에서는 한 TCP 커넥션을 통해 요청을 보냈을 때 그에 대한 응답이 도착하고 나서야 같은 TCP 커넥션으로 다시 요청을 보낼 수 있다. 따라서 웹 브라우저들은 회전 지연을 줄이기 위해 여러 개의 TCP 커넥션을 만들어 동시에 보내는 방법을 사용한다. 그렇다고 TCP 커넥션을 무한정 만들 수는 없기 때문에 한 페이지에 보내야 할 요청이 수십개에서 수백게에 달하는 오늘날에는 회전 지연이 늘어나는 것을 피하기 어렵다.

HTTP/2 에서는 하나의 커넥션에 여러 개의 스트림이 동시에 열릴 수 있다. 따라서 하나의 HTTP/2 커넥션을 통해 여러 개의 요청이 동시에 보내질 수 있기 때문에 위 문제는 쉽게 해결될 수 있다. 뿐만 아니라, 스트림은 우선 순위를 가질 수 있다. (그러나 이 우선 순위에 따르는 것은 의무사항이 아니기 때문에, 요청이 우선순위대로 처리된다는 보장은 없다.) `Multiplexed Streams를 이용하여 동시에 여러 개의 메시지를 주고 받을 수 있으며 응답은 순서에 상관없이 Stream으로 주고 받는다. 이로 인해 HOL 문제를 해결할 수 있다.`

모든 스트림은 31비트의 무부호 정수로 된 고유한 식별자를 갖는다. 스트림이 **클라이언트에 의해 초기화되었다면 이 식별자는 반드시 홀수**여야 하며 **서버에 의해 초기화되었다면 짝수**여야 한다. 또한, 새로 만들어지는 스트림의 식별자는 이전에 만들어졌거나 예약된 스트림들의 식별자보다 커야 한다. 한 번 사용한 스트림 식별자는 다시 사용할 수 없다. 커넥션을 오래 사용하다보면 스트림에 할당될 수 있는 식별자가 고갈되기도 하는데, **이런 경우엔 커넥션을 다시 맺으면 된다.**

1. 헤더 압축

HTTP/1.1 에서는 헤더는 아무런 압축 없이 그대로 전송되었다. 과거에는 웹 페이지 하나를 방문할 때의 요청이 많지 않았기 때문에 헤더의 크기가 큰 문제가 되지는 않았는데, 요즘 웹 페이지를 보내기 위해서는 수십, 수백 번의 요청을 보내기 때문에 헤더의 크기가 회전 지연과 대역폭 양쪽 모두에 영향을 미치게 된다.

HTTP2는 클라이언트와 서버 사이에 가상 테이블을 만들어서 동일하고 중복되는 헤더 값들을 테이블에 저장하고 참고하는 방식으로 중복 전달을 제거한다.

가상 테이블은 정적 테이블과 동적 테이블로 나눌 수 있는데, 정적 테이블에는 미리 정의된 자주 사용되는 헤더 필드를 저장한다. 동적 테이블에는 클라이언트와 서버가 주고 받는 값들을 업데이트한다.

또한, 헤더 압축 알고리즘인 HPACK을 사용해 허프만 알고리즘 방식으로 헤더를 압축한다.

1. 서버 푸시

HTTP/2는 서버가 하나의 요청에 대해 응답으로 여러 개의 리소스를 보낼 수 있도록 해준다. 이 기능은 서버가 클라이언트에서 어떤 리소스를 요구할 것인지 미리 알 수 있는 상황에서 유용하다.

예를 들어 HTML 문서를 요청 받은 서버는 그 HTML 문서가 링크하고 있는 이미지, CSS파일, 자바스크립트 파일 등의 리소스를 클라이언트에게 푸시할 수 있을 것이다. 이는 클라이언트가 HTML 문서를 파싱해서 필요한 리소스를 다시 요청하여 발생하게 되는 트래픽과 회전 지연을 줄여준다.

### 알려진 보안 이슈

- 중개자 캡슐화 공격
    - HTTP2 메시지를 중간 프록시가 HTTP1.1 메시지로 변환할 때 메시지의 의미가 변질될 가능성이 있다. HTTP2는 헤더 필드의 이름과 값에 이진 포맷을 사용하기 때문에 어떠한 문자열이든 사용할 수 있게 해준다. 이는 정상적인 HTTP2 요청이 변질된 HTTP1.1 메시지로 변역되는 것을 유발할 수 있다.
- 긴 커넥션 유지로 인한 개인정보 누출
    - HTTP2는 사용자가 요청을 보낼 때의 회전 지연을 줄이기 위해 클라이언트와 서버 사이의 커넥션을 오래 유지하는 것을 염두에 두고 있다. 이것은 개인 정보의 유출에 악용될 가능성이 있다.

### HTTP3의 등장

TCP는 오래된 프로토콜로 성능보다는 기능에 초점이 맞춰져 있다. 그러므로 멀티미디어 콘텐츠를 다양한 기기에 빠르게 전달해야 하는 상황에서 TCP의 한계를 극복하고 최저고하하는 것이 많은 기업들의 과제였다. 한 마디로 요약하자면 TCP가 문제이다. 그래서 UDP를 채택한 HTTP3이 등장하게 된 것이다.

![Untitled](02%20HTTP,%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC,%20%E1%84%87%E1%85%A9%E1%84%8B%E1%85%A1%E1%86%AB%20810e9edea62a4e37b58a00f169b29a3b/Untitled%204.png)

 HTTP3를 알아보기에 앞서 구글이 개발한 QUIC(Quick UDP Internet Connections)를 알아야 한다. QUIC는 UDP를 채택해 TCP의 성능을 개선하려는 기술이다. 전달 속도의 향상과 더불어 클라이언트와 서버의 연결 수를 최소화하고 대역폭을 예상해 패킷 혼잡을 피하는 것이 QUIC의 주요 특징이다.

QUIC는 이전에 클라이언트가 한 번이라도 접속했던 서버라면, 별도의 정보 교환 없이 바로 데이터를 보내는 기술을 소개했다. 이 기능을 Zero RTT라고 한다. 

### HTTP3의 특징

TCP 자체의 HOLB (Head Of Line Blocking)분명 HTTP 2에서 HTTP 1.1의 파이프라이닝 HOLB 문제를 멀티플렉싱(Multiplexing)을 통해 해결했다고 하였다. 하지만 기본적으로 TCP는 패킷이 유실되거나 오류가 있을때 재전송하는데, 이 재전송 과정에서 패킷의 지연이 발생하면 결국 HOLB 문제가 발생된다. TCP/IP 4 계층Visit Website을 보면, 애플리케이션 계층(L4)에서 HTTP HOLB를 해결하였다 하더라도, 전송 계층(L3)에서의 TCP HOLB 를 해결한건 아니기 때문이다.

![Untitled](02%20HTTP,%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC,%20%E1%84%87%E1%85%A9%E1%84%8B%E1%85%A1%E1%86%AB%20810e9edea62a4e37b58a00f169b29a3b/Untitled%205.png)

### 연결 시 레이턴시 감소

기존 TLS + TCP 에서는 TLS 연결을 위한 핸드쉐이크와 TCP 연결을 위한 핸드쉐이크가 각각 발생했다. QUIC에서는 이를 한 단계로 줄였다. 

UDP 위에서 동작하는 QUIC는 통신을 시작할 때 핸드쉐이크 과정을 거치지 않아도 되기 때문에 첫 연결 설정에 1 RTT만 쇼요된다. 그 이유는 연결 설정에 필요한 정보와 함께 데이터도 보내버리기 때문이다. 이론 상으로는 한 번 연결된 정보를 캐싱해놓고 있다가 다음 연결 때 캐시를 불러와 0 RTT만으로도 통신을 시작할 수 있다고는 하나 현실적으로 어려움이 있다고 한다.

QIOC 내에 아예 TLS 인증서를 내포하고 있기 때문에, 최초의 연결 설정에서 필요한 인증 정보와 데이터를 함께 전송한다. 그래서 클라이언트가 서버에 어떤 신호를 주고, 서버도 거기에 응답하기만 하면 바로 본 통신을 시작할 수 있다는 것이다.

---

## HTTPS

![Untitled](02%20HTTP,%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC,%20%E1%84%87%E1%85%A9%E1%84%8B%E1%85%A1%E1%86%AB%20810e9edea62a4e37b58a00f169b29a3b/Untitled%206.png)

보안이 강화된 HTTP로 Hypertext Transfer Protocol Over Secure Socket Layer의 약자이다. 모든 HTTP 요청과 응답 데이터는 네트워크로 보내지기 전에 암호화된다. HTTPS는 HTTP의 하부에 SSL과 같은 보안계층을 제공함으로써 동작한다.

## SSL (Secure Socket Layer) & TLS (Tranport Layer Security)

사실 두 개념은 동일하다. SSL 3.0 이후 버전을 계승한 것이 TLS이다. (기존 버전들과 구분하기 위해)

SSL의 암호화 방식에는 대칭키 암호화 방식과 공개키(비대칭키) 암호화 방식이 있다. 

### 대칭키 암호화

- 인코딩과 디코딩에 클라이언트와 서버가 같은 키를 사용하는 알고리즘
- 같은 키를 사용하기 때문에 대칭키를 전달하는 과정에서 탈취되는 등의 위험이 있다.
- 공개키 방식에 비해 복호화 속도가 빠르다는 장점이 있다.

### 공개키 암호화

- 인코딩과 디코딩에 다른 키를 사용하는 알고리즘
- A키로 인코딩화면 B키로만 디코딩을 할 수 있다.
- 인코딩 키는 `공개`되어 있으며 보통 디지털 인증서 안에 포함되어 있다.
- 디코딩 키는 호스트만이 `비밀키`로 가지고 있다.

> 인증서
> 
> - 인증서의 내용은 CA (Certificate Authority - 인증 기관) 의 비공개 키를 이용해서 암호화 되어 웹브라우저에게 제공된다.
>     - 서비스 정보 (인증서 발급자, CA의 디지털 서명, 서비스 도메인)
>     - 서버측 공개키

### **SSL 인증서의 서비스 보증방법**

- 웹브라우저가 서버에 접속하면 서버는 제일 먼저 인증서를 제공한다.
- 브라우저는 인증서를 발급한 CA가 자신이 갖고있는 CA 리스트에 있는지 확인한다.
- 리스트에 있다면 해당 CA의 공개키를 이용해서 인증서를 복호화 한다.
- 인증서를 복호화 할 수 있다는 것은 이 인증서가 CA의 비공개키에 의해서 암호화 된 것을 의미한다. 즉 데이터를 제공한 사람의 신원을 보장해주게 되는 것이다.

### **SSL 동작방법**

- 공개키 암호 방식은 알고리즘 계산방식이 느린 경향이 있다.
- 따라서 SSL은 암호화된 데이터를 전송하기 위해서 `공개키와 대칭키 암호화 방식을 혼합`하여 사용한다.
- 안전한 의사소통 채널을 수립할 때는 공개키 암호를 사용하고, 이렇게 만들어진 안전한 채널을 통해서 임시의 무작위 대칭키를 생성 및 교환한다. 해당 대칭키는 나머지 데이터 암호화에 활용한다.
    - 실제 데이터 암호화 방식 : 대칭키
    - 상기 대칭키를 서로 공유하기 위한 암호화 방식 : 공개키

### SSL 동작 과정

SSL은 크게 핸드 셰이크, 세션, 세션 종료 세 단계로 이루어진다.

![Untitled](02%20HTTP,%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC,%20%E1%84%87%E1%85%A9%E1%84%8B%E1%85%A1%E1%86%AB%20810e9edea62a4e37b58a00f169b29a3b/Untitled%207.png)

1. 클라이언트는 서버에게 Client Hello를 건낸다. `이때 랜덤한 데이터와 현재 지원할 수 있는 암호화 방식을 서버에게 전달한다.` 암호화 방식을 전달하는 이유는 같은 대칭키, 공개키 기법이라도 다양한 암호화 방식이 있기 때문이다.
2. 클라이언트의 인사를 받는 서버도 똑같이 인사를 보낸다. 이때 서버는 세 가지 내용을 전달한다. 앞서 클라이언트가 전달한 내용과 `동일한 랜덤 데이터와 지원 가능한 암호화 방식, 그리고 인증서`이이다.
3. 인증서를 받은 클라이언트는 이 인증서가 CA가 발급한 인증서 목록에 존재하는지 확인하고, 존재한다면 CA 에서 공유하는 공개키를 가지고 인증서를 복호환다.
4. 본격적으로 키를 주고 받기 위해 클라이언트에서 앞선 통신 1, 2 과정에서 주고 받은 랜덤 데이터를 조합하여 `임시 대칭키`를 만든다. 해당 대칭키 역시 노출되면 안되기 때문에 인증서에 들어있던 서버측 공개키로 암호화하여 서버에게 전달한다.
5. 클라이언트로부터 전달받은 임시 대칭키를 다시 서버의 비밀키로 복호화하여 세션키 (대칭키) 를 생성한다.
6. 본격적으로 클라이언트와 서버가 세션키를 통해 지속적으로 통신을 한다.
7. 모든 과정이 완료되면 세션을 종료하고 세션키를 삭제하여 마무리한다.

---

## CORS (Cross Origin Resource Sharing, 교차 출처 리소스 공유)

![Untitled](02%20HTTP,%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC,%20%E1%84%87%E1%85%A9%E1%84%8B%E1%85%A1%E1%86%AB%20810e9edea62a4e37b58a00f169b29a3b/Untitled%208.png)

서로 다른 출처에 대해서 접근을 차단하는 것을 말하낟.

### 동일 출처 정책이 필요한 이유

사실 출처가 다른 두 애플리케이션이 자유로이 소통할 수 있는 환경은 꽤 위험하다. 제약이 없다면 해커가 XSS나 CSRF 등의 방법을 이용해서 개인 정보를 가로챌 수 있기 때문이다.

### 출처 비교와 차단은 브라우저가 한다.

출처를 비교하는 로직은 서버에 구현된 스펙이 아닌 `브라우저에 구현된 스펙`이다. 따라서 브라우저를 통하지 않는다면 CORS 정책에 걸리지 않게 된다.

![Untitled](02%20HTTP,%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC,%20%E1%84%87%E1%85%A9%E1%84%8B%E1%85%A1%E1%86%AB%20810e9edea62a4e37b58a00f169b29a3b/Untitled%209.png)

![Untitled](02%20HTTP,%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC,%20%E1%84%87%E1%85%A9%E1%84%8B%E1%85%A1%E1%86%AB%20810e9edea62a4e37b58a00f169b29a3b/Untitled%2010.png)

## XSS (Cross - Site Scripting)

취약성이 있는 웹사이트를 방문한 사용자의 브라우저에서 부정한 HTML 태그나 JavaScript 등을 동작시키는 공격을 말한다. 주로 다른 웹 사이트와 정보를 교환하는 식으로 작동하므로 사이트 간 스크립팅이라고 명칭한다.

이 취약점은 `웹 애플리케이션이 입력 받은 값을 제대로 검사하지 않고 사용할 경우 나타나며`, 공격에 성공하면 사이트에 접속한 사용자는 삽입된 코드를 실행하게 된다. 

보통 의도치 않은 행동을 수행시키거나 쿠키 & 세션 등의 민감한 정보를 탈취한다.

### Stored XSS와 Reflected XSS

![Untitled](02%20HTTP,%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC,%20%E1%84%87%E1%85%A9%E1%84%8B%E1%85%A1%E1%86%AB%20810e9edea62a4e37b58a00f169b29a3b/Untitled%2011.png)

## CSRF (Cross-Site Request Foregery, 크로스 사이트 리퀘스트 포저리)

사용자가 자신의 의지와는 무관하게 공격자가 의도한 행위를 특정 웹사이트에 요청하게 하는 공격을 말한다.

`XSS가 사용자가 특정 웹 사이트를 신용하는 점을 노린 것이라면, CSRF는 특정 웹 사이트가 사용자의 웹 브라우저를 신용하는 상태를 노린 것이다.`
일단 사용자가 웹사이트에 로그인한 상태에서 사이트간 요청 위조 공격 코드가 삽입된 페이지를 열면, 공격 대상이 되는 웹사이트는 위조된 공격 명령이 믿을 수 있는 사용자로부터 발송된 것으로 판단하게 되어 공격에 노출된다.

CSRF는 공격자가 사용자의 컴퓨터를 감염시키거나 사이트 해킹을 해서 이뤄지는 공격이 아니므로 공격이 성공하려면 다음 조건을 만족해야 한다.

- 위조 요청을 전송하는 서비스에 희생자가 로그인 상태여야 함.
- 희생자는 공격자가 만든 피싱 사이트에 접속해야 함.

> CSRF Token을 사용하거나 Back-end 단에서 Request의 Referrer를 확인하여 검증할 수 있다.
> 

![Untitled](02%20HTTP,%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC,%20%E1%84%87%E1%85%A9%E1%84%8B%E1%85%A1%E1%86%AB%20810e9edea62a4e37b58a00f169b29a3b/Untitled%2012.png)

---

## SQL Injection

SQL 인젝션이란 웹 애플리케이션을 이용하고 있는 데이터베이스에 SQL을 부정하게 실행하는 공격이다. 커다란 위협을 일으킬 수 있는 취약성으로 개인 정보나 기밀 정보 누설로 직결되기도 한다.

ex) `SELECT * FROM users WHERE [users.id](http://users.id) = 123` 정상 쿼리

 `SELECT * FROM users WHERE [users.id](http://users.id) = 123 or 1 = 1`  공격 쿼리 (항상 참이 되는 값을 넣는 등 전체 조회를 해올 수도 있음)

### 🟩쿠키

쿠키는 웹 브라우저에서 로컬에 작은 텍스트 데이터를 저장하는 영역이라고 할 수 있습니다. 따라서 별도로 서버에 쿠키 정보를 저장하지 않고 사용할 수 있습니다. 쿠키는 클라이언트와 서버 간의 상태를 유지하고 추적하기 위해 사용을 하는데, 보통 `Name`과 `Value`로 이루어져 있고, 추가적인 정보를 설정할 수도 있습니다.

보통 쿠키가 언제까지 유효한지 설정되어 있지 않으면, 브라우저가 닫힐 때 쿠키도 함께 사라집니다. 이런 쿠키를 `세션 쿠키`라고 부릅니다.

```
response.addHeader("Set-Cookie", "ID=" + userId);
```

위와 같이 Header에 `Set-Cookie` 값을 넣어주어 사용할 수 있습니다.

![https://blog.kakaocdn.net/dn/bSxs6E/btsyn01OSy4/lQyZFdIDe90JtdK6sbi3y1/img.png](https://blog.kakaocdn.net/dn/bSxs6E/btsyn01OSy4/lQyZFdIDe90JtdK6sbi3y1/img.png)

응답으로는 다음과 같이 Response Header 영역에 담겨오게 됩니다.

**쿠키의 단점**

1. JS로 쿠키에 접근이 가능하다 -> JS 코드로 `document.cookie`를 입력하면 현재 쿠키 값을 확인할 수 있습니다.
    - 이는 XSS (Cross Site Scripting) 공격에 취약합니다. 대표적으로는 게시판 제목 또는 이미 src에 해커사이트 주소로 연결되게 작성하여 쿠키를 탈취합니다.
2. 하나의 브라우저에 저장되기 때문에 다른 브라우저로 요청시 접근이 불가능합니다.
3. 헤더에 담기는 내용이기 때문에 조작이 가능합니다.
4. 쿠키의 용량은 4KB로 제한되어 있어 많은 정보를 담을 수 없습니다.

```
Set-Cookie: 쿠키명=쿠키값; path=/; HttpOnly
```

사실 JS로 쿠키에 접근이 가능하다는 점은 위와 같이 `HttpOnly` 설정을 (가장 마지막에 접미사만 추가하면 됨) 통해 접근을 차단하여 XSS 공격을 막을 수 있습니다. 해당 설정을 적용하면 브라우저에서 해당 쿠키로 접근할 수 없게 되지만, 쿠키에 포함된 정보의 대부분이 브라우저에서 접근할 필요가 없기 때문에 `Http Only Cookie`는 기본적으로 적용하는 것이 좋습니다.

### 🟩세션

세션은 쿠키와 다르게 서버에서 세션 정보를 저장하여 관리합니다. Spring Boot를 통해 만든 프로젝트라면 보통 내장 톰캣의 메모리에 세션 정보가 저장되게 됩니다. 서버에서 직접 관리하기 때문에 요청이 많아도 네트워크에 걸리는 부하가 적다는 장점이 있습니다.

클라이언트가 서버에 로그인하면 서버는 해당 사용자에 대한 세션을 생성하고 세션 저장소에 저장한 뒤 Response Header에 담아 반환합니다. 이후 로그인 유지를 위해서 사용자는 요청에 세션을 담아 서버로 보내며, 서버는 사용자에게 받은 세션값과 세션 저장소와 비교하여 일치하는 경우 로그인을 유지하게 합니다.

**세션의 단점**

1. 서버 측에서 상태를 유지하게 되기 때문에 많은 수의 세션 데이터가 저장되는 경우 서버에 부하가 증가합니다.
    - 이를 해소하기 위해 `Sticky Session`, `Session Clustering` 과 같은 방안이 존재하지만, 이를 사용하기 위한 추가적인 비용이 발생합니다.
        - Sticky Session의 경우 처음 지정받은 서버만 사용할 수 있기 때문에 해당 서버가 터지거나 과부하가 걸리면 대응할 수 없다고 합니다.
        - Session Clustering은 모든 서버마다 세션을 복사해줘야 하기 때문에 상당한 메모리를 요구한다고 합니다.
2. 서버 환경이 다중 서버로 구성되어있다면 로그인 유지가 되지 않을 수 있습니다.

2번의 경우는 세션 정보가 없는 서버에 요청을 보내게 되는 경우 발생하는 문제인데 별도의 세션 저장소를 두고 다수의 서버가 세션 저장소를 바라보게 한다면 해결할 수 있겠습니다.

### 🟩JWT (Json Web Token)

JWT는 세션이나 일반적인 토큰과는 다르게 자체적으로 값이 암호화 되어있으며, JWT를 서버에서 복호화 후 인증하기 때문에 기본적으로 DB를 사용하지 않으면서 보다 안전하게 사용할 수 있다는 장점이 있습니다. 쿠키와 세션에도 유효기간이 존재하지만, 토큰의 경우 그 유효기간이 상대적으로 훨씬 짧기 때문에 탈취되더라도 보안 공백이 그리 길지 않습니다. (리프레시 토큰의 경우는 그렇지 못한데 이는 다음에 설명하도록 하겠습니다)

```java
response.setHeader("Authorization", "Bearer " + token);
```

일반적으로 위와 같이 Header의 name 값으로 `Authorization` 을 전달해주며 `인증타입 token`의 형태를 value로 전달해줍니다. Oauth와 JWT를 사용하는 경우 일반적으로 인증타입을 Bearer로 지정합니다.

**JWT 구조**

JWT는 `Hader, Payload, Signature`으로 나눌 수 있으며 각각 `.` 을 통하여 분류합니다.

- Header : 토큰의 암호화 종류(alg)와 타입(typ)이 정의된 Json을 Base64로 인코딩 합니다

```java
// 인코딩 전
{
	"alg" : "HS256",
	"typ" : "JWT"
}

// Base64 인코딩 후// eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
```

- Paylaod : 해당 토큰이 포함할 몇 가지 필드 (Key : Value)가 담긴 Json을 Base64로 인코딩 합니다.

```java
// 인코딩 전
{
	"iat" : 1554076800000,
	"exp" : 1554681600000,
	"uuid" : "09226aac-5f8c-11e9-8647-d663bd873d93"
}

// Base64 인코딩 후// eyJpYXQiOjE1NTQwNzY4MDAwMDAsImV4cCI6MTU1NDY4MTYwMDAwMCwidXVpZCI6IjA5MjI2YWFjLTVmOGMtMTFlOS04NjQ3LWQ2NjNiZDg3M2Q5MyJ9
```

일반적으로 JWT에서 payload에 담는 key-value 데이터를 `claim`이라고 합니다. payload에 어떤 데이터를 포함할 것인지는 개발자와 개발 환경에 따라 다르지만, JWT 문서 대부분에서 `발급시간(iat)`과 `만료시간(exp)`은 포함하기를 권장하고 있습니다.

- Signature : 위변조를 검증하기 위해 사용하며, 검증은 발급자쪽에서만 가능합니다. Header와 Payload의 encode 값 + 서버에 별도로 저장되어 있는 Secret Key를 기반으로 암호화됩니다. `Header와 Payload의 encode 값을 포함하여 암호화되기 때문에 Header 혹은 Payload의 위변조를 파악할 수 있습니다.`

```java
var encodedHeader = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9";
var encodedPayload = "eyJpYXQiOjE1NTQwNzY4MDAwMDAsImV4cCI6MTU1NDY4MTYwMDAwMCwidXVpZCI6IjA5MjI2YWFjLTVmOGMtMTFlOS04NjQ3LWQ2NjNiZDg3M2Q5MyJ9";

var secret = "서버에서 지정하는 비밀키";
var signature = HS256.encode(encodedHeader + "." + encodedPayload, secret).toBase64String();

// signature// 0mgNdgQZNw_C2QfloguymnakqIpDizuvo0uFRuxDWQo
```

**JWT의 단점**

1. 쿠키와 세션 방식에 비해 Header에 담아줘야 하는 정보의 양이 많아 네트워크에 부하가 생길 수 있습니다. → 네트워크의 성능이 계속 좋아지고 있기 때문에 큰 단점이 아닌 것 같음
2. 별도의 DB를 사용하지 않고 서버 메모리 상에서 검증을 통해 인증이 이루어지기 때문에 사실상 탈취에 대해서는 무력하다고 볼 수 있습니다. → 리프레시 토큰을 사용하면 어느정도 보안성을 높일 수 있지만 마찬가지로 탈취에 대해서는 머리 아픔..

### Ref

[https://inpa.tistory.com/entry/WEB-📚-CORS-💯-정리-해결-방법-👏](https://inpa.tistory.com/entry/WEB-%F0%9F%93%9A-CORS-%F0%9F%92%AF-%EC%A0%95%EB%A6%AC-%ED%95%B4%EA%B2%B0-%EB%B0%A9%EB%B2%95-%F0%9F%91%8F)

[https://inpa.tistory.com/entry/WEB-🌐-HTTP-30-통신-기술-이제는-확실히-이해하자](https://inpa.tistory.com/entry/WEB-%F0%9F%8C%90-HTTP-30-%ED%86%B5%EC%8B%A0-%EA%B8%B0%EC%88%A0-%EC%9D%B4%EC%A0%9C%EB%8A%94-%ED%99%95%EC%8B%A4%ED%9E%88-%EC%9D%B4%ED%95%B4%ED%95%98%EC%9E%90)

[https://velog.io/@wonizizi99/SpringSpring-security-CSRF란-disable](https://velog.io/@wonizizi99/SpringSpring-security-CSRF%EB%9E%80-disable) 

[https://github.com/dongkyun-dev/TIL/blob/master/web/HTTP1.1과 HTTP2.0%2C 그리고 간단한 HTTP3.0.md](https://github.com/dongkyun-dev/TIL/blob/master/web/HTTP1.1%EA%B3%BC%20HTTP2.0%2C%20%EA%B7%B8%EB%A6%AC%EA%B3%A0%20%EA%B0%84%EB%8B%A8%ED%95%9C%20HTTP3.0.md) 

[https://brunch.co.kr/@swimjiy/47](https://brunch.co.kr/@swimjiy/47)