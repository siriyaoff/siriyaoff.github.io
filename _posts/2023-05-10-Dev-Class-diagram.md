---
layout: single
title: "Class Diagram 정리"
categories:
  - Dev
tags:
  - Class diagram
  - UML
  - Association
  - Inheritance
  - Implementation
  - Dependency
  - Aggregation
  - Composition
---

# 클래스 다이어그램

![class diagram 1](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/classdiagram1.png)

- 아래 설명들에서 A-B이라고 하면 위 그림 기준 A가 왼쪽, B가 오른쪽에 있는 것임

## Association

- A-B이면 A가 B를 참조함
  - A의 필드에 대입하는 형식이 아닌, A의 메소드에서 B의 메소드를 호출하는 등의 형식
- 서로 참조할 경우 끝모양 없이 이어서 나타냄

## Inheritance

- A-B이면 A가 B를 상속받음
  - B가 부모 클래스
- 상속하다 단어의 뜻이 이어주다, 이어받다 둘 다 되기 때문에 B가 A에게 상속하다, A가 B를 상속 받다로 쓰는게 명확할 듯

## Realization(Implementation)

- A-B이면 A가 interface B의 구현체임

## Dependency

- Association과 비슷하게 참조하지만, 일시적으로 유지됨(메소드 내부나 블록 내부에서 호출)
- Association일 경우 Dependency에도 속함(역은 성립하지 않음)

## Aggregation

- A-B이면 A의 instance가 생성될 때 B의 인스턴스를 참조함
  - A의 생성자 안에서 필드에 B를 참조하는 형식

## Composition

- A-B이면 A의 instance가 생성될 때 B의 instance도 생성됨

  - life cycle이 동일함
  - A의 생성자 안에서 B의 생성자도 호출되는 형식

- Instance-level relationship : Dependency, Association, Aggregation, Composition
- Class-level relationship : Inheritance, Implementation
