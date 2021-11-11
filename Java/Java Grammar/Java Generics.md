# Java Generics

생성일: 2021년 11월 11일 오후 3:00

### 제너릭(Generics)

- 제너릭은 클래스 또는 메소드 내부에서 사용할 타입을 외부에서 인자로 받는 것이다.
- 이 인자는 인스턴스를 생성하는 시점이나 메소드를 호출하는 시점에 정할 수 있다.

```java
class Box<T> {
	private T fruit;

	public T get() {
		return fruit;
	}

	public void set(T fruit) {
		this.fruit = fruit;
	}
}

Box<Apple> aBox = new Box<>(); // <>보통 생략.
Box<Orange> aBox = new Box<>();

aBox.set(new Orange()); //error
```

- 해당 클래스에서 사용하는 타입을 T라고 정의하고 해당 타입은 인스턴스 생성 시 받아서 정해진다는 의미이다.
- 컴파일러 단계에서 타입 에러를 잡아낼 수 있음.
- 형 변환이 필요 없음.

### 제너릭 여러 개 사용하기

```java
class DBox<L, R> {
	private L left;
	private R right;

	public void set(L left, R right) {
		this.left = left;
		this.right = right;
	}
}

DBox<String, Integer> box = new DBox<>();
box.set("Apple", 25);
```

### 자주 사용하는 타입 인자

![Untitled](Java%20Generics%2075500b3c27d041e9a186edcbe193a419/Untitled.png)

### 타입 인자 제한하기

- 제너릭을 이용하면 내부에서 사용할 타입을 외부에서 받아올 수 있는데 이것을 좀 더 명확히 제한할 수 있다.

```java
//T가 Number 클래스를 상속하는 하위 클래스라고 범위 제한
class Box<T extends Number> {
	private T object;

	public T get() {
		return object;
	}
	
	public void set(T object) {
		this.object = object;
	}
}

Box<String> sBox = new Box<>(); // error발생

//범위 제한시 특정 메소드 호출 봊
public int toIntValue() {
  return object.intValue();
}
```

### 인터페이스 타입 인자 제한

```java
interface Eatable {
	String eat();
}
// Eatable 인터페이스 구현하고 있다고 보장.
class Box<T extends Eatable> {
private T object;

  public void set(T object) {
    this.object = object;
  }

  public T get() {
    System.out.println(object.eat()); // 호출 가능
    return object;
  }
}
```

### 제너릭 메소드

- 메서드에서도 제너릭을 사용할 수 있다.
- 메소드의 매개변수 타입이나 리턴 타입을 호출 시에 지정할 수 있다는 뜻이 된다.
- 해당 T를 어떤 의미인지 알 수 없기에 메소드 시그니처에 매개변수 타입을 추가로 명시해야 한다. 위치는 메소드 리턴 타입 앞.

```java
public Box<T> makeBox(T, o) {}

public <T> Box<T> makeBox(T o) {
	Box<T> box = new Box<>();
	box.set(o);
	return box;
}
```

### 타입 추론

```java
Box<String> sBox = boxFactory.<String>makeBox("Sweet");
Box<String> sBox = boxFactory.makeBox("Sweet"); // 생략한 모습
```

### 와일드카드

- 와일드 카드는 ? 키워드로 표시되는 것으로 제너릭과 비슷하지만 좀 더 간결하고 확장된 문법 제공

```java
class Unboxer {
  public static <T> T openBox(Box<T> box) {
    return box.get();
  }

  public static <T> void peekBox(Box<T> box) {
    System.out.println(box);
  }
}
//Box<?> 해당 타입으로 여러가지가 올 수 있다.
public static void peekBox(Box<?> box) {
  System.out.println(box);
}

```

**상한 제한 Upper-Bounded**

- 와일드카드의 범위를 특정 객체의 하위 객체로 제한
- 상위 참조변수는 하위 클래스를 참조할 수 있지만, 반대로 하위 참조변수는 상위 클래스를 참조할 수 없다.

```java
public static void peekBox(Box<? extends Number> box) {
	System.out.println(box);
}

public static void outBox(Box<? extends Toy> box) {
  Toy toy = box.get();
  // Box<Robot> 에는 Toy 를 담을 수가 없다.
  box.set(new Toy()); // compile error!
  System.out.println(toy);
}
```

**하한 제한 Lower-Bounded**

- 상한 제한과 반대로 동작.

```java
public static void inBox(Box<? super Robot> box, Robot robot) {
  box.set(robot);
}
```

### 제너릭 메소드와 와일드카드

- 와일드카드의 타입 제한에서 해당 타입을 제너릭으로 외부에서 받아올 수 있다.

```java
public static void outBox(Box<? extends Toy> box) {}
```

### 제너릭 인터페이스

```java
interface Getable<T> {
	T get();
}

class Box<T> implements Getable<T> {
}
```