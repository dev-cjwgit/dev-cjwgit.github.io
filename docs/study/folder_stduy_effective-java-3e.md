---
layout: default
title: Effective Java 3E
nav_order: 1
has_children: true
parent: 스터디
---

# 소개

- 이 책은 입문자 용이 아니다.
- 피애야 할 관행을 보여주는 코드 예시도  자주 등장한다.
안티패턴(AntiPattern) “// 따라하지 말 것!” 과 같은 주석 존재
- 명료성(Clarity)와 단순성(Simplicity) 이 두가지는 무엇보다 중요하다.
- 컴포넌트는 사용자를 놀라게 하는 동작을 해서는 절대 안 된다.
(정해진 동작 및 예측할 수 있는 동작만 수행)
- 컴포넌트는 가능한 한 작되, 그렇다고 너무 작으면 안 된다.
(컴포넌트 : 개별 메서드부터 여러 패키지로 이뤄진 복잡한 프레임워크 까지 재사용 가능한 모든 소프트웨어 요소)
- 이 책은 성능보단 프로그램을 명확, 정확, 유용, 견고, 유연, 관리가 용이하도록 짜는 데 집중한다.
(이러한 목표를 만족하는 코드를 작성했다면 대부분 상황에서 원하는 성능을 도달 할 것)
- [https://git.io/fAm6s](https://git.io/fAm6s) - github src

# Java 8용 언어 명세

- 지원 자료형
    - 인터페이스 (interface)
    - 클래스 (class)
    - 배열 (array)
    - 기본 타입 (primitive)
    
    인터페이스, 클래스, 배열은 참조타입이다.
    
- 어노테이션(annotation)은 인터페이스의 일종, 열거 타입(enum)은 클래스의 일종
- 상속(inheritance) == 서브클래싱(subclassing) 동의어로 사용