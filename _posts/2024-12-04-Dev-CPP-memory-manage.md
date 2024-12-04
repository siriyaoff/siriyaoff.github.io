---
layout: single
title: "C++의 메모리 관리"
categories:
  - Dev
tags:
  - C++
  - Memory
  - Garbage collection
  - RAII
  - Smart pointer
  - unique_ptr
  - shared_ptr
  - weak_ptr
  - make_unique
  - make_shared
  - Dangling pointer
---

Java에서는 다양한 gc 알고리즘을 통해 heap 영역의 메모리가 관리된다.  
따라서 사용자가 메모리 release 과정을 신경쓸 필요가 없다.

하지만 C++의 경우, `new` 키워드 또는 `malloc` 함수를 통해 heap에 할당한 메모리는 직접 해제해야 한다.  
이 내용을 포함하여 C++에서는 어떤 방법으로 메모리를 관리하는지 궁금해서 정리해보았다.  
C++에서의 메모리 관리에 대한 사상, 메모리 관리 방법 순서로 나열했다.

## RAII
- RAII(Resource Acquisition Is Initialization) : 자원의 소유권과 생명 주기를 객체이 생명 주기에 연계시키는 디자인 패턴
  - C++에서의 자원 관리를 위한 디자인 원칙
  - 객체가 생성될 때 자원 획득, 소멸될 때 자동으로 해제
  - 주로 스택에서 관리됨
- RAII 객체의 특징
  - exception이 발생해도 소멸자가 자동으로 호출됨
  - 명시적 해제가 필요하지 않음
- 적용 방법
  - RAII를 지원하는 표준 라이브러리 사용
    - 표준 컨테이너 : `vector`, `list`, `map`, `stack`, `queue`, ...
      - 대입 연산자도 RAII 기반으로 구현되어 있기 때문에 새로운 객체 대입 등의 상황에서도 필요한 경우 소멸자를 자동으로 호출함
    - 동적 메모리 관련 : `unique_ptr`, `shared_ptr`(`<memory>`에 존재)
    - 파일 관련 : `fstream`, `ofstream`, `ifstream`
    - 멀티스레드 관련 : `lock_guard`, `unique_lock`
  - 힙 영역에서의 동적 할당이 꼭 필요할 경우, 스마트 포인터를 사용하여 객체 생성
    - 예시  

    ```cpp
    class Resource {
    public:
        Resource() { std::cout << "Resource acquired.\n"; }
        ~Resource() { std::cout << "Resource released.\n"; }
    };

    int main() {
        {
            auto ptr = std::make_unique<Resource>();
        }
        return 0;
    }
    ```
    - Resource는 ptr이 스코프를 벗어날 때 자동으로 소멸함
    - `make_unique`, `make_shared` 함수를 사용해서, `new` 대신 스마트 포인터를 사용하여 heap 영역에 동적 할당 가능

### 스마트 포인터
- RAII를 기반으로 함
  - 객체의 생명 주기를 자동으로 관리
  - 스코프를 벗어날 때 자동으로 메모리 해제
- `unique_ptr` : 독점 소유권을 가짐
  - 동일한 자원은 하나의 `unique_ptr`만 가질 수 있음
  - `unique_ptr` 객체가 스코프를 벗어나면 메모리가 자동으로 해제됨
  - 예시  
  
    ```cpp
    void example() {
        auto ptr = std::make_unique<int>(42);
        cout << *ptr << endl;
    }
    ```
    - `ptr`이 `example` 스코프를 벗어나면 메모리가 자동으로 해제됨
- `shared_ptr` : 참조 카운트를 기반으로 메모리 관리
  - 동일한 자원이 여러 개의 `shared_ptr`에 의해 참조될 수 있음
  - 참조 카운트가 0이 될 때 메모리가 자동으로 해제됨
  - 순환 참조 구조가 만들어지면 메모리 누수 발생 가능
  - 예시  

    ```cpp
    void example() {
      auto ptr1 = std::make_shared<int>(42);  // 참조 카운트: 1
      {
          auto ptr2 = ptr1;  // 참조 카운트: 2
          cout << *ptr2 << endl;
      }  // ptr2가 스코프를 벗어나 참조 카운트: 1
      cout << *ptr1 << endl;  // 여전히 ptr1이 참조 중
    }  // 여기서 ptr1이 스코프를 벗어나 참조 카운트: 0, 메모리 해제
    ```
- `weak_ptr` : `shared_ptr`과 함께 사용됨
  - 참조 카운트를 증가시키지 않는 약한 참조 제공
  - 메모리를 소유하지 않고, 메모리 관리에 관여하지 않음
  - 자원이 해제된 후 `weak_ptr`으로 접근하면 `expired` 상태를 반환함

## C++에서의 메모리 관리 과정
- stack 영역 : Java와 마찬가지로, 스코프를 벗어나면 해제된다.
- heap 영역 : Java와 달리, GC가 없다. 따라서 명시적으로 해제하는 구문을 넣거나, 스마트 포인터를 사용해서 관리해야 한다.
  - `new`, `malloc`으로 할당한 메모리 : **명시적으로** 해제해줘야 한다.
    - `new` : `delete`
    - `malloc` : `free`
    - 해제하지 않을 경우 메모리 누수로 이어지게 된다.
    - 해제하더라도 포인터가 그대로 남아있는 경우, 이 포인터는 해제된 주소를 참조하는 dangling pointer가 된다.
      - 이 포인터를 사용하면 정의되지 않은 동작이 발생하게 된다.
  - `make_unique`, `make_shared` 함수로 생성한 객체 : 메모리 할당 과정이 RAII 원칙을 따르는 스마트 포인터에 의해 관리됨
    - **`new` 또는 `malloc`을 사용하는 것 보다 `make_unique`, `make_shared` 함수를 사용하여 동적 할당을 하는 것이 권장됨**