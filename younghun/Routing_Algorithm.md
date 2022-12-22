## 1. Router의 기능
![](https://velog.velcdn.com/images/0_hun/post/2da40aad-95a1-496a-bf8d-7f84e21033ac/image.png)

**Router의 기능**은 크게 **Forwarding**과 **Routing**으로 나누어 집니다.
Forwarding은 **router의 input에서부터 output까지 패킷을 옮겨주는 역할**을 하며 Routing은 **출발지부터 목적지까지 패킷이 전송되는 route를 결정**하는 역할을 합니다.

또한 forwarding 역할을 하는 router의 부분을 **Data plane**, routing 역할을 하는 부분을 **Control plane** 이라고 부릅니다.

## 2. Routing Algorithm의 종류
그렇다면 routing 할 땐 어떤 알고리즘을 통해 route를 결정할까요?
다음과 같은 2가지 종류의 알고리즘 유형이 있습니다.

### Link State Algorithm
- **Complete topology**(**global information**)를 알고 있을 때 사용하는 알고리즘. 
(즉, 라우터와 연결된 전체 네트워크 정보를 알고 있을 때 사용)
- Route의 정확도가 높다.
- 전체 네트워크 topology 정보를 얻기 위한 오버헤드가 존재한다.
- 대표적인 알고리즘 : **Dijkstra's algorithm**

### Distance Vector Algorithm
- **Partial topology**(**decentralized information**)를 알고 있을 때 사용하는 알고리즘. 
(즉, 라우터와 연결된 일부 네트워크 정보만 알고 있을 때 사용)
- Route의 정확도가 상대적으로 떨어진다. (최적의 경로가 아닐 수 도 있다.)
- 오버헤드가 적어 상대적으로 빠르다.
- 대표적인 알고리즘 : **Bellman-Ford algorithm**

## 3. Dijkstra's algorithm
![](https://velog.velcdn.com/images/0_hun/post/b81ac7cb-87ab-4fc3-a4e3-04f62bfa8b28/image.png)

위 그림은 다익스트라 알고리즘을 통해 최적의 경로를 구하는 과정을 Step별로 나눈 모습입니다. 네트워크 topology가 주어지고 해당 정보들로 알고리즘을 실행하는 것을 볼 수 있습니다. 

먼저 그림 나온 값들의 의미를 먼저 파악해보겠습니다.
- **N'** : u 노드에서 출발 했을 때 최적의 경로가 결정된 노드들의 집합
- **D(v)** : u 노드에서 출발 했을 때 v 노드까지의 거리
- **p(v)** : v 노드에 도착하기 바로 전 노드

각 Step들에 대하여 알고리즘은 다음과 같은 방식으로 동작합니다.

**Step 0** : u 노드에 출발하여 도달 할 수 있는 노드들에 대하여 
D(n), p(n) 값들을 초기화 합니다. 그중 거리값이 가장 작은 노드 x를 선택합니다.

**Step 1~N** : 아직 N'에 포함되지 않는 노드들을 대상으로 
원래 값과 노드 x를 거쳐서 가는 거리 중 더 작은 값으로 초기화하고 
그 중 거리값이 가장 작은 노드 y를 선택합니다.

이러한 방식을 반복해서 u에서 출발할 때의 각 노드들에 대한 최적의 경로들을 구한 뒤 routing table에 기록하여 패킷들을 routing 해줍니다.

## 4. Bellman-Ford algorithm
![](https://velog.velcdn.com/images/0_hun/post/65cd4130-0846-4a9a-a5f6-401278fbca28/image.png)

위 그림은 벨만-포드 알고리즘을 통해 최적의 경로를 구하는 예시입니다.
네트워크 topology 그림이 해당 자료에 있지만 
실제로는 **dv(z)**, **dx(z)**, **dw(z)** 라는 일부 정보만 주어진 상황을 가정하고 있습니다.

그럼 이번에도 먼저 그림 나온 값들의 의미를 먼저 파악해보겠습니다.
- **dv(z)** : v 노드에서 출발 했을 때 z 노드까지의 **최소거리**
- **c(u, v)** : u 노드에서 v 노드까지의 거리

벨만-포드 알고리즘은 다이나믹 프로그래밍과 유사합니다.
u 노드에서 z 노드까지의 최적 경로를 구하고 싶은데 
정보가 부족하니 주어진 정보를 활용해서 **du(z)**를 구하자는 방식입니다.

예를 들어, dv(z) = 5, dx(z) = 3, dw(z) = 3 이라는 값을 알고 있고 
u 노드에서 주변 연결된 정보만 알고 있으면 다음과 같은 식을 통해 du(z)를 구할 수 있습니다.
```
du(z) = min { c(u,v) + dv(z), c(u,x) + dx(z), c(u,x) + dw(z) }
```

<br>

### 실제 동작 방식
![](https://velog.velcdn.com/images/0_hun/post/adcad54c-6614-46b7-abd8-5b735be6966f/image.png)

실제로는 초기엔 자기 노드의 라우팅 테이블 정보만 알고 있기 때문에
**자신과 물리적으로 연결된 cost 밖에 알지 못한다.**

따라서 각 노드들은 패킷들을 주고 받으면서 **cost에 대한 정보를 공유**하여 
**벨만-포드 알고리즘을 통해** 자기 라우팅 테이블의 정보들을 **갱신**한다.


Reference : 
https://ddongwon.tistory.com/95
Computer Networking: A Top Down Approach 7th Edition, Global Edition Jim Kurose, Keith RossPearsonApril 2016.

