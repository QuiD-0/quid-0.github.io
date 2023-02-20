# 인덱스 자료구조

## 후보군&#x20;

### HashMap

* 단건 검색 속도 O(1)
* 범위 탐색은 O(N)
* 전방 일치 탐색 불가  ex) like "AB%"

### List

* 정렬되지 않은 리스트의 탐색은 O(N)
* 정렬된 리스트의 탐색을 O(log N)
* 정렬되지 않은 리스트의 정렬 시간 복잡도는 O(N) \~ O(N log N)
* 삽입 / 삭제 비용이 매우 높음

### Tree

* 트리 높이에 따라 시간 복잡도가 결정됨 (높이를 최소화하는 것이 중요)
* 한쪽으로 노드가 치우치지 않도록 균형을 잡아주는 트리 사용
* ex) Red-Black Tree, B+ Tree



## B+ Tree

* 삽입 / 삭제시 항상 균형을 이룬다.
* 하나의 노드가 여러개의 자식노드를 가질 수 있다.
* 리프노드에만 데이터 존재 -> 연속적인 데이터 접근시 유리

<figure><img src="https://s3.ap-south-1.amazonaws.com/s3.studytonight.com/tutorials/uploads/pictures/1608306929-76844.png" alt=""><figcaption></figcaption></figure>



> 인덱스를 이용하면 읽기 성능은 좋아지지만 삽입 / 삭제 코스트가 증가한다.\
> 모든것은 트레이트 오프.
