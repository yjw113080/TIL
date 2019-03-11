
### Graph
1. 개념
- 유한한 노드의 집합
- edge 라고 불리는 형태 (u,v)로 정렬된 노드 쌍의 집합.
- (u,v)와 (v,u)는 각각 다른 값을 가리킨다.
- edge 는 weight/value/cost 를 가질 수 있다.

2. 그래프 분류 방법
2.1. Direction <br/><br/>
![](https://www.geeksforgeeks.org/wp-content/uploads/Tree_overview_of_data_structures_1.jpg)
* Undirected Graph : 모든 edge가 양방향
* Directed Graph : 모든 edge 가 단방향

2.2. Weight
![](https://www.google.co.id/url?sa=i&source=images&cd=&cad=rja&uact=8&ved=2ahUKEwjTmfeWivTfAhXFbX0KHXp7CeAQjRx6BAgBEAU&url=https%3A%2F%2Fstackoverflow.com%2Fquestions%2F20807286%2Fminimum-sum-weight-of-connecting-3-vertices-in-an-undirected-weighted-graph-wi&psig=AOvVaw3BPVZmtarw6hEi_P9tem-a&ust=1547789179652860)
* Weighted Graph : 각 edge에 대한 frequency (==weight)가 표기됨. 
* Unweighted Graph : 표기되지 않음.

3. Time Complexity <br/>
3.1. 인접 Matrix :
* Traversal :(By BFS or DFS) O(V^2)
* Space : O(V^2)

3.2. 인접 리스트 :
* Traversal :(By BFS or DFS) O(ElogV)
* Space : O(V+E)

4. 사용 예시<br/>
네트워크 상에서 가장 근접한 경로를 검색해야 할 때. 구글 맵 등.<br/>
SNS 에서 친구 추천 알고리즘. 



### Trie
1. 개념
* 사전에서 단어를 찾는 것과 같은 자료 구조. 검색 복잡도가 찾고자 하는 단어의 길이에 정비례 한다.
* Prefix Search 가 가능한 것도 큰 특징.Trie로는 시작하는 단어 기준(prefix) 로 모든 단어를 찾을 수 있다.

2. Time Complexity
M은 문자열의 길이, N은 trie 내에 있는 키의 개수 (예: 알파벳 개수)
* 입력 : O(M) 
* 검색  : O(M) 
* 공간 : O(ALPHABET_SIZE * M * N)
* 삭제 : O(M)

3. 사용 예시
* prefix 검색 기능을 활용한 dictionaries 구현
* 스펠 쳌 기능과 같은 대략적인 매칭 알고리즘
* 연락처/전화번호부의 검색 기능



### Segment Tree

1. 개념 <br/>
![](https://www.geeksforgeeks.org/wp-content/uploads/Tree_overview_of_data_structures_4.jpg)
* 값의 특정 집합에 대해 많은 쿼리를 수행할 때
* 쿼리라 함은, 최소값, 최대값, 합계 등 주어진 집합의 인풋 범위 내에서 이루어진 모든 것을 의미한다. 집합 내에 새 요소를 추가하는 것도 포함된다.
* segment tree 는 배열을 이용해 구현된다.


2. Time Complexity
segment tree 최초 생성 : O(N)
Query : O(log N)
Update : O(log N)
공간 : O(N) [Exact space = 2*N-1]

3. 사용 예시
* 최대/최소/합계 등 범위 내 숫자들을 이용하여 간단한 연산을 많이 하는 경우.



### Suffix Tree

1. 개념 <br/>
![](https://www.geeksforgeeks.org/wp-content/uploads/Tree_overview_of_data_structures_5.jpg)
* 텍스트 내에서 특정 패턴을 검색할 때 주로 사용됨.
* 검색 작동이 패턴의 길이에만 정비례 하도록 텍스트를 선처리 하는 것.
* 예, 패턴 검색 알고리즘 (KMP, Z ...) 은 텍스트 길이에 비례한다. 
* 단, 수정이 많이 일어나지 않는 텍스트에 사용해야 한다. 텍스트 에디터 같이 수정이 빈번하게 발생하는 경우에는 적합하지 않다.

* Suffix Tree 는 모든 접미어에 대한 trie의 압축 버전이라고 생각하면 된다.
실제 구현하기 위해서는 다음과 같은 작업이 수행된다.
1) 주어진 텍스트의 모든 접미어를 생성한다.
2) 모든 접미어가 개별 단어라고 생각하고 trie를 생성한다.

2. 사용 예시
* 문자열의 패턴을 찾을 때.
* 반복되는 문자열 부분 중에서 가장 긴 걸 찾을 때.
* 가장 긴 공통 문자열, 회문 등.
