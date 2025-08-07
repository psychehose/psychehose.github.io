해시 테이블은 key - value를 저장하는 자료구조로 해시 함수를 이용해서 키를 인덱스로 변환한다.
실제 value는 배열에 저장된다. 이를 통해 평균적으로 O(1) 시간에 삽입, 삭제, 검색을 수행할 수 있다.

해시 테이블의 핵심 구성요소는 해시 함수와 버킷이라고 할 수 있다.
* 해시 함수: 키를 배열 인덱스로 변환
* 버킷 (Bucket): 실제 데이터가 저장되는 배열의 아이템

### 해시 함수

좋은 해시 함수는 아래 조건을 충족 해야 한다.
1. 결정적: 같은 입력에 대해서는 같은 출력을 보장해야함
2. 효율성: 빠른 계산 속도

STL에서 해시 함수는 타입마다 구현을 달리 한다. 

int, long long 같은 타입들은 이미 그 자체로 hash이기 때문에 더 이상 계산을 안한다. 그래서 항등함수를 사용한다.

double이나 long double 같은 타입들은 거의 그 값에 맞게 해싱하지만 정확히 항등함수는 아니다. 

실제로 clang에서 llbcxx 구현체를 보면 hash 함수를 확인할 수 있다.

#### 정수형 타입

```cpp
// libcxx/include/__functional/hash.h

template <class _Tp>
struct __hash_impl<_Tp, __enable_if_t<is_integral<_Tp>::value && (sizeof(_Tp) <= sizeof(size_t))>>
    : __unary_function<_Tp, size_t> {
  _LIBCPP_HIDE_FROM_ABI size_t operator()(_Tp __v) const _NOEXCEPT { return static_cast<size_t>(__v); }
};

template <class _Tp>
struct __hash_impl<_Tp, __enable_if_t<is_integral<_Tp>::value && (sizeof(_Tp) > sizeof(size_t))>>
    : __scalar_hash<_Tp> {};
```

조건 1. `__enable_if_t<is_integral<_Tp>::value` - 정수 타입인지 확인하는 type trait다.
조건 2. `sizeof(_Tp) <= sizeof(size_t)` -  64비트 시스템에서 size_t는 8바이트라서 8바이트 이하여야 함.

Tip. 두번째 특화에서 정수형 타입이면서 8바이트를 넘는 자료형이 있을까 싶었는데 있다고 한다.

```cpp
// GCC / Clang에서 지원하는 확장
__int128 big_int; // 16 byte
```

#### 부동소수점 타입

```cpp

// libcxx/include/__functional/hash.h
template <class _Tp>
struct __hash_impl<_Tp, __enable_if_t<is_floating_point<_Tp>::value> > : __scalar_hash<_Tp> {
  _LIBCPP_HIDE_FROM_ABI size_t operator()(_Tp __v) const _NOEXCEPT {
    // -0.0 and 0.0 should return same hash
    if (__v == 0.0f)
      return 0;
    return __scalar_hash<_Tp>::operator()(__v);
  }
};

// __scalar_hash 구현
template <class _Tp, size_t = sizeof(_Tp) / sizeof(size_t)>
struct __scalar_hash;

template <class _Tp>
struct __scalar_hash<_Tp, 0> : public __unary_function<_Tp, size_t> {
  _LIBCPP_HIDE_FROM_ABI size_t operator()(_Tp __v) const _NOEXCEPT {
    union {
      _Tp __t;
      size_t __a;
    } __u;
    __u.__a = 0;
    __u.__t = __v;
    return __u.__a;
  }
};

template <class _Tp>
struct __scalar_hash<_Tp, 1> : public __unary_function<_Tp, size_t> {
  _LIBCPP_HIDE_FROM_ABI size_t operator()(_Tp __v) const _NOEXCEPT {
    union {
      _Tp __t;
      size_t __a;
    } __u;
    __u.__t = __v;
    return __u.__a;
  }
};

template <class _Tp>
struct __scalar_hash<_Tp, 2> : public __unary_function<_Tp, size_t> {
  _LIBCPP_HIDE_FROM_ABI size_t operator()(_Tp __v) const _NOEXCEPT {
    union {
      _Tp __t;
      struct {
        size_t __a;
        size_t __b;
      } __s;
    } __u;
    __u.__t = __v;
    return std::__hash_memory(std::addressof(__u), sizeof(__u));
  }
};

template <class _Tp>
struct __scalar_hash<_Tp, 3> : public __unary_function<_Tp, size_t> {
  _LIBCPP_HIDE_FROM_ABI size_t operator()(_Tp __v) const _NOEXCEPT {
    union {
      _Tp __t;
      struct {
        size_t __a;
        size_t __b;
        size_t __c;
      } __s;
    } __u;
    __u.__t = __v;
    return std::__hash_memory(std::addressof(__u), sizeof(__u));
  }
};

template <class _Tp>
struct __scalar_hash<_Tp, 4> : public __unary_function<_Tp, size_t> {
  _LIBCPP_HIDE_FROM_ABI size_t operator()(_Tp __v) const _NOEXCEPT {
    union {
      _Tp __t;
      struct {
        size_t __a;
        size_t __b;
        size_t __c;
        size_t __d;
      } __s;
    } __u;
    __u.__t = __v;
    return std::__hash_memory(std::addressof(__u), sizeof(__u));
  }
};


```

부동소수점인 경우에는 `__scalar_hash<_Tp>::operator()(__v);`를 리턴한다.

`__scalar_hash` 의 구현을 살펴보면 `struct __scalar_hash<_Tp, 1>` 이런식으로 꺽쇠 안에 숫자가 붙어 있다. 

맨 위 정의에 `_Tp / sizeof(size_t)` 를 의미하는데 64bit 시스템에서는 size_t가 8바이트이므로 1인 경우는 double 같은 타입이다

특징으로는 union을 이용해서 메모리 레이아웃을 이용해 비트 캐스팅을 이용한다.

예를 들어 double = 3.14159; 면 메모리 레이아웃은
 `01000000 00001001 00100001 11111001 11110000 00011011 10000110 01101110`

이걸 size_t로 재해석하면 -> 4614256650576692846가 나온다.

```cpp
// Test
int main() {
  std::hash<double> double_hasher;
  cout << "double hash 3.14159::" << double_hasher(3.14159) << endl;
  return 0;
}
// double hash 3.14159 ::4614256650576692846
```

계속해서 2,3,4인 경우는 `std::__hash_memory` 를 리턴하는데 hash_memory 함수는 이렇게 처리되어 있다.

```cpp
#if _LIBCPP_AVAILABILITY_HAS_HASH_MEMORY
[[__gnu__::__pure__]] _LIBCPP_EXPORTED_FROM_ABI size_t __hash_memory(_LIBCPP_NOESCAPE const void*, size_t) _NOEXCEPT;
#else
_LIBCPP_HIDE_FROM_ABI inline size_t __hash_memory(const void* __ptr, size_t __size) _NOEXCEPT {
  return __murmur2_or_cityhash<size_t>()(__ptr, __size);
}
#endif
```

hash_memory 함수는 주소값과, 사이즈를 파라미터로 받는다.

만약 hash_memory가 라이브러리에 있다면 최적화된 플랫폼 전용 해시 함수를 사용하라는 뜻이고
없다면 `__murmur2_or_cityhash<size_t>`를 사용하라는 뜻이다.

murmurhash2는 해시 함수로, 32bit에서 주로 사용되는 함수다.
cityhash는 해시함수로, Google에서 개발했고 64bit에 최적화 되어있다.

이 두 함수중 하나를 거쳐서 size_t를 리턴하게 된다.


#### 포인터 타입

포인터 타입도 부동소수점의 사이즈가 16 바이트를 넘어갈 때의 처리랑 유사하다. hash_memory 함수를 이용한다.

```cpp
template <class _Tp>
struct hash<_Tp*> : public __unary_function<_Tp*, size_t> {
  _LIBCPP_HIDE_FROM_ABI size_t operator()(_Tp* __v) const _NOEXCEPT {
    union {
      _Tp* __t;
      size_t __a;
    } __u;
    __u.__t = __v;
    return std::__hash_memory(std::addressof(__u), sizeof(__u));
  }
};

```

#### 문자열 타입

문자열 타입은 모두 hash.h에 구현된 것과 다르게 각 타입에 구현되어 있다.
`__do_string_hash` 함수를 보면 결국 `std::__hash_memory`를 사용한다.

위에서 쓴 것과 같이 `std::__hash_memory` 플랫폼 함수가 있다면 사용하고, 없다면 murmurhash2 or cityhash를 사용한다.

``` cpp

// libcxx/include/ext/__hash

template <>
struct hash<const char*> : public std::__unary_function<const char*, size_t> {
  _LIBCPP_HIDE_FROM_ABI size_t operator()(const char* __c) const _NOEXCEPT {
    return std::__do_string_hash(__c, __c + strlen(__c));
  }
};

template <>
struct hash<char*> : public std::__unary_function<char*, size_t> {
  _LIBCPP_HIDE_FROM_ABI size_t operator()(char* __c) const _NOEXCEPT {
    return std::__do_string_hash<const char*>(__c, __c + strlen(__c));
  }
};


// libcxx/include/__string/char_traits.h
template <class _Ptr>
inline _LIBCPP_HIDE_FROM_ABI size_t __do_string_hash(_Ptr __p, _Ptr __e) {
  typedef typename iterator_traits<_Ptr>::value_type value_type;
  return std::__hash_memory(__p, (__e - __p) * sizeof(value_type));
}

// libcxx/include/string.h
template <class _CharT, class _Allocator>
struct __string_hash : public __unary_function<basic_string<_CharT, char_traits<_CharT>, _Allocator>, size_t> {
  _LIBCPP_HIDE_FROM_ABI size_t
  operator()(const basic_string<_CharT, char_traits<_CharT>, _Allocator>& __val) const _NOEXCEPT {
    return std::__do_string_hash(__val.data(), __val.data() + __val.size());
  }
};
```


### 해시 충돌

위에서와 같이 해시함수를 통과했을 때 이는 그냥 size_t이므로 무조건 유효한 인덱스가 아닐 것이다. 이를 실제 hash table 구현체에서 buckets.size()으로 나머지 연산을 하면 버킷에 맞게 들어간다. 

```cpp
// 단순한 예
class hash_table {
private:
    size_t bucket_count_;
    size_t compress(size_t hash_value) const {
        // 단순 모듈러 연산
        return hash_value % bucket_count_;
    }
public:
    template<typename Key>
    size_t get_bucket(const Key& key) const {
        size_t hash_val = std::hash<Key>{}(key);
        return compress(hash_val);
    }

```

해싱을 했을 때 다른 입력값이지만 같은 출력이 나오는 경우를 생각할 수 있다. 이런 경우를 해시 충돌이라고 한다. 

해시 충돌을 해결하는 방법이 여러가지가 있는데 대표적인 방식은 체이닝 방식과, 오픈 어드레싱 방법이 있다.  C++ STL에서는 체이닝 방식을 사용한다.

#### 체이닝 방식
체이닝 방식은 각 버킷이 연결 리스트의 헤드 포인터(시작점)를 가지고 있는 것이다. 즉 체이닝은 같은 인덱스로 가는 데이터들을 연결리스트로 묶어서 관리하는 방법을 의미한다.

```cpp
struct Node {
    string key;
    int value;
    Node* next;  // 다음 노드를 가리키는 포인터
};

// buckets
Node* table[4];  // 포인터 배열

// 초기 상태
table[0] = nullptr;
table[1] = nullptr;
table[2] = nullptr;
table[3] = nullptr;
```

정리하자면 배열의 각 칸에는 포인터가 들어가고 이 포인터는 연결리스트의 첫번째 노드를 가르킨다.
충돌 시 같은 연결리스트에 새 노드로 추가하고, 검색 시에 해당 연결리스트를 순서대로 확인한다.

#### 오픈 어드레싱 (Open Addressing)

오픈 어드레싱은 비어 있는 슬롯을 찾아 데이터를 저장하는 방식이다. 비어 있는 슬롯을 찾는 행위를 프로빙(프로빙)이라고 한다. 프로빙은 선형 프로빙, 쿼드러틱 프로빙 (Quardratic), 더블 해싱이 있다.

선형 프로빙은 index를 1씩 증가 시켜서 비어 있는 슬롯을 찾는 것이다.

쿼드러틱 프로빙은 i^2 씩 증가 시켜서 비어 있는 슬롯을 찾는다. (1, 4, 9 ...)

더블 해싱은 다른 해쉬 함수를 사용해서 해싱을 한번 더 하는 것이다. hash2라고 한다면 hash(x) + 1* hash2) -> hash(x) + 2 * hash2 ... 이런식으로 프로빙을 하게 된다.



### 재해싱과 로드 팩터 

hash table에 계속 데이터를 집어 넣어서 buckets이 모두 찬다면 buckets 사이즈를 늘려줘야 한다.

buckets 사이즈를 늘리면 기존의 buckets 있는 모든 요소들을 다시 해싱 해야한다. 이 과정은 평균적으로 O(n) 시간이 소요된다.

그러면 C++ STL에서 해시 테이블을 구현할 때 buckets이 꽉 차면 사이즈를 확장하고 재해싱을 할까?

 hash table에서 로드 팩터 (load factor)라는 변수를 두고 재해싱이 일어날지 말지를 결정한다. 

* 로드 팩터 = 저장된 원소 수 / 배열 크기

로드 팩터의 임계값은 보통 0.75로 알려져있지만, C++ STL에서는 1.0이다.

```cpp
int main() {
  std::unordered_map<int, int> map;
  std::cout << "기본 max_load_factor: " << map.max_load_factor() << std::endl;
  return 0;
}
// 기본 max_load_factor: 1

```


C++ STL은 해시 충돌 해결을 체이닝을 사용하기 때문에 로드 팩터가 1.0을 넘어도 동작이 가능하다. 그리고 평균적으로 버킷의 연결리스트가 많이 길지 않기 때문에 1.0으로 설정되어 있다.

### 마무리

해시 테이블은 빠른 검색을 특징으로 여러 분야에서 사용된다.
* 캐싱시스템  (LRU Cache)
* 컴파일러의 심볼 테이블
