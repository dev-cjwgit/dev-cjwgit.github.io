---
layout: default
title: 3장 모든 객체의 공통 메서드
parent: Effective Java 3E
grand_parent: 스터디
nav_order: 2
has_toc: true
---

# 아이템 10 equals는 일반 규약을 지켜 재정의하라

## 다음 상황 중 하나에 해당 된다면 재정의 하지 않는 것이 최선이다.

- **각 인스턴스가 본질적으로 고유하다.**
값을 표현하는 것이 아닌, 동작 개체를 표현하는 클래스
ex) Thread
- **인스턴스의 ‘논리적 동치성(logical equality)’을 검사할 일이 없다.**
java.util.regex.Pattern은 equals를 재정의해서 두 Pattern의 인스턴스가 같은 정규표현식을 나타내는지 검사하는 방법도 있다.
하지만 설계자는 클라이언트가 이 방식을 원치 않거나, 필요하지 않다고 판단하면 구현 X
- **상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다.**
대부분의 Set 구현체는 AbstractSet이 구현한 equals를 상속받아 쓰고, 
List 구현체들은 AbstractList로 부터, 
Map 은 AbstractMap으로 부터 상속받아 그대로 쓴다.
- **클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없다.**
    
    ```java
    @Override
    public boolean equals(Object o) {
    	throw new AssertionError(); // 호출 금지!
    }
    ```
    
    실수라도 equals를 호출 하는 것을 방지하고 싶다면 위 코드를 활용하자.
    
    **그럼 언제 재정의할까?**
    
    ex) Integer와 String처럼 값 클래스의 값이 같은지 알고싶다.
    

## equals 메서드 재정의 시 일반 규약

common : null이 아닌 모든 참조 값

- **반사성(reflexivity)** : x에 대해, x.equals(x)는 true
- **대칭성(symmetry)** : x, y에 대해, x.equals(y)가 true면 y.equals(x)도 true
- **추이성(transitivity)** : x, y, z에 대해, x.equals(y)가 true이고 y.equals(z)도 true면 x.equals(z)도 true
- **일관성(consistency)** : x, y에 대해, x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환
- **null-아님** : x에 대해, x.equals(null)은 false

```java
// 코드 10-1 잘못된 코드 - 대칭성 위배! (54-55쪽)
public final class CaseInsensitiveString {
    private final String s;

    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
    }

    // 대칭성 위배!
    @Override public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString)
            return s.equalsIgnoreCase(
                    ((CaseInsensitiveString) o).s);
        if (o instanceof String)  // 한 방향으로만 작동한다!
            return s.equalsIgnoreCase((String) o);
        return false;
    }

    // 문제 시연 (55쪽)
    public static void main(String[] args) {
        CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
        String s = "polish";

        List<CaseInsensitiveString> list = new ArrayList<>();
        list.add(cis);

        System.out.println(list.contains(s));
    }
}
```

cis.equals(s)는 true를 반환한다.

하지만 String.equals(Object) 는 CaseInsensitiveString의 존재를 알지 못한다.

```java
// 수정한 equals 메서드 (56쪽)
@Override
public boolean equals(Object o) {
	return o instanceof CaseInsensitiveString &&
		((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
}
```

문제를 해결하려면 CaseInsensitiveString의 equals를 String과도 연동 하겠다는 것을 포기한다.

```java
@Override
public boolean equals(Object o) {
	if(!(o instanceof ColorPoint))
		return false;

	return super.equals(o) && ((ColorPoint) o).color == color;
}
```

Point는 equals 색상을 무시하고 ColorPoint의 equals는 입력 매개변수의 클래스 종류가 다르다면 매번 false만 반환한다.

**ColorPoint instanceof Point = true;**

**Point instanceof ColorPoint = false;**

```java
@Override
public boolean equals(Object o) {
	if (!(o instanceof Point))
		return false;

	if (!(o instanceof ColorPoint))
		return o.equals(this);

	return super.equals(o) && ((ColorPoint) o).color = color;
}
```

ColorPoint p1 = new ColorPoint(1, 2, Color.RED);

Point p2 = new Point(1, 2);

ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);

**p1.equals(p2), p2.equals(p3) = true**

**p1.equals(p3) = false**

## 중간 결론

```java
구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않는다.
```

```java
@Override
public boolean equals(Object o) {
	if (o == null || o.getClass() != getClass())
		return false;

	Point p = (Point) o;
	return p.x == x && p.y == y;
}
```

같은 구현 클래스의 객체와 비교 할 때만 true를 반환한다.

LSP는 하위 타입이 상위 타입으로 교체되어 쓸 수 있어야 하는데 이 방식은 그렇게 사용하지 못한다.

```java
public class ColorPoint {
	private final Point point;
	private final Color color;
	
	public COlorPoint(int x, int y, Color color) {
		point = new Point(x, y);
		this.color = Objects.requireNonNull(color);
	}

	public Point asPoint() {
		return point;
	}

	@Override
	public boolean equals(Object o) {
		if (!(o instanceof ColorPoint))
			return false;

		ColorPoint cp = (ColorPoint) o;
		return cp.point.equals(point) && cp.color.equals(color);
	}
}
```

**구체 클래스를 확장해 값을 추가한 클래스**

java.sql.Timestamp는 java.util.Date를 확장한 후 nanoseconds 필드를 추가

⇒ Timestamp의 equals는 대칭성을 위배하며, Date 객체와 한 컬렉션에 넣거나 서로 섞어 사용하면 엉뚱하게 동작할 수 있다.

**Timestamp의 API 설명에는 Date와 섞어 쓸 때 주의 사항을 언급**하고 있다.

**※ Timestamp 설계는 실수니 절대 따라해선 안된다.**

**equals의 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안 된다.**

- 일관성 조건을 만족시키기가 아주 어렵다.
    - java.net.URL의 equals는 주어진 URL과 매핑된 호스트의 IP주소를 이용한다.
    호스트 이름을 IP 주소로 바꾸려면 네트워크를 통해야 하는데, 그 결과가 한상 같다고 보장할 수 없다 - 일관성 문제
    
    URL의 equals또한 일반 규악을 어기게 하고, 실무에서도 종종 문제를 일으킨다. (따라하지 말 것)
    

```java
@Override
public boolean equals(Object o) {
	if (o == null)
		return false;
}
```

동치성을 검사하려면 equals는 건네받은 객체를 적절히 형 변환 후 필수 필드들의 값을 알아내야 한다.

```java
@Override
public boolean equals(Object o) {
	if (!(o instanceof MyType))
		return false;

	MyType mt = (MyType) o;
}
```

## 단계적으로 양질의 equals 메서드 구한 방법 정리

1. **== 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.**
자기 자신이면 true를 반환한다
→ 단순 성능 최적화 용도
2. **instanceof 연산자로 입력이 올바른 타입인지 확인한다.**
그렇지 않다면 false를 반환한다.
Set, List, Map, Map.Entry등 클래스 equals가 아닌 인터페이스를 이용해야 한다.
3. **입력을 올바른 타입으로 형변환한다.**
2번에서 instanceof 검사를 했기 때문에 100% 성공한다.
4. **입력 객체와 자기 자신의 대응되는 ‘핵심’ 필드들이 모두 일치하는지 하나씩 검사한다.**
모든 필드가 일치하면 true, 하나라도 다르면 false를 반환한다.

float과 double을 제외한 기본 타입 필드는 == 연산자로 비교, 참조 타입 필드는 각각의 equals, float와 double는 Float.compare()와 Double.comapre()로 비교한다.

- Float.NaN, -0.0f, 특수한 부동 소수 값 등을 다뤄야 하기 때문이다.

**euqals를 다 구현했다면 세 가지만 자문해보자.**

1. **대칭적인가?**
2. **추이성이 있는가?**
3. **일관적인가?**

## 핵심

```
꼭 필요한 경우가 아니라면 equals를 재정의하지 말자. 많은 경우에 Object의 equals가 여러분이 원하는 비교를 정확히 수행해준다.
재정의해야 할 때는 그 클래스의 핵심 필드 모두를 빠짐없이, 다섯 가지 규악을 확실히 지켜가며 비교해야한다.
```