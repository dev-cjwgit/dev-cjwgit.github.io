---
layout: default
title: 4장 클래스와 인터페이스
parent: Effective Java 3E
grand_parent: 스터디
nav_order: 3
has_toc: true
---

# 아이템 15 클래스와 멤버의 접근 권한을 최소화하라

어설프게 설계 된 컴포넌트와 잘 설계된 컴포넌트의 가장 큰 차이는 **클래스 내부 데이터와 내부 구현 정보를 외부 컴포넌트로부터 얼마나 잘 숨겼느냐**다. - 캡슐화

```java
잘 설계된 컴포넌트는 모든 내부 구현을 완벽히 숨겨, 구현과 API를 깔끔히 분리한다.
```

## 정보 은닉의 장점

- **시스템 개발 속도 향상** : 여러 컴포넌트를 병렬로 개발
- **시스템 관리 비용 감소** : 각 컴포넌트를 빨리 파악하여 디버깅 및 교체가 수월
- **성능 최적화에 도움** : 다른 컴포넌트에 영향을 주지 않고 해당 컴포넌트만 최적화 가능
- **소프트웨어 재 사용성 증가** : 외부에 의존하지 않고 컴포넌트라면 낯선 환경에서도 사용
- **큰 시스템 제작 난이도 감소** : 빅픽쳐가 그려지지 않아도 개별 컴포넌트의 동작 검증 가능

```java
모든 클래스와 멤버의 접근성을 가능한 한 좁혀야 한다.
```

- public 클래스의 인스턴스 필드는 되도록 public이 아니여야 한다.
- public 가변 필드를 갖는 클래스는 일반적으로 스레드 안전하지 않다.
- 클래스에서 public static final 배열 필드를 두거나 이 필드를 반환하는 접근자 메서드를 제공해서는 안 된다.
<hr>
# 아이템 16 public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

```java
class Point {
	public double x;
	public double y;
}
```

```java
class Point {
	private double x;
	private double y;

	public Point(double x, double y) {
		this.x = x;
		this.y = y;
	}

	public double getX() { return x; }
	public double getY() { return y; }

	public void setX(double x) { this.x = x; }
	public void setY(double y) { this.y = y; }
}
```

## 핵심

```java
public 클래스는 절대 가변 필드를 직접 노출해서는 안 된다. 불편 필드라면 노출해도 덜 위험하지만 완전히 안심할 수는 없다. 하지만 package-private 클래스나 private 중첩 클래스에서는 종종 필드를 노출하는 편이 나을 때도 있다.
```
