# `Index`

![](https://velog.velcdn.com/images/roycewon/post/a079c8a1-aa85-46b2-8e62-e7ca7bc8f335/image.png)
> *Index는 DB 분야에 있어서 테이블에 대한 동작의 속도를 높여주는 자료 구조를 일컫는다. Index는 테이블 내 1개의 컬럼, 혹은 여러 개의 컬럼을 이용하여 생성될 수 있다. 고속의 검색 동작뿐만 아니라 레코드 접근과 관련 효율적인 순서 매김 동작에 대한 기초를 제공한다.* [위키피디아](https://ko.wikipedia.org/wiki/%EC%9D%B8%EB%8D%B1%EC%8A%A4_(%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%B2%A0%EC%9D%B4%EC%8A%A4))


`Index`라는 자료구조는 데이터와 그 데이터의 주소값(pointer)이 키, 값으로 되어있으며, 쿼리가 들어오면 이 index를 바탕으로 주소값을 통해 검색한다.
- `[책의 목차와 같은 역할]`


Index 데이터들은 정렬된 형태를 갖는다. 기존엔 Where문으로 특정 조건의 데이터를 찾기 위해서 테이블의 전체를 조건과 비교해야 하는 '풀 테이블 스캔(Full Table Scan)' 작업이 필요했는데, 인덱스를 이용하면 데이터들이 정렬되어 있기 때문에 조건에 맞는 데이터를 빠르게 찾을 수 있다. 

또 `ORDER BY` 문이나 `MIN/MAX` 같은 경우도 이미 정렬이 되어 있기 때문에 빠르게 수행할 수 있다는 특징이 있다.

## `Index`의 자료구조
Index는 여러 자료구조를 이용해서 구현되어 있는데, 대표적인 자료구조로 해시 테이블과 B+Tree가 있다.

### Hash Table (해시 테이블)
![](https://velog.velcdn.com/images/roycewon/post/119726d7-2537-4081-a5ab-57b2b95f74f6/image.png)

해시 테이블은 Key와 Value를 한 쌍으로 데이터를 저장하는 자료구조이다. (Key, Value)로 쌍을 표현하며, Key값을 이용해 대응되는 Value값을 구하는 방식이다.

Key 값으로 Value가 저장되어 있는 주소를 바로 산출할 수 있기 때문에, 해시 테이블의 평균적인 시간 복잡도는 $O(1)$이지만, 해시 충돌이라는 단점이 존재한다. 그리고 해시 함수는 Key가 조금이라도 다르면 완전히 다른 해시 값을 생성한다는 특징이 있기에 등호(=) 연산에는 효율이 좋지만, 부등호 연산(>, <)은 부적합 하다는 특징이 있다. 또, 해시 테이블은 내부 데이터들이 정렬되어 있지 않아 탐색이 효율적이지 않기 때문에 실제 Index 테이블에 사용되진 않는다고 한다.

### B-Tree 
![](https://velog.velcdn.com/images/roycewon/post/9de4e7ac-19d6-4387-8c98-156145d97ec4/image.png)

`B-Tree`란 자식 노드가 2개 이상인 트리를 의미한다. 이진검색 트리처럼 각 Key의 왼쪽 자식은 항상 Key보다 작은 값을, 오른쪽 자식은 큰 값을 가진다.

`Hash Table`과 달리 `B-Tree` 기반의 DB Index는 구조는 Key-Value 값들아 항상 Key를 기준으로 오름차순 정렬되어 있기 때문에 이로 인해 부등호 연산(>, <)에 대해 보다 효율적인 데이터 탐색을 기대할 수 있다. 
`B-Tree`는 평균 시간 복잡도는 $O(logN)$.

### B+Tree

`B-Tree`는 `write` 작업에 의해 균형이 깨지면, 성능이 악화되고, 순차 검색의 경우 트리를 순회하여야 하여 많은 노드를 확인해야 하는 단점이 있다.
실제 DB엔진에서는 이러한 단점을 보완한 B+Tree 자료구조를 사용하고 있다고 한다.
![](https://velog.velcdn.com/images/roycewon/post/1cad26e9-357c-4f0f-a390-679e5b81955f/image.jpg)

`B+Tree`는 `B-Tree`를 확장 및 개선한 자료 구조로서, 리프 노드에만 데이터의 위치를 저장한다.

리프 노드들끼리는 `LinkedList` 구조로 서로를 참조하고 있는데, 이러한 구조는 순차 검색 연산을 하는 경우, 리프 노드를 저장한 `LinkedList`를 탐색하면 되는 이점을 가진다.


또한 중간 브랜치 노드에 Value가 없어서 `B-Tree`보다 메모리를 덜 차지하는 만큼, 노드의 메모리에 더 많은 Key를 저장할 수 있다는 장점도 존재한다.


## `Index` 주의 사항
### `Index`의 단점
1. `Index`의 빠른 검색 기능과 동반하여 발생하는 가장 큰 문제점은 정렬된 상태를 계속 유지 시키기 때문에 레코드 내에 데이터값이 바뀌는 부분에 대해선 성능이 악화될 수 있다.
`INSERT`, `UPDATE`, `DELETE`를 통해 데이터가 추가되거나 값이 바뀐다면 INDEX 테이블 내에 있는 값들을 다시 정렬이 필요하다.

2. `Index`의 검색 역시 항상 성능 향상을 불러오진 않는다.
데이터가 적은 경우(수천건 미만) 인 경우에는 Index가 존재하는 것 보단 `Index` 없이 `Full scan` 하는 것이 성능이 더 좋다고 한다.

3. `Index`를 관리하기 위해서는 데이터베이스의 약 10%에 해당하는 저장공간이 추가로 필요하다.


### `Index` 고려 사항

위와 같은 단점들이 있기에, 무분별한 `Indexing`은 성능 악화를 초래 할 수 있다. `Index`의 특징을 고려해보면 다음과 같은 컬럼(데이터)에 대해 활용하는것이 효율적이다.

> 1. 조건절에 자주 등장하는 컬럼
3. 중복되는 데이터가 최소한인 컬럼 (Cardinality가 높은) 컬럼
4. ORDER BY 절에서 자주 사용되는 컬럼
5. 조인 조건으로 자주 사용되는 컬럼
\+ 규모가 작지 않은 테이블


### 참고 자료
- [DB Index 입문](https://tecoble.techcourse.co.kr/post/2021-09-18-db-index/)
- [[DB] 데이터베이스(DB) 인덱스(Index) 란 무엇인가?](https://choicode.tistory.com/27)
- [[DB] 데이터베이스 인덱스(Index) 란 무엇인가?](https://coding-factory.tistory.com/746)
- [[자료구조] 그림으로 알아보는 B+Tree](https://velog.io/@emplam27/%EC%9E%90%EB%A3%8C%EA%B5%AC%EC%A1%B0-%EA%B7%B8%EB%A6%BC%EC%9C%BC%EB%A1%9C-%EC%95%8C%EC%95%84%EB%B3%B4%EB%8A%94-B-Plus-Tree)


- [쉬운코드 - DB Index (유튜브 강의)](https://www.youtube.com/watch?v=IMDH4iAQ6zM)