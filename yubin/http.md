# http 0.9, 1.0, 1.1, 2.0, QUIC

<img src="https://user-images.githubusercontent.com/69676101/209159073-29984be9-a7ad-40cc-908d-b32ad92fd68b.png"  width="400">   



### HTTP/0.9

- 가능한 메서드 : GET
- Header X
- 오직 HTML 파일만 전송 가능
- 상태 확인을 위한 코드가 존재하지 않았음

### HTTP/1.0

- 헤더 추가
- 버전관리
    - 사용된 버전을 요청 라인에 추가하여 명시적으로 알려주게끔 함
- 상태코드
    - HTTP 응답에 상태코드가 포함되어 수신자가 요청 처리 상태 (성공 혹은 실패)를 확인할 수 있음
- content-type
    - http header덕분에, html이아닌 다른 문서 유형 전송 가능
- POST, HEAD 메소드 추가

### HTTP/1.1

- 호스트 헤더
    - 1.1버전부터 사양에 따라 호스트헤더를 요구
- persistent connection   
<img src="https://user-images.githubusercontent.com/69676101/209159267-9171108b-7de1-4f99-9942-2df464fbc0ff.png"  width="400">
    - http 1.0에서는 각 요청/응답 쌍은 매번 새 연결을 열어야 했음
    - 1.1부터는 단일 연결에서 지정한 timeout동안 여러 요청을 실행함

        ```java
        HTTP/1.1 200 OK
        Connection: Keep-Alive
        Content-Encoding: gzip
        Content-Type: text/html; charset=utf-8
        Date: Thu, 11 Aug 2016 15:23:13 GMT
        Keep-Alive: timeout=5
        Last-Modified: Mon, 25 Jul 2016 04:32:39 GMT
        Server: Apache
        
        (body)
        ```

    - 따라서 기존 연결에 대해 handshake 생략이 가능함
- 파이프라이닝 추가
    - 이전 요청의 응답이 전송되기 전에 다음 전송을 가능케 함 → latency를 낮춤
    - 아쉬운점
        - 1.1의 파이프라이닝 방식은 ‘한번에 순차적인 여러 요청을 연속적으로 하고 그 **순서에 맞춰** 응답을 받는 방식’
          <img src="https://user-images.githubusercontent.com/69676101/209159603-e7a36a7e-3249-4d69-8565-b28f631e6082.png"  width="400">

        - 먼저 받은 요청이 끝나지 않으면 그 뒤의 요청의 처리가 아무리 빨리 끝나도 먼저 온 요청이 끝날때까지 기다려야 하는 HOL(head of line) blocking 문제가 발생
- continue status
    - 서버가 처리할 수 없는 요청을 거부하는 것을 방지하기 위해 클라이언트 측에서 먼저 요청 헤더만 보내고 상태코드 100을 수신하는지 확인할 수 있음
- 새로운 메소드 추가
    - PUT, PATCH, DELETE, CONNECT, TRACE, OPTIONS등 메소드 추가

### HTTP/2.0

- http 메시지 전송 방식의 변화   
  <img src="https://user-images.githubusercontent.com/69676101/209159777-b865753f-3dcf-44a2-af2d-47ccdd564826.png"  width="400">   

    - 바이너리 프레이밍 계층 사용
        - 줄 바꿈으로 구분된 일반 텍스트 HTTP/1.x 프로토콜과 달리 모든 HTTP/2 통신은 더 작은 메시지와 프레임으로 분할되며 각각은 이진 형식으로 인코딩됨
        - 파싱, 전송 속도 향상
        - 오류 발생 가능성 줄어듦   
          <img src="https://user-images.githubusercontent.com/69676101/209159906-32d8e66a-4d7c-4d63-9bfb-59c902c7b20c.png"  width="400">

        - 요청과 응답의 다중화가 가능해짐
            - HOLB(head of line blocking) 해결
            - 프레임으로 쪼개졌기 때문에 메시지 간의 순서가 사라짐
            - 프레임의 interleaving(끼어들기)이 가능해짐
                - 먼저 도착한게 조립되면서, 요청과 응답의 순서가 의미가 없어짐
            - 즉, HTTP 메시지를 독립적인 프레임으로 분해하고 인터리브한 다음 다른 쪽 끝에서 재조립하게 됨   
<img src="https://user-images.githubusercontent.com/69676101/209159948-bd48d565-62c2-47b1-a88b-620880f86029.png"  width="400">

- multiplexed streams (위와 같은 결의 내용)
    - 한 커넥션에서 동시에 여러 메시지를 주고받고, 응답은 순서에 상관없이 stream으로 주고받음
    - http 1.1 파이프라이닝과 달리 각 스트림은 서로 독립적이며 순서대로 전송하거나 수신할 필요가 없음
    - http 1.1의 파이프라이닝과 멀티플렉싱의 가장 큰 차이는 순차적 응답 처리에 따른 HOLB를 완벽하게 해결함
- stream prioritization
    - 리소스간 우선순위를 정함
        - 스트림에 우선순위 가중치를 정해줌
        - A 리소스가 B 리소스에 의존적일때, B리소스의 우선순위를 A리소스보다 높게하여 렌더링이 늦어지지 않게 함
- server push
    - 클라이언트가 요청하지 않은 리소스를 서버측에서 보낼 수 있게됨
        - 기존엔 html을 모두 읽고, html을 해석하면서 필요한 리소스를 재요청하는 방식이었지만
        - http 2.0에서는 서버가 html에 포함된 리소스를 클라이언트에게 push해주는 방식을 사용함
            - 클라이언트의 요청을 최소화하여 성능향상
        - 서버가 전송한 리소스에 대해서는 클라이언트는 이제 요청하지 않음(필요x)
- header compression   
  <img src="https://user-images.githubusercontent.com/69676101/209160008-9e9d34a0-300d-4932-94fc-7fa79f17f333.png"  width="400">

    - 헤더의 중복을 압축
    - static table, dynamic table을 이용하여 중복을 검출함
        - 중복된 것은 index만 뽑고, 중복되지 않은 데이터는 허프만 인코딩을 통해 압축함
- 아쉬운점
    - 어쩔수 없이 TCP기반이기 때문에 발생하는 HOLB 문제   
      <img src="https://user-images.githubusercontent.com/69676101/209160031-77ab82f7-3260-4fbd-b714-892658dc0eab.png"  width="400">

        - 스트림이 병렬로 처리될 때 2번 스트림이 유실되면
        - 서버에서 2번 스트림을 다시 찾아올 때까지 나머지가 다 기다리고 있어야함
        - 초록색 스트림들은 빨간색과 상관이 없음에도 불구
        - QUIC의 경우, 독립된 스트림을 구성하여 향상된 멀티플렉싱이 가능해짐
            - 빨간색 스트림에 문제가 생겨도 나머지 초록,하늘 스트림은 문제 없음

### 1.1, 2, 2 with push를 애니메이션으로 설명해주는 사이트

[HTTP 1.1 vs HTTP.2 vs HTTP/2 with Push - Manning](https://freecontent.manning.com/animation-http-1-1-vs-http-2-vs-http-2-with-push/)

### QUIC

- 전송계층 프로토콜 (TCP, UDP같은 것들)
- 구글이 만듦
- UDP
    - 왜?
        - TCP는 신뢰성을 위해 지연을 희생함
        - 데이터 전송에만 집중한 설계   
          <img src="https://user-images.githubusercontent.com/69676101/209160069-19d4a115-4d19-4821-af37-d642644d5703.png"  width="400">

        - 별도의 기능이 없으므로 원하는 기능 구현이 가능
- connection UUID로 서버와 연결
    - 모바일의 경우 많은 이점
    - 스마트폰의 경우 wifi→데이터 등 한 네트워크에서 다른 네트워크로의 전환이 많음
- TLS 기본 적용

# http Q/A

[Tech-Interview/03-NETWORK.md at main · VSFe/Tech-Interview](https://github.com/VSFe/Tech-Interview/blob/main/03-NETWORK.md)

- 401 (Unauthorized) 와 403 (Forbidden)은 의미적으로 어떤 차이가 있나요?
- 200 (ok) 와 201 (created) 의 차이에 대해 설명해 주세요.
- HTTP Method의 멱등성에 대해 설명해 주세요.
- GET과 POST의 차이는 무엇인가요?
- POST와 PUT, PATCH의 차이는 무엇인가요?
- HTTP 1.1 이후로, GET에도 Body에 데이터를 실을 수 있게 되었습니다. 그럼에도 불구하고 왜 아직도 이런 방식을 지양하는 것일까요?
- 공개키와 대칭키에 대해 설명해 주세요.
- 왜 HTTPS Handshake 과정에서는 인증서를 사용하는 것 일까요?
- SSL과 TLS의 차이는 무엇인가요?
- HTTP/1.1과 HTTP/2의 차이점은 무엇인가요?
- HOL Blocking 에 대해 설명해 주세요.
- HTTP/3.0의 주요 특징에 대해 설명해 주세요.
- 왜 HTTP는 TCP를 사용하나요?
- 그렇다면, 왜 HTTP/3 에서는 UDP(QUIC) 를 사용하나요? 위에서 언급한 UDP의 문제가 해결되었나요?
- Stateless와 Connectionless에 대해 설명해 주세요.
- 왜 HTTP는 Stateless 구조를 채택하고 있을까요?
- HTTP Persistence Connection 이 무엇인가요?
- TCP의 keep-alive와 HTTP의 keep-alive의 차이는 무엇인가요?