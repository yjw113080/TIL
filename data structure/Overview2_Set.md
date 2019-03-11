### Binary Tree
1. 이진트리의 개념
* 트리는 선형과는 달리 위계가 있는 데이터 구조다. 이진트리는 한 노드가 최대 두 개의 child를 갖는 경우.
* 이진트리의 노드는 1) 데이터 2) 좌측 child 포인터 3) 우측 child 포인터로 이루어져있다.
* 트리가 비어있는 경우, 루트의 값은 NULL 이다.

2. 트리 순회 방향:
   1) 깊이 기준 순회: Inorder(좌-뿌리-우), Preorder(뿌리-좌-우), Postorder(좌-우-뿌리)
   2) 넓이 기준 순회: 층위 단위로 순회

3. 이진 트리의 속성
* x 층에 존재할 수 있는 최대의 노드 개수: 2^(x-1).
* 전체 이진 트리에 존재할 수 있는 최대의 노드 개수: 2^h – 1.
* Minimum possible height =  (Log2(n+1))의 올림값<br/>
  O(log(n))의 의미: 초반엔 급격히 상승하지만 개수 (n) 이 많아질 수록 증가 폭이 줄어든다
* 이진 트리에서 left node(자식이 없는 노드) 개수는 항상 자식을 둘 가진 node 개수 보다 1 많다.
* Time Complexity of Tree Traversal: O(n)

4. 트리/이진트리를 사용하는 예시
위계가 있는 파일 시스템, JavaScript 의 DOM.



### Binary Search Tree
1. 이진검색트리의 개념
* 이진트리 기준으로 몇 가지 추가적인 속성이 더해짐.
1) 좌측 하위 트리는 노드의 키보다 작은 키를 갖고 있다.
2) 우측 하위 트리는 노드의 키보다 큰 키를 갖고 있다.
3) 좌/우측 하위 트리는 각각 위 조건을 만족시킨다.

2. 이진검색트리의 속성
* Search :  O(h)
* Insertion : O(h)
* Deletion : O(h)
* Extra Space : O(n) for pointers
* 왼쪽 하위 트리와 오른쪽 하위 트리의 높이 차이가 1 을 넘지 않는 Tree를 Height Balanced Tree 라고 하는데, Height balanced 되었을 경우 h = O(Log n) 
* Self-Balancing 트리: Node를 삽입/삭제할 때 Height balanced 상태를 자동으로 유지(높이를 가능한한 작게)해주는 Tree를 말하고, 
구현 방법으로는 AVL, 레드 블랙 트리 같은 게 있습니다. 일반 BST는 Worst case(한 노드에 하나의 노드만 달려있음)의 경우에 O(n) 으로 시간 복잡도가 높아지지만, 자가 균형 BST의 경우  O(Log n) 으로 복잡도를 줄일 수 있다.


3. 이진검색트리 사용 예시
* 값 접근/검색은 LinkedList 보다는 빠르고 Array 보다 느리다.
* 값 삽입/삭제는 Array 보다는 빠르고 LinkedList 보다 느리다.
* 데이터가 계속 들어오고 나가는 검색 시스템, 정렬되어 출력되어야 하는 데이터. 예를 들면 쇼핑몰 사이트에서 새 상품이 추가되고, 품절되고, 모든 상품이 정렬되어 출력되는 부분.



### Binary Heap
1. 이진 힙 개념
* 이진트리 기준으로 몇 가지 추가적인 속성이 더해짐.
1) 완전 트리. 가장 마지막 층 빼고 각 층이 모두 채워져있음. 마지막 레벨에 있는 모든 child node들은 가능한한 왼쪽 (노드) 에 위치해야 함.
2) Min Heap이거나 Max Heap이다. Min Binary Heap의 경우, 루트의 키는 전체 트리의 키 중 제일 작다. Max Heap은 그 반대의 경우.

2. 이진 힙 속성
* Min Heap에서 최솟값 찾기(Get): O(1) 
* Min Heap 최솟값 추출(Extract): O(Log n) 
* Min Heap에서 키 감소시키기: O(Log n) 
* Insert: O(Log n) 
* Delete: O(Log n)

3. 이진 힙 사용 예시
효율성을 추구하는 우선순위 기반 큐에서 사용됨. 예를 들어 OS의 프로세스 스케줄링. Dijstra’s and Prim’s graph algorithms 에서도 사용됨. 일반적인 검색에서는 잘 사용되지 않음.



### Hashing
1. 관련 개념
* HashingHash Function: 주어진 인풋 키를 실제 사용할 integer 값으로 바꾸어 주는 함수. 매핑된 정수 값은 hash table 에서 인덱스로 사용된다. 
* 좋은 Hash Function 은 1) 계산이 효율적이고(Efficiently Computable) 2) 균일하게 키를 배분한다.
* Hash Table: 주어진 정보를 가리키는 포인터를 담는 배열. 
* Collision Handling: bigint나 string 이 Hash Function을 통해 int로 변형되는 과정에서 같은 값으로 잘못 변형될 수도 있다. 이건 절대 발생해서는 안되는 케이스이기 때문에 만약 발생했다면 이를 적절히 해결해야 한다.

![](https://files.slack.com/files-pri/T9Y35UY03-FFG27F4SF/hash-table-chaining-1.png)<br/>
* 이걸 해결하기 위한 방법으로는 1) Chaining, 2) Open Addressing이 있다. Chaining 은 동일한 키가 있는 경우 기존 데이터의 linked list 뒤에 새로운 데이터를 삽입하는 방식으로 충돌을 해결.
Open Addressing 은 모든 요소가 해시 테이블에 저장된다. 각 테이블 내용값은 레코드를 갖거나 NIL 값을 갖는다. 요소를 검색할 때 원하는 값이 나올 때까지 테이블 칸을 하나하나 뒤진다. 

2. Hashing의 속성
* Space : O(n)
* Search    : O(1) [Average]    O(n) [Worst case]
* Insertion : O(1) [Average]    O(n) [Worst Case]
* Deletion  : O(1) [Average]    O(n) [Worst Case]
이진검색트리보다 조금 더 나아보이지만, 이진검색트리는 요소들이 정렬되어 있는 반면 해싱 내에서는 정렬되어 있지 않다. 이진검색트리 보다 구현도 어려운 것이 흠. 

3. Hashing 의 사용 예시
* 요소 set의 중복을 제거하는 데에 사용할 수 있다. 모든 항목의 frequency 체크에도 사용한다.
* 브라우저에서 방문 기록이 hashing을 이용한다.
* 방화벽에서 스팸을 걸러낼 때, IP 주소를 hash할 때에도 사용함.

