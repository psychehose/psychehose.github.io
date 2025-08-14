## 힙 Heap

완전 이진 트리의 일종으로 두가지 조건 충족

1. 완전 이진 트리
2. 최대 힙 (부모 노드값  > 자식 노드값) , 최소 힙 (부모 노드 값 < 자식 노드값)

## 우선순위 큐 Priority Queue

우선순위 큐는 각 요소가 우선순위를 가지고 있으며, 우선순위가 높은 요소가 먼저 나가는 특수한 큐

1. 힙을 사용하여 효율적으로 구현 가능
2. 삽입 -  새로운 요소를 힙의 마지막에 추가한 후 힙 정렬 O(log n)
3. 삭제 - 힙의 루트 요소를 제거 하고 마지막 요소를 루트로 이동시킨 후 힙 정렬 O(log n)
4. 탐색(루트 요소 확인) - O(1)


### STL 이용한 힙 & 우선순위 큐 구현

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> heap = {10, 20, 30, 5, 15};

    // 힙으로 변환
    std::make_heap(heap.begin(), heap.end());

    std::cout << "Initial max-heap: ";
    for (int i : heap) std::cout << i << " ";
    std::cout << std::endl;

    // 힙에 요소 추가
    heap.push_back(99);
    std::push_heap(heap.begin(), heap.end());

    std::cout << "After adding an element: ";
    for (int i : heap) std::cout << i << " ";
    std::cout << std::endl;

    // 힙에서 최대 요소 제거
    std::pop_heap(heap.begin(), heap.end());
    heap.pop_back();

    std::cout << "After removing the max element: ";
    for (int i : heap) std::cout << i << " ";
    std::cout << std::endl;

    return 0;
}
```


### 우선순위 큐

```cpp
#include <iostream>
#include <queue>
#include <vector>

int main() {
    std::priority_queue<int> pq;
    // 아래처럼 생성하면 최소힙
	// std::priority_queue<int, std::vector<int>, std::greater<int>> min_pq;
    // 요소 삽입
    pq.push(10);
    pq.push(20);
    pq.push(30);
    pq.push(5);
    pq.push(15);

    std::cout << "Priority queue elements: ";
    while (!pq.empty()) {
        std::cout << pq.top() << " ";
        pq.pop();
    }
    std::cout << std::endl;

    return 0;
}
```


```cpp
#include <iostream>
#include <queue>
#include <vector>

struct Task {
    int priority;
    std::string description;

    // 커스텀 비교 함수 (작은 값이 높은 우선순위)
    bool operator<(const Task& other) const {
        return priority > other.priority;
    }
};

int main() {
    std::priority_queue<Task> taskQueue;

    // 요소 삽입
    taskQueue.push({1, "Low priority task"});
    taskQueue.push({3, "High priority task"});
    taskQueue.push({2, "Medium priority task"});

    std::cout << "Tasks in priority order: ";
    while (!taskQueue.empty()) {
        Task t = taskQueue.top();
        std::cout << "(" << t.priority << ", " << t.description << ") ";
        taskQueue.pop();
    }
    std::cout << std::endl;

    return 0;
}
```

### 배열을 이용한 힙 구현

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

class MaxHeap {
private:
    std::vector<int> heap;

    void heapifyUp(int index) {
        while (index > 0) {
            int parent = (index - 1) / 2;
            if (heap[index] > heap[parent]) {
                std::swap(heap[index], heap[parent]);
                index = parent;
            } else {
                break;
            }
        }
    }

    void heapifyDown(int index) {
        int size = heap.size();
        while (index * 2 + 1 < size) {
            int left = index * 2 + 1;
            int right = index * 2 + 2;
            int largerChild = left;

            if (right < size && heap[right] > heap[left]) {
                largerChild = right;
            }

            if (heap[index] < heap[largerChild]) {
                std::swap(heap[index], heap[largerChild]);
                index = largerChild;
            } else {
                break;
            }
        }
    }

public:
    void insert(int value) {
        heap.push_back(value);
        heapifyUp(heap.size() - 1);
    }

    int extractMax() {
        if (heap.empty()) {
            throw std::runtime_error("Heap is empty");
        }

        int maxValue = heap[0];
        heap[0] = heap.back();
        heap.pop_back();
        heapifyDown(0);

        return maxValue;
    }

    void printHeap() {
        for (int value : heap) {
            std::cout << value << " ";
        }
        std::cout << std::endl;
    }

    void preorderTraversal(int index = 0) {
        if (index >= heap.size()) {
            return;
        }

        // 현재 노드 방문
        std::cout << heap[index] << " ";

        // 왼쪽 자식 노드 방문
        preorderTraversal(2 * index + 1);

        // 오른쪽 자식 노드 방문
        preorderTraversal(2 * index + 2);
    }

    void inorderTraversal(int index = 0) {
        if (index >= heap.size()) {
            return;
        }

        // 왼쪽 자식 노드 방문
        inorderTraversal(2 * index + 1);

        // 현재 노드 방문
        std::cout << heap[index] << " ";

        // 오른쪽 자식 노드 방문
        inorderTraversal(2 * index + 2);
    }

    void postorderTraversal(int index = 0) {
        if (index >= heap.size()) {
            return;
        }

        // 왼쪽 자식 노드 방문
        postorderTraversal(2 * index + 1);

        // 오른쪽 자식 노드 방문
        postorderTraversal(2 * index + 2);

        // 현재 노드 방문
        std::cout << heap[index] << " ";
    }
};

int main() {
    MaxHeap heap;

    heap.insert(10);
    heap.insert(20);
    heap.insert(30);
    heap.insert(5);
    heap.insert(15);

    std::cout << "Heap elements: ";
    heap.printHeap();

    std::cout << "Preorder traversal: ";
    heap.preorderTraversal();
    std::cout << std::endl;

    std::cout << "Inorder traversal: ";
    heap.inorderTraversal();
    std::cout << std::endl;

    std::cout << "Postorder traversal: ";
    heap.postorderTraversal();
    std::cout << std::endl;

    std::cout << "Extracted max: " << heap.extractMax() << std::endl;
    std::cout << "Heap after extraction: ";
    heap.printHeap();

    return 0;
}
```

