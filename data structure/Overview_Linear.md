* 배열 Array
배열은 같은 종류의 요소(Element)를 같은 공간에 저장하기 위해 사용되는 자료 구조다. 배열의 크기는 데이터를 담기 전에 사전에 정의되어야 한다.


* Linked List
 - 배열과 같은 선형 자료구조. 다만 각 요소(Element)가 개별적인 객체라는 점이 다르다. 각 요소(==노드)는 두 가지로. 구성이 되는데, 하나는 그 데이터 값 자체이고 다른 하나는 다음 노드에 대한 참조값이다. 
 - One big drawback of linked list is, random access is not allowed. With arrays, we can access i’th element in O(1) time. In linked list, it takes Θ(i) time.
 Linked List의 단점은 무작위 접근이 불가능하다는 것. 배열에서는 배열 내의 어떤 값에 접근해도 그 시간이 동일한데(O(1)), Linked List 에서는 요소의 개수에 따라 접근하는 데에 걸리는 시간이 증가한다(O(n)).

* Stack
 - 스택은 LIFO(Last in, First Out) 이라고도 불리는 추상 자료형이다. 구성 요소를 추가하는 Push, 마지막 요소를 제거하는 Pop 연산을 내장한다.
 - 스택은 주로 단어를 거꾸로 뒤집을 때, 텍스트 에디터에서 실행취소(Ctrl+Z) 할 때, 웹 브라우저에서 뒤로 버튼을 눌릴 때에 사용된다.

* Queue
 - 큐는 FIFO(First in, First out) 이라고 불리는 추상 자료형이다. 해당 집합에 새 요소를 추가하는 enqueue, 가장 첫 요소를 삭제하는 dequeue 연산을 내장한다. 
 - 큐는 두 프로세스 간의 비동기 데이터 처리에서 주로 활용된다. I/O Buffer, Pipe, File I/O 등에도 활용된다.