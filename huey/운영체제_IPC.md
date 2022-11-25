<!-- - title: 프로세스간 통신 IPC  -->
<!-- - author: 장현희  -->
<!-- - date: 2022-11-25 12:00:00 +0800 -->

# 들어가며

### 시간표 웹사이트

- 서버 없이 custom 시간표 웹사이트를 만들고 싶었음
- 일정 이름, 시작일시, 종료일시 입력하면 시간표에 저장됨
- 저장된 일정은 시간표라는 TimeTable에 렌더링

### 문제 상황

- 페이지를 나가고 다시 접속하면 데이터가 전부 날라감
- 서버가 없기 때문에 데이터를 저장할 수 없었음

### 선택했던 구현 방법

- 저장 요청 시 : 로컬 환경에 txt 파일로 데이터를 저장
- 페이지 로드 시 : 로컬에 저장된 파일 데이터를 자동으로 불러와 렌더링

### 결론

- 자동으로 로컬에서 파일을 불러오는건 불가능
- 직접 파일을 업로드해서 렌더링은 가능
- So, 웹 페이지에 접속해서, 저장했던 시간표 데이터 파일을 업로드 하면 렌더링
- ~~(사실 로컬 스토리지나 쿠키를 쓰면 된다)~~

### why?

> 당시 개발 잘하는 친구한테 물어봤을 때의 답변
> - 보안 때문에 그렇다.
> - 웹 브라우저에서 JS 코드가 파일에 접근하게 된다면 악의적인 공격이 너무 쉬워짐

# 프로그램과 프로세스

우리가 인터넷을 사용하는 크롬, 사파리, 엣지 등은 브라우저라는 프로그램이며, 이를 실행시키면 프로세스가 된다.

이 때 프로세스는 각각 Code, Data, Stack, Heap의 구조로 되어있는 독립된 메모리 영역을 할당 받는다.

각 프로세스는 별도의 주소 공간에서 실행되며, 서로 독자적인 메모리 공간을 갖기 때문에 서로 메모리 공간을 공유할 수 없다. (Protection)

즉, 다른 프로세스의 변수나 자료구조에 접근할 수 없다.

> ### 위에서 불가능 했던 이유
>
> 프로세스들은 각각 독자적인 메모리 공간을 가지며, 다른 프로세스의 변수에 마음대로 접근할 수 없다.\
> -> 브라우저 프로세스에서 파일 시스템 프로세스에 접근해서 데이터를 마음대로 가져올 수 없다.

**but, 만약 다른 프로세스에 접근하고 싶다면?**


# IPC

프로세스 간 통신(Inter-Process Communication, IPC)이란 프로세스들 사이에 서로 데이터를 주고받는 행위 또는 그에 대한 방법이나 경로를 뜻한다.

프로세스는 원래 독립적이지만, 상황에 따라 프로세스끼리 협력해야 되는 경우가 발생한다. 이럴 때 프로세스간 자원과 데이터를 공유할 수 있어야 하는데, 서로간의 통신을 위해 별도의 매커니즘이 필요하고, IPC를 이용해 프로세스간 통신을 할 수 있게 된다.

**IPC 가 필요한 이유 (프로세스 간 통신이 필요한 이유)**

✔ 정보 공유(Information Sharing): 
여러 사용자가 동일한 정보에 엑세스 할 필요가 있을 수 있다.

✔ 편의성(Convenience):
다수의 사용자가 동시에 여러가지 작업을 수행할 수 있다.

✔ 가속화(Computation Speed-up): 
특정 작업(task)을 여러 개의 서브 작업(sub-task)로 쪼개어 
프로세스의 병렬성을 키움으로써 처리 속도를 높일 수 있다. 
이 때 메인 작업과 서브 작업은 서로 통신의 필요성이 생긴다.

✔ 모듈화(Modularity):
특정한 시스템 기능을 별도의 프로세스(스레드)로 구분하여 모듈화된 시스템을 구성할 수 있다.
이 때 모듈간의 통신이 필요하다

### 멀티 프로세싱 (기타 설명)

> 가속화 모듈화에 대한 설명입니다.

하나의 응용 프로그램을 여러개의 프로세스로 구성한다.

각 프로세스가 하나의 작업을 처리하도록 한다

MSA와 비슷한 개념(?)

**크롬 예시**

- 크롬은 대표적인 멀티 프로세싱 프로그램
- 탭 하나를 키고 프로세스를 살펴보았다.

![chrome process image 1](https://user-images.githubusercontent.com/77145383/203822996-312aeb26-6253-4a5c-ae52-cf6bc41dd93b.png)

![chrome process image 2](https://user-images.githubusercontent.com/77145383/203823301-8a6fdcd1-c556-485f-a501-c4798ed4303b.png)

- PID : 프로세스 고유 아이디
- PPID : 부모 프로세스의 아이디

![chromium image](https://www.chromium.org/developers/design-documents/plugin-architecture/pluginsoutofprocess.png)
_Image Reference: https://www.chromium.org/developers/design-documents/plugin-architecture/_


> 크롬 멀티프로세싱 관련 refs
> - [크롬 동작 원리](https://developer.chrome.com/blog/inside-browser-part2/)
> - [https://chanto11.tistory.com/70](https://chanto11.tistory.com/70)
> - [https://d2.naver.com/helloworld/2922312](https://d2.naver.com/helloworld/2922312)


# IPC 종류

### 파이프 (Pipe)

특징

- 단방향 통신
- 양방향 통신을 위해선 파이프를 총 2개 만들어야 함
- byte Stream 으로 데이터를 주고 받음 = 메시지 간 경계 X

종류

- 지명 파이프 (Named Pipe)
  - 부모 프로세스 -> 자식 프로세스 (같은 PPID 가짐)
- 익명 파이프 (Anonymous Pipe)
  - 서로 다른 프로세스 통신
  - 파이프의 이름만 알면 통신이 가능

### 메세지 큐 (Message Queue)

- 양방향 통신
- 메시지 패싱(Message Passing)이라는 IPC 방법론의 구현 방법 중 하나
- 메모리를 사용한 파이프
- 구조체를 기반으로 통신
- 커널에 링크드 리스트(queue) 형태로 저장 후 FIFO

### 공유 메모리 (Shared Memory)

- [커널](https://ko.wikipedia.org/wiki/커널_(컴퓨팅)) 공간에 메모리 공간을 만들고, 해당 공간을 변수처럼 쓴다.
- 전역 변수와 비슷한 개념
- 메모리 주소를 키값으로 접근

### 소켓 (Socket)

- IPC 기법을 위해 존재하는 건 아님
- 네트워크 소켓통신을 사용함
- client-server 구조로 통신

etc...


# 리눅스 파이프 메소드 '|'

nohup 등을 이용해 백그라운드로 실행되고 있는 서버 pid 찾기 

`ps -ef | grep .jar`

-> `|` 파이프 명령어
-> "ps의 출력을 grep의 입력으로!" 라는 뜻

- ps(Process Status) : 현재 실행중인 프로세스 목록을 보여주는 명령어
- grep : 특정 문자열 찾는 명령어


# References

- [https://www.chromium.org/chromium-projects/](https://www.chromium.org/chromium-projects/)
- [https://doitnow-man.tistory.com/110](https://doitnow-man.tistory.com/110)
