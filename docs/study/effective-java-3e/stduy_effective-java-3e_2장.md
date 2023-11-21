---
layout: default
title: 2장 객체 생성과 파괴
parent: Effective Java 3E
grand_parent: 스터디
nav_order: 1
has_toc: true
---


# 아이템 1 생성자 대신 정적 팩터리 메서드를 고려하라

클라이언트가 클래스의 인스턴스를 얻는 전통적인 수단은 public 생성자다.

**클래스는 생성자와 별도로 정적 팩터리 메서드(static factory method)를 제공할 수 있다.**

```java
public static Boolean valueOf(boolean b){
	return b ? Boolean.TRUE : Boolean.FALSE;
}
```

위 코드는 boolean의 Wrapper class인 Boolean에서 발췌한 간단한 예시다.

기본 타입인 boolean 값을 받아 Boolean 객체 참조로 변환한다.

## 장점

1. **이름을 가질 수 있다. - 메서드 이기 때문**
생성자에 넘기는 매개변수와 생성자 자체만으로는 반환될 객체의 특성을 제대로 설명하지 못한다.
하지만, 정적 팩터리는 메소드 명만 잘 지으면 반환될 객체의 특성을 쉽게 묘사할 수 있다.
2. **호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다. - static 이기 때문**
불변 클래스는 인스턴스를 미리 만들거나 새로 생성한 인스턴스를 캐싱하여 재활용하는 식으로 불필요한 객체 생성을 피할 수 있다.
대표적으로 Boolean.valueOf(boolean) 메서드는 객체를 아예 생성하지 않는다.
- 이 방법은 생성 비용이 큰 객체가 자주 요청되는 상황이라면 성능을 상당히 끌어올려 준다.
비슷한 패턴으로 **플라이웨이트 패턴(Flyweight Pattern)**이 존재한다.
3. **반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다. - 다형성**
반환할 객체의 클래스를 자유롭게 선택할 수 있게 하는 ‘엄청난 유연성’을 제공한다.
4. **입력 매개변수에 따라 매번 다른 클래스의 개체를 반환할 수 있다. - 다형성**
반환 타입의 하위 타입이기만 하면 어떤 클래스의 객체를 반환하든 상관 없다.
5. **정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.**
서비스 제공자 프레임워크를 만드는 근간 ex) JDBC
서비스 제공자 3개의 핵심 컴포넌트
    1. 서비스 인터페이스 : 구현체의 동작을 정의
    2. 제공자 등록 API : 제공자가 구현체를 등록
    3. 서비스 접근 API : 클라이언트가 서비스의 인스턴스를 얻을 때 사용
    
    Connection = 서비스 인터페이스
    DriverManager.registerDriver = 제공자 등록 API
    
    Driver = 서비스 제공자 인터페이스
    
    브리지 패턴(Bridge Pattern), 의존 객체 주입(Dependency Injection) 프레임워크도 강력한 서비스 제공자이다.
    

## 단점

1. **상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.**
2. **정적 팩터리 메서드는 프로그래머가 찾기 어렵다.**
생성자 처럼 API 설명에 드러나지 않으니 사용자는 정적 팩터리 메서드 방식 클래스를 인스턴스화 할 방법을 알아내야 한다.
**정적 팩터리 메서드에 흔히 사용하는 명명 방식**
    1. **from** : 매개 변수를 하나 받아 해당 타입의 인스턴스를 반환하는 형변환 메서드
        
        ```java
        Date d = Date.from(instant);
        ```
        
    2. **of** : 여러 매개변수를 받아 적절한 타입의 인스턴스를 반환하는 집계 메서드
        
        ```java
        Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);
        ```
        
    3. **valueOf** : from과 of의 더 자세한 버전
        
        ```java
        BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);
        ```
        
    4. **instance, getInstance** : 매개변수로 명시한 인스턴스를 반환 단, 같은 인스턴스임은 보장X
        
        ```java
        StackWalker luke = StackWalker.getInstance(options);
        ```
        
    5. **create, newInstance** : (d)와 같지만 매번 새로운 인스턴스를 생성함을 보장
        
        ```java
        Object newArray = Array.newInstance(classObject, arrayLen);
        ```
        
    6. **getType** : (d)와 같으나 생성할 클래스가 아닌다른 클래스에서 팩터리 메서드를 정의
        
        ```java
        FileStore fs = Files.getFileStore(path);
        ```
        
    7. **newType** : (e)와 같으나 생성할 클래스가 아닌다른 클래스에서 팩터리 메서드를 정의
        
        ```java
        BufferedReader br = Files.newBufferedReader(path);
        ```
        
    8. **type** : (f), (g)의 간결한 버전
        
        ```java
        List<Complaint> litany = Collections.list(legacyLitany);
        ```

<hr>
# 아이템 2 생성자에 매개변수가 많다면 빌더를 고려하라

**정적 팩터리와 생성자에는 똑같은 제약이 존재하는데 이는 선택적 매개변수가 많을 때 적절히 대응하기가 어렵다는 것이다.**

```java
NutritionFacts cocaCola = new NutritionFacts
	.Builder(240, 8)
	.calories(100)
	.sodium(35)
	.carbohydrate(27)
	.build()
```

빌더 패턴은 (파이썬과 스칼라에 있는) 명명된 선택적 매개변수를 흉내 낸 것이다.

빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기에 좋다.

```java
public abstract class Pizza {
    public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE }
    final Set<Topping> toppings;

    abstract static class Builder<T extends Builder<T>> {
        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
        public T addTopping(Topping topping) {
            toppings.add(Objects.requireNonNull(topping));
            return self();
        }

        abstract Pizza build();

        // 하위 클래스는 이 메서드를 재정의(overriding)하여
        // "this"를 반환하도록 해야 한다.
        protected abstract T self();
    }
    
    Pizza(Builder<?> builder) {
        toppings = builder.toppings.clone(); // 아이템 50 참조
    }
}
```

Pizza.Builder 클래스는 재귀적 타입 한정을 이용하는 제네릭 타입이다.

```java
public class Calzone extends Pizza {
    private final boolean sauceInside;

    public static class Builder extends Pizza.Builder<Builder> {
        private boolean sauceInside = false; // 기본값

        public Builder sauceInside() {
            sauceInside = true;
            return this;
        }

        @Override public Calzone build() {
            return new Calzone(this);
        }

        @Override protected Builder self() { return this; }
    }

    private Calzone(Builder builder) {
        super(builder);
        sauceInside = builder.sauceInside;
    }

    @Override public String toString() {
        return String.format("%s로 토핑한 칼초네 피자 (소스는 %s에)",
                toppings, sauceInside ? "안" : "바깥");
    }
}
```

```java
public class NyPizza extends Pizza {
    public enum Size { SMALL, MEDIUM, LARGE }
    private final Size size;

    public static class Builder extends Pizza.Builder<Builder> {
        private final Size size;

        public Builder(Size size) {
            this.size = Objects.requireNonNull(size);
        }

        @Override public NyPizza build() {
            return new NyPizza(this);
        }

        @Override protected Builder self() { return this; }
    }

    private NyPizza(Builder builder) {
        super(builder);
        size = builder.size;
    }

    @Override public String toString() {
        return toppings + "로 토핑한 뉴욕 피자";
    }
}
```

## 핵심

```java
**생성자나 정적 팩터리가 처리해야 할 매개변수가 많다면 빌더 패턴을 선택하는 게 더 낫다.**
매개변수 중 다수가 필수가 아니거나 같은 타입이면 특히 더 그렇다. 빌더는 점층적 생성자보다 클라이언트 코드를 읽고 쓰기가 훨씬 간결하고, 자바빈즈보다 훨씬 안전하다.
```
<hr>
# 아이템 3 private 생성자나 열거 타입으로 싱글턴임을 보증하라

**클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기가 어려워질 수 있다.**

- 타입을 인터페이스로 정의한 다음 그 인터페이스를 구현해서 만든 싱글턴이 아니라면 싱글턴 인스턴스를 가짜(mock) 구현으로 대체할 수 없기 때문
1. public static final 필드 방식의 싱글턴
    
    ```java
    public class Elvis {
    	public static final Elvis INSTANCE = new Elvis();
    	private Elvis() { ... }
    	public void leaveTheBuilding() { ... }
    }
    ```
    
    Elvis.INSTANCE를 초기화할 때 딱 한번만 호출한다.
    다만, 예외로는 AccessibleObject.setAccessible을 사용해 private 생성자를 호출할 수 있다.
    
    (생성자를 수정하여 두번째 객체가 생성되려 할 때 예외를 던지게 해야함)
    
2. 정적 팩터리 방식의 싱글턴
    
    ```java
    public class Elvis {
    	public static final Elvis INSTANCE = new Elvis();
    	private Elvis() { ... }
    	public static Elvis getInstance() { return INSTANCE; }
    
    	public void leaveTheBuilding() { ... }
    }
    ```
    
    제 2의 Elvis 인스턴스는 만들어지지 않는다 (리플렉션 제외)
    
# 아이템 3 private 생성자나 열거 타입으로 싱글턴임을 보증하라

**클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기가 어려워질 수 있다.**

- 타입을 인터페이스로 정의한 다음 그 인터페이스를 구현해서 만든 싱글턴이 아니라면 싱글턴 인스턴스를 가짜(mock) 구현으로 대체할 수 없기 때문
1. public static final 필드 방식의 싱글턴
    
    ```java
    public class Elvis {
    	public static final Elvis INSTANCE = new Elvis();
    	private Elvis() { ... }
    	public void leaveTheBuilding() { ... }
    }
    ```
    
    Elvis.INSTANCE를 초기화할 때 딱 한번만 호출한다.
    다만, 예외로는 AccessibleObject.setAccessible을 사용해 private 생성자를 호출할 수 있다.
    
    (생성자를 수정하여 두번째 객체가 생성되려 할 때 예외를 던지게 해야함)
    
2. 정적 팩터리 방식의 싱글턴
    
    ```java
    public class Elvis {
    	public static final Elvis INSTANCE = new Elvis();
    	private Elvis() { ... }
    	public static Elvis getInstance() { return INSTANCE; }
    
    	public void leaveTheBuilding() { ... }
    }
    ```
    
    제 2의 Elvis 인스턴스는 만들어지지 않는다 (리플렉션 제외)
    
3. **열거 타입 방식의 싱글턴(권장)**
    
    ```java
    public enum Elvis {
    	INSTANCE;
    
    	public void leaveTheBuilding() { ... }
    }
    ```
    
    더 간결하고 직렬화가 가능하다.
    또한, 리플렉션 공격에서도 제 2의 인스턴스가 생기는 일을 완벽하게 막아준다.
<hr>
# 아이템 4 인스턴스화를 막으려거든 private 생성자를 사용하라

**추상 클래스로 만드는 것으로는 인스턴스화를 막을 수 없다.**

→ 하위 클래스를 만들어 인스턴스화 하면 그만이다.

private 생성자를 추가하면 클래스의 인스턴스화를 막을 수 있다.

```java
public class UtilityClass {
	// 기본 생성자가 만들어지는 것을 막는다(인스턴스화 방지용)
	private UtilityClass(){
		throw new AssertionError();
	}

	... // 나머지 코드 생략
}
```
<hr>
# 아이템 5 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

많은 클래스가 하나 이상의 자원에 의존한다.

```java
public class SpellChecker {
	private static final Lexicon dictionary = ...;
	
	private SpellChecker() {} // 객체 생성 방지

	public static boolean isValid(String word) { ... }
	public static List<String> suggestions(String type) { ... }
}
```

```java
public class SepllChecker {
	private static final Lexicon dictionary = ...;
	
	private SpellChecker(...) {} // 객체 생성 방지
	public static SpellChecker INSTANCE = new SpellChecker(...);

	public static boolean isValid(String word) { ... }
	public static List<String> suggestions(String type) { ... }
}
```

```java
public class SpellChecker {
	private final Lexicon dictionary;

	**public SpellChecker(Lexicon dictionary){
		// DI
		this.dictionary = Objects.requireNonNull(dictionary);
	}**

	public static boolean isValid(String word) { ... }
	public static List<String> suggestions(String type) { ... }
}
```

자원이 몇 개든 의존 관계가 어떻든 상관없이 잘 작동한다.

또한, dictonary가 final로 불변을 보장하여 여러 클라이언트가 의존 객체들을 안심하고 공유할 수 있다.

이는 **생성자, 정적 팩터리, 빌더** 모두에 똑같이 응용할 수 있다.

## 핵심

```
클래스가 내부적으로 하나 이상의 자원에 의존하고, 그 자원이 클래스 동작에 영향을 준다면 싱글턴과 정적 유틸리티 클래스는 사용하지 않는 것이 좋다. 이 자원들을 클래스가 직접 만들게 해서도 안된다. 대신 필요한 자원을 (혹은 그 자원을 만들어주는 팩터리를) 생성자에 (혹은 정적 팩터리나 빌더에) 넘겨주자.
의존 객체 주입이라 하는 이 기법은 클래스의 유연성, 재사용성, 테스트 용이성을 기막히게 개선해준다.
```

<hr>
# 아이템 6 불필요한 객체 생성을 피하라

똑같은 기능의 객체를 매번 생성하기보다는 객체 하나를 재사용 하는 편이 나을 때가 많다.

```java
String s = new String("bikini"); // 따라 하지 말 것!
```

이러한 문장이 for 혹은 빈번히 호출되는 메서드 안에 있다면 쓸데없는 String 인스턴스가 수백만 개 만들어질 수 있다.

```java
String s = "bikini";
```

이 방식을 사용한다면 같은 가상 머신 안에서 이와 똑같은 문자열 리터럴을 사용하는 모든 코드는 같은 객체를 재사용함이 보장된다.

```java
static boolean isRomanNumeral(String s) {
	return s.matches("^(?=.)M*(C[MD]|D?C{0,3})" + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
}
```

이 방식의 문제는 String.matches 메소드를 이용한다. 이 인스턴스는 한 번 쓰고 버려져서 바로 GC의 대상이 된다.

캐싱을 이용하여 성능을 올려보자

```java
public class RomanNumerals {
	private static final Pattern ROMAN = Pattern
												.compile("^(?=.)M*(C[MD]|D?C{0,3})" +
																 "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
	
	static boolean isRomanNumeral(String s) {
		return ROMAN.matcher(s).matches();
	}
}
```
<hr>
# 아이템 7 다 쓴 객체 참조를 해제하라

C/C++ 처럼 메모리를 직접 관리하다 Java로 오면 GC가 어느정도 해준다.

흔히 하는 실수가 “Java는 메모리 관리 필요 없어” 라고 하는 사람이 있다.
결코 사실이 아니다.

```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        return elements[--size];
    }

    /**
     * 원소를 위한 공간을 적어도 하나 이상 확보한다.
     * 배열 크기를 늘려야 할 때마다 대략 두 배씩 늘린다.
     */
    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }

    public static void main(String[] args) {
        Stack stack = new Stack();
        for (String arg : args)
            stack.push(arg);

        while (true)
            System.err.println(stack.pop());
    }
}
```

테스트를 해도 별 문제 없이 동작한다. 하지만, 큰 문제가 존재한다.
**”메모리 누수”** 이다.
이 코드에서는 스택에서 꺼내진 객체들을 가비지 컬렉터가 회수하지 않는다.

```java
public Object pop() {
	if (size == 0)
		throw new EmptyStackException();
	Object result = elements[--size];
	elements[size] = null; // 다 쓴 참조 해제
	return result;
}
```

다 쓴 참조를 null로 처리하면 다른 이점도 존재한다.

만약 null 처리한 참조를 실수로 사용하게 된다면 NullPointerException을 던지며 종료한다.

→ 처리하지 않았으면 아무 이상 없이 잘못된 일을 수행할 것

**자기 메모리를 직접 관리하는 클래스라면 프로그래머는 항시 메모리 누수에 주의해야 한다.**

**캐시 역시 메모리 누수를 일으키는 주범이다.**

## 핵심

```java
메모리 누수는 겉으로 잘 드러나지 않아 시스템에 수년간 잠복하는 사례도 있다. 이런 누수는 철저한 코드 리뷰나 힙 프로파일러 같은 디버깅 도구를 동원해야만 발견되기도 한다. 그래서 이런 종류의 문제는 예방법을 익혀두는 것이 매우 중요하다.
```

# 아이템 8 finalizer와 cleaner 사용을 피하라

자바는 두 가지 객체 소멸자를 제공한다.
**finailzer는 예측할 수 없고, 상황에 따라 위험할 수 있어 일반적으로 불필요하다.**

**cleaner는 finalizer보다는 덜 위험하지만, 여전히 예측할 수 없고, 느리고, 일반적으로 불필요하다.**

자바의 finlizer와 cleaner은 C++의 파괴자와는 다르다.

C++은 파괴자에서 자원을 회수하는 보편적인 방법이다. 이는 비메모리 자원을 회수하는 용도로도 쓰이지만, 자바에서는 try-with-resources와 try-finally를 사용해 해결한다.

**자바에서 cleaner와 finalizer는 제때 실행되어야 하는 작업은 절대 할 수 없다.**

파일 닫기를 소멸자에다 맡기면 큰 오류를 일으킬 수 있다.

→ 시스템이 파일을 열 수 있는 한계가 있기 때문

자바 언어 명세는 finalizer나 cleaner의 수행 시점뿐 아니라 수행 여부조차 보장하지 않는다.

따라서 프로그램 **생애주기와 상관없는, 상태를 영구적으로 수정하는 작업에서는 절대 finalizer나 cleaner에 의존해서는 안된다.**

System.gc나 System.runFinalization 은 실행될 가능성을 높여줄 뿐, 보장해주진 않는다.

**finalizer와 cleaner는 심각한 성능 문제도 동반한다.**

- AutoCloseable 객체를 생성 후 GC가 수거하기까지 12ns 소요
- finalizer는 550ns가 소요

## 방(Room) 자원을 수거하기 전 반드시 청소(Clean)해야 한다고 가정한 예시

```java
import java.lang.ref.Cleaner;

// 코드 8-1 cleaner를 안전망으로 활용하는 AutoCloseable 클래스 (44쪽)
public class Room implements AutoCloseable {
    private static final Cleaner cleaner = Cleaner.create();

    // 청소가 필요한 자원. 절대 Room을 참조해서는 안 된다!
		// 방을 청소할 때, 수거할 자원들을 담고 있다.
    private static class State implements Runnable {
        int numJunkPiles; // Number of junk piles in this room

        State(int numJunkPiles) {
            this.numJunkPiles = numJunkPiles;
        }

        // close 메서드나 cleaner가 호출한다.
        @Override public void run() {
            System.out.println("Cleaning room");
            numJunkPiles = 0;
        }
    }

    // 방의 상태. cleanable과 공유한다.
    private final State state;

    // cleanable 객체. 수거 대상이 되면 방을 청소한다.
    private final Cleaner.Cleanable cleanable;

    public Room(int numJunkPiles) {
        state = new State(numJunkPiles);
        cleanable = cleaner.register(this, state);
    }

    @Override public void close() {
        cleanable.clean();
    }
}
```

State 인스턴스가 Room의 인스턴스를 참조하게 된다면, 순환 참조가 발생하여 GC가 Room 인스턴스를 회수 해갈 기회가 오지 않는다.