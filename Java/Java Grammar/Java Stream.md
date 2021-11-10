# 자바 Stream

생성일: 2021년 11월 9일 오후 1:36

### 스트림

- 자바 8에서 추가한 스트림은 람다를 활용할 수 있는 기술 중 하나이다.
- 스트림은 '데이터의 흐름’이다. 배열 또는 컬렉션 인스턴스에 함수 여러 개를 조합해서 원하는 결과를 필터링하고 가공된 결과를 얻을 수 있다.
- 람다를 이용해서 코드의 양을 줄이고 간결하게 표현할 수 있습니다. 즉, 배열과 컬렉션을 함수형으로 처리할 수 있다.
- 병렬처리(multi-threading)이 가능하여 쓰레드를 이용해 많은 요소들을 빠르게 처리할 수 있다.
- 스트림의 과정은 3가지로 나눌 수 있다.
    - 생성하기: 스트림 인스턴스 생성
    - 가공하기: 필터링(filtering) 및 매핑(mapping)등 원하는 결과를 만들어가는 중간 작업(intermediate operations)
    - 결과 만들기: 최종적으로 결과를 만들어내는 작업
    

### 생성하기

**배열 스트림**

- 스트림은 배열 또는 컬렉션 인스턴스를 이용해서 생성할 수 있다.

```java
String[] arr = new String[] {"a", "b", "c"};
Stream<String> stream = Arrays.stream(arr);
```

**컬렉션 스트림**

- 컬렉션 타입(Collection, List, Set)의 경우 인터페이스에 추가된 디폴트 메소드 stream을 이용해서 만들 수 있다.

```java
public interface Collection<E> extends Iterable<E> {
	default Stream<E> stream() {
		return StreamSupport.stream(spliterator(), false);
	}
//....
}

List<String> list = new Arrays.toList("a", "b", "c");
Stream<String> stream = list.stream();
Stream<String> parallelStream = list.parallelStream(); //병렬 처리 스트림
```

**비어 있는 스트림**

- 빈 스트림은 요소가 없을 때 null 대신 사용할 수 있다.

```java
public Stream<String> streamOf(List<String> list) {
	return list == null || list.isEmpty()
	? Stream.empty()
	: list.stream();
}
```

**Stream.builder()**

- 빌더를 사용하면 스트림에 직접적으로 원하는 값을 넣을 수 있다.

```java
Stream<String> builderStream = 
	Stream.<String>.builder()
		.add("dfdf").add("dfd")
		.build();
	
```

**Stream.generate()**

- generate 메서드를 이용하면 Supplier<T>에 해당하는 람다로 갑을 넣을 수 있다.
- Supplier<T> 는 인자는 없고 리턴 값만 있는 함수형 인터페이스이기에 람다에서 리턴한 값이 들어간다.

```java
public static<T> Stream<T> generate(Supplier<T> s) {}

Stream<String> generateStream = 
	Stream.generate(() -> "gen").limit(5); 
```

**Stream.iterate()**

- iterate 메서드를 사용하면 초기값과 해당 값을 다루는 람다를 이용해서 스트림에 들어갈 요소를 만든다.

```java
Stream<Integer> iteratedStream = 
	Stream.iterate(30, n -> n + 2).limit(5);
```

**기본 타입형 스트림**

```java
//제네릭을 사용하지 않기 때문에 불필요한 오토박싱(auto-boxing)이 일어나지 않습니다.
 //필요한 경우 boxed 메소드를 이용해서 박싱(boxing)할 수 있습니다.
IntStream intStream = IntStream.range(1, 5).boxed();
LongStream longStream = LongStream.rangeClosed(1, 5); // 5포함

*//Random 클래스는 난수를 가지고 세가지 타입의 스트림(IntStream, LongStream)
//DoubleStream을 만들어 낼 수 있다.
Dou*bleStream doubles = new Random().doubles(3);
```

**문자열 스트림**

```java
Stream<String> stringStream = 
  Pattern.compile(", ").splitAsStream("Eric, Elena, Java");
```

**파일 스트림**

```java
Stream<String> lineStream = 
	Files.lines(Paths.get("file.txt"), Charset.forName("UTF-8"));
```

**병렬 스트림 Parallel Stream**

- 스트림 생성시 parallelStream 메소드를사용해서 병렬 스트림을 쉽게 생성할 수 있다.

```java
Stream<Product> parallelStream = productList.parallelStream();
Arrays.stream(arr).parallel();

IntStream intStream = IntStream.range(1, 150).parallel();
//다시 sequential로 되돌림.
IntStream intStream = intStream.sequential();

boolean isParallel = parallelStream.isParallel();

//병렬 처리
boolean isMany = parallelStream
.map(product -> product.getAmount() * 10)
.anyMatch(amount -> amount > 200);
```

**스트림 연결하기**

- Stream.concat 메서드를 이용해 두 개의 스트림을 연결해서 새로운 스트림 생성

```java
Stream<String> stream1 = Stream.of("dfd");
Stream<String> stream2 = Stream.of("dfdf");
Stream<String> concat = Stream.concat(stream1, stream2);
```

### 가공하기

- 전체 요소 중에서 내가 원하는 것만 뽑아낼 수 있다.
- 이러한 작업은 channing할 수 있다.

**Filtering**

- 필터는 스트림 내 요소들을 하나씩 평가해서 걸러내는 작업이다.

```java
Stream<T> filter(Predicate<? super T> predicate);

Stream<String> stream = 
names.stream()
.filter(name -> name.contains("a"));
```

**Mapping**

- 맵은 스트림 내 요소들을 하나씩 특정 값으로 변환해준다.

```java
<R> Stream<R> map(Function<? super T, ? extends R> mapper);

Stream<String> stream = 
names.stream()
.map(String::toUppercase);

Stream<Integer> stream = 
  productList.stream()
  .map(Product::getAmount);
```

- flatMap: 중첩 구조를 한 단계 제거하고 단일 컬렉션으로 만들어주는 역할을 한다. (이러한 작업을 flattening)

```java
List<List<String>> list = 
  Arrays.asList(Arrays.asList("a"), 
                Arrays.asList("b"));

//flatMap을 사용해서 중첩 구조를 제거한 후 작업 가능
List<String> flatList = 
list.stream()
.flatMap(Collection::stream)
.collect(Collectors.toList());

students.stream()
	.flatMapToInt(student -> 
								IntStream.of(student.getKor(),
														student.getEng()))
.average().isPresent(avg -> Math.round(avg * 10));
```

**Sorting**

- 정렬의 방법은 다른 정렬과 마찬가지로 Comparator를 이용한다.

```java
Stream<T> sorted();
Stream<T> sorted(Comparator<? super T> Comparator);

IntStream.of(15, 11, 20)
.sorted()
.boxed()
.collect(Collectors.toList());

List<String> lang = 
  Arrays.asList("Java", "Scala", "Groovy", "Python", "Go", "Swift");
//내림차순
lang.stream()
.sorted(Comparator.reverseOrder())
.collect(Collectors.toList());

lang.stream()
  .sorted(Comparator.comparingInt(String::length))
  .collect(Collectors.toList());
// [Go, Java, Scala, Swift, Groovy, Python]

//s1 < s2 -> 내림차순

lang.stream()
  .sorted((s1, s2) -> s2.length() - s1.length())
  .collect(Collectors.toList());
// [Groovy, Python, Scala, Swift, Java, Go]
```

### Iterating

- 스트림 내 요소들 각각을 대상으로 특정 연산을 수행하는 메소드로는 peek이 있다.
- peek은 특정 결과를 반환하지 않는 함수형 인터페이스 Consumer를 인자로 받는다.

```java
int sum = IntStream.of(1, 3, 5)
.peek(System.out::println)
.sum();
```

### 결과 만들기

**Calculating**

- 스트림 API은 다양한 종료 작업을 제공한다. 최소, 최대, 합 ,평균 등 기본형 타입으로 결과를 만들어 낼 수 있다.

```java
long count = IntStream.of(1, 2, 3).count();
long sum = IntStream.of(1, 3, 4).sum();

OptionalInt min = IntStream.of(1, 3, 5, 7, 9).min();
OptionalInt max = IntStream.of(1, 3, 5, 7, 9).max();

//ifPresent 메서드를 이용해서 Optional처리
DoubleStream.of(1.2, 3.2)
.average()
.ifPresent(System.out::println);
```

**Reduction**

- 스트림은 reduce라는 메소드를 이용해서 결과를 만들어낸다.
- reduce 메소드는 3가지 파라미터를 받을 수 있다.
    - accumulator: 각 요소를 처리하는 계산 로직
    - identity: 계산을 위한 초기값으로 스트림이 비어도 이 값 리턴
    - combiner: 병렬 스트림에서 나눠 계산한 결과를 하나로 합치는 로직

```java
Optional<T> reduce(BinaryOperator<T> accumulator);

T reduce(T identity, BinaryOperator<T> accumulator);

<U> U reduce(U identity, BinaryOperator<U, ? super T, U> accumulator,
BinaryOperator<U> Combiner);

OptionalInt reduced = 
IntStream.range(1, 5)
.reduce((a, b) -> {
	return Integer.sum(a, b);
});

int reducedTwoParams = 
IntStream.range(1, 4)
.reduce(10, Integer::sum);

//병렬 스트림에서만 동작.
Integer reducedParams = Arrays.asList(1, 2, 3)
.parallelStream()
.reduce(10, Integer::sum,
(a, b) -> return a + b); // 36
//간단한 경우 부가적인 처리가 필요하기에 시퀀셜보다 병렬이 느릴 수 있다.
```

**Collecting**

- Collector 타입의 인자를 받아서 처리를 하는데 자주 사용하는 작업은 Collectors 객체에서 제공하고 있다.
- Collectors.toList():작업한 결과를 담은 리스트를 반환

```java
List<String> collectorCollection = 
productList.stream()
.map(Product::getName)
.collect(Collectors.toList());
```

- Collectors.joining(): 작업한 결과를 하나의 스트링으로 이어 붙일 수 있다.

```java
string listToString = 
productList.stream()
.map(Product::getName)
.collect(Collectors.joining(", ", "<", ">"));
//delimiter
//prefix: 결과 맨 앞에 붙는 문자
//suffix: 결과 맨 뒤에 붙는 문자
//<potatoes, orange, lemon, bread, sugar>
```

- averagingInt(): 숫자 값의 평균

```java
Double averageAmount = 
	productList.stream()
.collect(Collectors.averagingInt(Product::getAmount));
```

- Collectors.summingInt(): 숫자 값의 합

```java
Integer summingAmount = 
 productList.stream()
  .collect(Collectors.summingInt(Product::getAmount));

// mapToInt -> IntStream으로 변경
Integer summingAmount =
productList.stream()
.mapToInt(Product::getAmount)
.sum();
```

- Collectors.summarizingInt()

```java
IntSummaryStatistics statistics = 
 productList.stream()
  .collect(Collectors.summarizingInt(Product::getAmount));
//IntSummaryStatistics {count=5, sum=86, min=13, average=17.200000, max=23}
```

- Collectors.groupingBy(): 특정 조건으로 요소들을 그룹지을 수 있다.

```java
Map<Integer, List<Product>> collectorMapOfLists =
 productList.stream()
  .collect(Collectors.groupingBy(Product::getAmount));
//결과는 맵타입 같은 수량이면 리스트로 묶어서 보여준다.

{23=[Product{amount=23, name='potatoes'}, 
     Product{amount=23, name='bread'}], 
 13=[Product{amount=13, name='lemon'}, 
     Product{amount=13, name='sugar'}], 
 14=[Product{amount=14, name='orange'}]}
```

Matching

- 매칭은 조건식 람다 Predicate를 받아서 해당 조건을 만족하는 요소가 있는지 체크한 결과를 리턴한다.

```java
//하나라도 조건을 만족하는 요소가 있는 지
boolean anyMatch = names.stream().anyMatch(name -> name.contains("a"));
//모두 조건을 만족하는지
boolean allMatch = names.stream().allMatch(name -> name.length() > 3);
//모두 조건을 만족하지 않는지
boolean noneMatch = names.stream().noneMatch(name -> name.endsWith("s"));

```

Iterating

- foreach는 요소를 돌면서 실행되는 최종 작업

```java
names.stream().forEach(System.out::println);
```