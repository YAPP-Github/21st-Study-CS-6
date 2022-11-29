# CPU Scheduling



## 배경

- `Process`는 `CPU`를 할당받아 Task를 수행하는데, 하나의 `Process`만 처리할 수 있다.

  

  > `Process`는 다음과 같은 `state`들을 가질 수 있다.![download (2)](/Users/wontaeyeon/Downloads/download (2).jpeg)
  >
  > 
  >
  > 이때, `CPU`가 처리중이던 `Process` 가 `I/O`를 기다리며 `waiting state`가 되었다면
  > 비싸고 아주 빠른 `CPU `는 사용되지 못하게 되는 엄청난 비효율성이 존재한다.
  >
  > 
  >
  > **CPU의 효율적인 사용을 위한 조치가 필요함**
  > **=> cpu scheduling**



## Scheduler

`Process` 상태를 제어하는 방법에 따라 `Scheduler`를 세분화해서 나누기도 함.

- #### Long Term or job scheduler : `Process`를 `ready state`로 가져오는 역할을 함
  `ready state`인 `Process`의 수를 제어 => 멀티 프로그래밍의 지표
   시간이 많이 걸리는 `I/O-bound`와 짧지만 CPU자원을 사용해야 하는 `CPU-bound` 간의 밸런스를 유지하여 효율성을 높힘

  

- #### Medium-term scheduler : `Process`를 일시 중단하고 다시 시작하는 역할을 담당.
  프로세스 혼합을 개선하기 위해 또는 메모리 요구 사항의 변경으로 인해 사용 가능한 메모리가 과도하게 커밋되어 메모리를 비워야 하는 경우 동작함. 멀티 프로그래밍의 정도를 줄여줌.

  

- #### **Short term or CPU scheduler** :  `running state` 로 가져올 `ready state`의 `Process`를 선택함.

  CPU 접근 권한을 주는건 `Dispatcher`가 담당한다. (이때, **context switching**이 발생함)



### CPU Scheduler

- CPU의 효율성(항시 이용)을 높이기 위해 할당 받을 `Process`를 관리하는 작업

  > *작업 순서와 시간을 할당하는 것으로, `Process`들이 자원을 사용하는 순서를 결정하는 일*

  -> *어떻게? 어떤 작업을 우선으로?*

### Scheduling 고려 사항

- CPU utilization : CPU 이용률

- **Throughput** : 처리량
  단위 시간당 얼마나 많은 프로세스가 실행되는 지

- Turnaround time : 총 처리 시간
  어떤 task 의 처음부터 끝을 의미함

- Waiting time : 대기 시간
  `Process`가 ready queue 에 있는 시간

- Response time : 응답 시간
  Process 실행 후 <u>첫 응답이 오는데</u> 걸리는 시간

  /+ starvation, fairness, dead line

### 선점형(Preemptive)

`Process`가 CPU를 점유하고 있을 때 다른 `Process`가 CPU를 빼앗아 차지할 수 있는 방법
-> *대화식 시분할 시스템과 같은 실시간 시스템에서 사용됨*

>#### 1. RR 스케줄링 (Round Robin)
>
>- 각 프로세스에 차례로 일정한 시간 할당량(Time slice) 동안 차지하도록 하는 기법.
>
>#### 2. SRT 스케줄링 (Shortest Remaining Time)
>
>- 남은 처리 시간이 가장 짧은 프로세스에게 CPU를 할당해 작업을 처리하도록 하는 방법.
>
>#### 3. MFQ 스케줄링 (Multilevel Feedback Queue)
>
>- 작업을 여러 단계로 나누어 처리하는 방식. 높은 단계는 시간 할당량을 적게, 낮은 단계는 많게 할당해 주는 방법

### 비선점형(Nonpreemptive)

`Process`의 작업이 끝날 때 까지 CPU를 독점하는 방법. 
응답 시간의 예측이 용이하지만, 작업을 모두 기다려야 하는 단점이 존재

>#### 1. FIFO 스케줄링 (First In First Out) = FCFS
>
>먼저 들어온 것을 우선처리(선입 선출)하는 방식으로, 가장 간단한 방식입니다.
>
>#### 2.HRN 스케줄링 (Highest Resopnse ratio Next)
>
>우선순위 = (대기시간 + 작업시간 / 작업시간) 으로 계산하여 우선순위에 따라 처리하는 방식
>
>#### 3. SJF 스케줄링 (Shortest Job First) 
>
>작업시간이 가장 짧은 것부터 먼저 처리하는 방식

![Different types of CPU Scheduling Algorithms](https://media.geeksforgeeks.org/wp-content/uploads/20220525174157/UntitledDiagram12.jpg)

cf) 다양한 스케쥴링 알고리즘 비교

- https://www.geeksforgeeks.org/cpu-scheduling-in-operating-systems/#comparison



## Linux Scheduling

- Linux 2.6.23 커널부터는 CFS, (Completely Fair Schduler)가 Linux 시스템의 디폴트 스케줄링 알고리즘

  CFS라는 용어에서 알 수 있듯이 런큐에서 실행 대기 상태로 기다리는 프로세스를 공정하게 실행하도록 기회를 부여하는 스케줄러

  ****

  - 타임 슬라이스

    **프로세스에게 부여한 실행 시간**. `Process`는 주어진 타임 슬라이스를 모두 소진하면 컨텍스트 스위칭된다. 

  - 우선순위

    우선순위가 높은 프로세스를 1. 가장 먼저 CPU에서 실행시키고 2. 더 많은 타임 슬라이스를 할당한다.

    리눅스는 총 140단계의 우선순위를 제공한다고 한다.

  - 가상 실행 시간(vruntime)
    커널에서 프로세스가 그동안 실행한 시간을 정규화한 시간 정보. 가중치 중 하나

- 간단하게 정리하자면,
  `Process`는 (타임 슬라이스 x 우선순위로 선정된 load_weight) 만큼의 CPU에 할당될 수 있고,
  vruntime에 따라 다음 실행할 `Process`를 선정하는 방식 (vruntime이 작다면, 실행이 된 시간이 적다는 뜻. 이때 Red-Black 트리를 사용함)

=> CFS의 목표는 런큐에 있는 실행 대기 상태의 프로세스에게 CPU 시간을 우선순위에 따라 공정하게 할당하는 것

cf)  [Linux CFS](https://cesl.tistory.com/entry/Linux-CFS-%EC%8A%A4%EC%BC%80%EC%A4%84%EB%A7%81-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%EC%A0%95%EB%A6%AC), [CFS 스케쥴러](https://jamcorp6733.tistory.com/201), https://wpaud16.tistory.com/260

>  **스케줄러 클래스**
>
> 리눅스 스케줄러는 모듈화돼 있어 여러 유형의 프로세스를 각기 다른 알고리즘을 통해 스케줄링할 수 있다.
> 모듈화된 형태를 스케줄러 클래스라고 한다.
>
> - 교체 가능한 여러 알고리즘을 동시에 사용하면서 클래스별로 독자적인 방식으로 프로세스를 스케줄링할 수 있다.
> - 완전 공정 스케줄러 CFS는 `SCHED_NORMAL( posix 표준에서 SCHED_OTHER)`로 정의된 리눅스의 일반 프로세스용 스케줄러다.



