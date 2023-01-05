<!-- - title: HTTPS  -->
<!-- - author: 장현희  -->
<!-- - date: 2023-01-04 12:00:00 +0800 -->

# HTTPS란

### 웹 프로토콜 HTTP

HTTP는 웹상에서 TCP 기반으로 정보를 주고받을 수 있는 웹 프로토콜입니다. 하지만 실제로 HTTP로 웹 통신을 하고 있을까요?

실제로 HTTP로 웹 페이지를 배포하고 해당 웹 사이트에 접속을 해보면 아래처럼 꽤 무서운 문구들이 표시됩니다.

![http_not_secured](https://user-images.githubusercontent.com/77145383/210742217-28710cf3-8db1-417f-a11c-51f74c137113.png)

이는 HTTPS를 사용하지 않아서 발생한 경고 메시지인데요. 그럼 HTTPS란 뭘까요?

### HTTPS

아래는 HTTPS에 대한 위키백과의 정의입니다.

> HTTPS(HyperText Transfer Protocol over Secure Socket Layer, HTTP over TLS, HTTP over SSL, HTTP Secure)는 월드 와이드 웹 통신 프로토콜인 HTTP의 보안이 강화된 버전이다.
> HTTPS는 통신의 인증과 암호화를 위해 넷스케이프 커뮤니케이션즈 코퍼레이션이 개발한 넷스케이프 웹 프로토콜이며, 전자 상거래에서 널리 쓰인다.
>
> HTTPS는 소켓 통신에서 일반 텍스트를 이용하는 대신에, SSL이나 TLS 프로토콜을 통해 세션 데이터를 암호화한다. 따라서 데이터의 적절한 보호를 보장한다. HTTPS의 기본 TCP/IP 포트는 443이다.
>
> 보호의 수준은 웹 브라우저에서의 구현 정확도와 서버 소프트웨어, 지원하는 암호화 알고리즘에 달려있다.
>
> HTTPS를 사용하는 웹페이지의 URI는 'http://'대신 'https://'로 시작한다.
>
> _출처 : [위키피디아](https://ko.wikipedia.org/wiki/HTTPS)_
> {: .text-right}

간단하게 얘기하면 HTTP에서 보안이 강화된 통신 방법이라고 볼 수 있습니다.

그럼 어떻게 HTTPS가 어떻게 HTTP보다 안전하게 통신을 할 수 있게 되었는지 알아봅시다.


# HTTPS 통신 과정

## 오브젝트

- 인증기관 (CA)
- 서버
- 클라이언트

## 간략한 통신 과정

### 서버 SSL 인증서 발행

1. 서버는 한 쌍의 공개키와 개인키를 생성하고, 사이트의 정보와 자신의 공개키를 인증기관에 전달하여 SSL 인증서를 요청합니다.
2. 인증기관은 SSL 인증서를 만들고, 자신의 개인키로 해당 인증서를 암호화하여 서버에게 전달합니다. 해당 인증서에는 서비스의 정보(인증서를 발급한 CA, 서비스의 도메인 등)와 서버의 공개키가 포함되어 있습니다.
3. 서버는 전달받은 암호화된 SSL을 그대로 게시합니다.

### 클라이언트 접속

1. 클라이언트는 암호화 되어 있는 서버의 인증서를 인증기관의 공개키로 복호화하고 서버의 정보를 확인합니다. 서버의 정보를 확인할 수 있다면 해당 인증서는 실제 인증서입니다. 그리고 복호화를 통해 서버의 공개키 역시 알게 되었습니다.
2. 클라이언트는 이제 서버를 신뢰하고, 해당 서버의 공개키로 대칭키인 세션키를 암호화하여 전송합니다.
3. 서버는 자신의 개인키로 암호화되어 있는 클라이언트의 요청을 복호화하고 대칭키인 세션키를 저장합니다.
4. 이후 대칭키 암호화 방식을 이용해 통신합니다.

## 클라이언트 접속의 상세한 통신 과정

클라이언트 접속은 실제로 SSL Handshake 과정을 거칩니다.

### 1. Client Hello

- 클라이언트 -> 서버
- 클라이언트가 서버에 연결을 시도하면서 헨드셰이크를 시작하는 패킷
- 내용
    - 지원하는 TLS(SSL) 버전
    - 랜덤 바이트 문자열
    - 지원하는 암호 제품군 목록 (Cipher Suite) : SSL Protocol Version, Hash 방식 등의 정보가 담겨 있음

![https_cipher_suite](https://user-images.githubusercontent.com/77145383/210742285-ebcee3fc-2d6a-4089-9f02-f687cebf8310.png)
_Image Reference : https://run-it.tistory.com/30_


### 2. Server Hello

- 서버 -> 클라이언트
- Client Hello 메시지에 대한 응답으로 서버가 보내주는 패킷
- 내용
    - 랜덤 바이트 문자열
    - 선택한 암호 제품군 (Cipher Suite) : Client Hello에서 보내준 목록 중 하나를 선택함

### 3. Certificate

- 서버 -> 클라이언트
- Server Hello 패킷과 같이 자신의 인증서를 보내주는 패킷
- 내용
    - 인증기관의 개인키로 암호화 되어 있는 SSL 인증서
    - SSL Protocol Version

### 4. Server Key Exchange

- 서버 -> 클라이언트
- 만약 SSL 안에 서버의 공개키가 없다면, 직접 공개키를 보내준다.
- 내용
    - 서버의 공개키

### 5. Verify server certificate

- 클라이언트가 서버의 SSL 인증서를 인증서 발행 기관(CA)의 공개키로 복호화하고 이를 검증합니다.
- 이를 통해 서버가 인증서에 명시된 서버인지, 클라이언트가 상호작용 중인 서버가 실제 해당 도메인의 소유자인지를 확인하고, 해당 서버의 공개키까지 갖게 되었습니다.

### 6. Client Key Exchange

- 클라이언트 -> 서버
- 클라이언트는 대칭키를 생성하여 서버의 공개키로 암호화하여 전송합니다.
- 해당 대칭키를 통해 실제로 암호화 통신을 진행합니다.
- 내용
    - 암호화된 대칭키

### 7. Finished

- 서버 -> 클라이언트
- 해당 패킷을 통해 SSL HandShake를 종료합니다.
- 이후 서버와 클라이언트가 갖게 된 대칭키를 통해 암호화된 통신을 주고받게 됩니다.


# HTTP와 HTTPS 성능

## 일반적으로

일반적으로 HTTP는 단순성 때문에 HTTPS보다 빠릅니다.

당연하게도 HTTPS에는 SSL Hand Shake가 추가로 필요하기 때문이라고 볼 수 있죠.

하지만 구글이 2012년 SPDY라는 새로운 네트워킹 프로토콜을 발표하고, 이를 통해 HTTP/2로 발전되면서 상황이 바뀌었습니다.

다양한 웹 브라우저가 HTTP/2 지원을 추가하면서 암호화된 프로토콜에서만 HTTP/2가 작동할 수 있도록 구현했다고 합니다.

또한 HTTP/2는 HTTPS를 위해 만들어진 것처럼 보이기도 합니다.\
HTTP/1.1의 plaintext 기반의 프로토콜은 HTTP/2에서 binary 기반의 프로토콜로 변경되었고, 한번 연결로 여러 번 통신하는 방식은 HTTP보다 HTTPS에서 더 효과적이기 때문입니다.

## 테스트 결과

그러나 아래 링크를 보면 HTTP보다 HTTPS가 더 빠르다고 합니다.

[HTTP vs HTTPS 속도 테스트](https://www.httpvshttps.com)

왜 이런 결과가 나왔을까요?

앞서 말했듯이 모든 브라우저에서 HTTP/2는 SSL/TLS에서만 지원한다고 했습니다.

즉, 이 테스트는 실제로는 HTTP vs HTTPS가 아닌 HTTP/1.1 vs HTTP/2 로 볼 수 있습니다. 결과는 당연히 성능적으로 많은 부분이 개선된 HTTP/2의 압승이 되어버렸죠.

## 실제로는

실제로 요즘의 CPU는 HTTPS 프로토콜이 개발되던 때와는 비교가 안될 정도로 빨라졌습니다.

따라서 오늘날의 두 프로토콜의 성능은 사실상 거의 차이가 없다고 봐도 무방합니다. -> HTTPS는 안 쓸 이유가 없다.


# References

[https://aws-hyoh.tistory.com/39](https://aws-hyoh.tistory.com/39)\
[https://ko.wikipedia.org/wiki/HTTPS](https://ko.wikipedia.org/wiki/HTTPS)\
[https://okky.kr/articles/480266](https://okky.kr/articles/480266)\
[https://ko.myservername.com/http-vs-https-an-depth-comparison-features](https://ko.myservername.com/http-vs-https-an-depth-comparison-features))
