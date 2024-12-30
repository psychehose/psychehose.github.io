
### Two Pointer

* 두 개의 포인터를 사용하여 배열을 순회하면서 문제를 해결
* 주로 배열이나 리스트와 같은 선형 자료구조를 다룰 때 사용
* 시간 복잡도 O(n^2) -> O(n)

예시

1.  정렬된 배열에서 두 수의 합이 특정 값이 되는 쌍 찾기
```cpp
#include <iostream>
#include <vector>
#include <utility>

std::vector<std::pair<int, int>> findAllPairsWithSum(const std::vector<int>& arr, int targetSum) {
    std::vector<std::pair<int, int>> result;
    int left = 0;
    int right = arr.size() - 1;

    while (left < right) {
        int currentSum = arr[left] + arr[right];
        
        if (currentSum == targetSum) {
            result.push_back({arr[left], arr[right]});
            
            // 중복 원소 건너뛰기
            while (left < right && arr[left] == arr[left + 1]) left++;
            while (left < right && arr[right] == arr[right - 1]) right--;
            
            left++;
            right--;
        } else if (currentSum < targetSum) {
            left++;
        } else {
            right--;
        }
    }
    
    return result;
}

int main() {
    std::vector<int> sortedArray = {1, 2, 3, 4, 5, 5, 6, 7, 8, 9};
    int target = 10;

    auto pairs = findAllPairsWithSum(sortedArray, target);
    
    if (!pairs.empty()) {
        std::cout << "합이 " << target << "인 모든 쌍:" << std::endl;
        for (const auto& pair : pairs) {
            std::cout << pair.first << ", " << pair.second << std::endl;
        }
    } else {
        std::cout << "합이 " << target << "인 쌍을 찾지 못했습니다." << std::endl;
    }

    return 0;
}
```

2. 배열에서 중복 요소 제거

```cpp
#include <iostream>
#include <vector>

int removeDuplicates(std::vector<int>& nums) {
    if (nums.empty()) return 0;
    
    int writePointer = 1;
    for (int readPointer = 1; readPointer < nums.size(); readPointer++) {
        if (nums[readPointer] != nums[writePointer - 1]) {
            nums[writePointer] = nums[readPointer];
            writePointer++;
        }
    }
    return writePointer;
}

int main() {
    std::vector<int> nums = {1, 1, 2, 2, 3, 4, 4, 5};
    int newLength = removeDuplicates(nums);
    
    std::cout << "새로운 배열 길이: " << newLength << std::endl;
    std::cout << "중복이 제거된 배열: ";
    for (int i = 0; i < newLength; i++) {
        std::cout << nums[i] << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

- `writePointer`: 고유한 요소를 쓸 위치를 가리킴
- `readPointer`: 배열을 순회하며 각 요소를 확인
* `readPointer`가 새로운 요소를 발견하면, 그 요소를 `writePointer` 위치에 씀
* 미리 readPointer가 들여다보고 중복여부 파악 후 쓸 지 결정
  
3. 부분 배열의 합이 특정 값이 되는 구간 찾기 (정렬되어 있어야함)

```cpp
#include <iostream>
#include <vector>

std::pair<int, int> findSubarrayWithSum(const std::vector<int>& nums, int targetSum) {
    int left = 0, right = 0;
    int currentSum = 0;

    while (right < nums.size()) {
        currentSum += nums[right];
        
        while (currentSum > targetSum && left < right) {
            currentSum -= nums[left];
            left++;
        }
        
        if (currentSum == targetSum) {
            return {left, right};
        }
        
        right++;
    }
    
    return {-1, -1}; // 합이 targetSum인 부분 배열을 찾지 못한 경우
}

int main() {
    std::vector<int> nums = {1, 4, 20, 3, 10, 5};
    int targetSum = 33;
    
    auto result = findSubarrayWithSum(nums, targetSum);
    
    if (result.first != -1) {
        std::cout << "합이 " << targetSum << "인 부분 배열을 찾았습니다: ";
        std::cout << "인덱스 " << result.first << "부터 " << result.second << "까지" << std::endl;
    } else {
        std::cout << "합이 " << targetSum << "인 부분 배열을 찾지 못했습니다." << std::endl;
    }

    return 0;
}
```

- `left`: 부분 배열의 시작을 가리킴.
- `right`: 부분 배열의 끝을 가리킴
* 현재 부분 배열의 합이 목표값보다 크면 `left`를 오른쪽으로 이동시켜 합을 줄이고, 작으면 `right`를 오른쪽으로 이동시켜 합을 늘림

4. 펠린드롬
```cpp
#include <iostream>
#include <string>

bool isPalindrome(const std::string& s) {
    int left = 0;
    int right = s.length() - 1;
    
    while (left < right) {
        if (s[left] != s[right]) {
            return false;
        }
        left++;
        right--;
    }
    
    return true;
}

int main() {
    std::string str1 = "racecar";
    std::string str2 = "hello";
    
    std::cout << str1 << " 은 팰린드롬" << (isPalindrome(str1) ? "입니다." : "이 아닙니다.") << std::endl;
    std::cout << str2 << " 은 팰린드롬" << (isPalindrome(str2) ? "입니다." : "이 아닙니다.") << std::endl;

    return 0;
}
```

