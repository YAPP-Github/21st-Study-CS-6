# RPC

### Remote Procedure Call

직역 : 원격 프로시저 호출

 `RPC`를 통해 다른 서버에 존재하는 함수나 프로시저를 호출하고 응답을 받을 수 있다. 

> 다른 컴퓨터에 존재하는 프로그램의 프로시저를 실행할 수 있도록 허용하는 프로토콜

## 동작 방식

![operating-system-remote-procedure-call-1](/Users/wontaeyeon/Downloads/operating-system-remote-procedure-call-1.png)

`Caller`에서 `procedure`를 호출하면, `Request Message`를 통해 함수의 `parameter`가 전달 되고, `Callee`에서 수행한 결과를 `Reply message`를 통해 `Caller`에게 전달 된다.

이 때, 주고 받는 함수에 대한 명세(parameter, method, .. )에 대해 알아야 하기 때문에 인터페이스를 `IDL(Interface Definition Langauge)`을 활용하여 정의하여야 한다.

프로그램이 작성된 언어는 모두 다르기 때문에, RPC는 특정 언어에 대한 종속되어있지 않다.
Client는 `Java`, Server는 `Go`로 작성되어 있다면, 이들 간의 원활한 소통을 위해 언어 차이를 극복해야한다.
 => `IDL`이라는 명세가 필요 (`Xml`, `Json`, `Proto`)



![operating-system-remote-call-procedure-working](/Users/wontaeyeon/Downloads/operating-system-remote-call-procedure-working.png)

>### `Stub`
>
>서버와 클라이언트는 다른 주소 공간을 사용하고 있는데, 함수에 사용하는 매개변수를 변환하는 작업이 필요
>
>특정 Machine에 종속된 매개변수를 `표준 포맷`으로 변환하는 역할을 한다.
>
>- Stub Compiler가 IDL 파일을 읽어 원하는 `Language`로 생성
>- Parameter 객체를 Message 형태로 `Marshaling/Unmarshalling` 하는 레이어
>
>`Marshaling`은 데이터를 바이트로 쪼개서 TCP/IP 같은 통신 채널을 통해 전송될 수 있는 형태로 바꿔주는 과정
>
>`UnMarshaling`은 반대로 전송받은 바이트를 원래의 형태로 복원하는 과정

## Why RPC

RPC 가 다른 프로세스, 서버와 연결해서 통신하는 방식이라면,  떠오르는 방식 중에 하나가 `HTTP` 기반의 `Rest API` 방식이 있다. `client-server`  모델 통신은 워낙 익숙하다.

실제로 `Rest API` 방식이 확산되기 전에는 `RPC` 를 통해 통신하는 방식이 많았다고 한다.

하지만, 최근에는 다시 RPC 를 이용한 방식들이 많이 등장하고 있다. 

최근에는 하나의 어플리케이션을 개발하고자 할때 모놀리틱 방식에서 MSA(Micro Service Architecture) 방식으로 많이 넘어가고 있다. 

하나의 서버에 DB, 백엔드, 모니터링 등등 다양한 기능들을 모두 합쳐서 개발하지 않고, 유연된 확장을 위해 각각의 서버마다 역할을 분담하여 `DB`, `백엔드`, `모니터링` 등등을 나누어서 개발을 한다. 이 때, 각각의 컴포넌트들을 네트워크를 통해 연결이 되어야 한다.

이러한 MSA 구조에서 각 컴포넌트 끼리 수 많은 통신이 이루어져야 하는데, `Rest API` 를 통해 통신한다는 것은 속도 측면에서 많은 부담이 된다.

 `Json`을 통해 `Payload`를 저장하는 `Rest Api`는 사람이 읽기 직관적이고 편리하지만 컴퓨터에서 동작하기에 인코딩이라는 비용이 발생하게 된다. 또, 엄격한 규약이 존재하지 않다는 특징이 있다.

이와 달리 `RPC`는 `IDL`이라는 규약이 반드시 필요하고, `Protobuf`라는 이진 메세지 형식을 통해 `Payload`를 전달하기 때문에 속도 측면에서 큰 이점을 가질 수 있다.

그래서 MSA에서 컴포넌트들 간의 통신에 채택되는 경우가 많이 있다.

## gRPC

- Google 에서 만든 오픈소스 RPC Framework

![download (3)](/Users/wontaeyeon/Downloads/download (3).jpeg)

`Service`간의 통신이 `Prote Request/Resonse`를 통ㅐ 이루어짐. 각각의 `Service`의 PL이 달라도 `gRPC Stub`를 통해 통신이 가능하다.

`gRPC`는 `IDL`을 `Protocol Buffer`를 통해 정의함.

```protobuf
service School { 
    rpc getPerson (int) returns (Person); 
}


message Person {
    string name = 1; 
    int32 id = 2;
}
```

ex) `School`이라는 `service`에 `getPerson()`이라는 함수가 존재. 인터페이스에 명시

입력 혹은 반환 타입이 구조체인 경우 `message` 형식으로 그 자료들을 묶어서 전달하거나 받을 수 있다고 한다.

more: [gRPC protocol buffers](https://developers.google.com/protocol-buffers/docs/overview)

### gRPC 의 장점

- 통신 속도가 빠르다 - `gRPC` 는 `Json` 데이터 포맷이 아닌 `프로토콜 버퍼` 기반의 메시지를 바이너리 형식으로 직렬화 하여 통신하기 때문에 보다 통신 속도 빠르다.
- 엄격한 타입 검사 - 프로토콜 버퍼로 원하는 메시지 타입을 미리 정의를 해 놓고 통신하기 때문에 런타임 에러 발생률이 줄어듦
- 이중 스트리밍 - `gRPC` 가 `HTTP/2` 를 사용하기 때문에, 단순 요청-응답 방식의 통신을 뛰어 넘어 스트리밍 방식이 지원 가능함