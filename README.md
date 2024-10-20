# Java Stream Learning

A Guide to Java Streams: In-Depth Tutorial With Examples

[Link](https://stackify.com/streams-guide-java-8/#)

## Stream
- Stream operations are divided into intermediate and terminal operations
- Intermediate operations such as filter() return a new stream on which further processing can be done. 
- Terminal operations, such as forEach(), mark the stream as consumed, after which point it can no longer be used further.

## Stream Creation

```jsx
private static Employee[] arrayOfEmps = {
    new Employee(1, "Jeff Bezos", 100000.0), 
    new Employee(2, "Bill Gates", 200000.0), 
    new Employee(3, "Mark Zuckerberg", 300000.0)
};

// Option 1
Stream.of(arrayOfEmps);

// Java 9+
// cleaner approach to handle an empty stream
// Need flatMap because ofNullable returns Stream<List<Employee>>
List<Employee> empList = null;
Stream.ofNullable(empList)
    .flatMap(Collection::stream)  
    .map(t -> t.getName())
    .forEach(t -> System.out.println(t));
// .map() will ignore the null stream and no NullPointerException
```


```jsx
// Option 2
private static List<Employee> empList = Arrays.asList(arrayOfEmps);

empList.stream();
```

```jsx
// Option 3
Stream.of(arrayOfEmps[0], arrayOfEmps[1], arrayOfEmps[2]);
```

```jsx
// Option 4
Stream.Builder<Employee> empStreamBuilder = Stream.builder();

empStreamBuilder.accept(arrayOfEmps[0]);
empStreamBuilder.accept(arrayOfEmps[1]);
empStreamBuilder.accept(arrayOfEmps[2]);
```

```jsx

```

## Stream Creation (Infinite streams)
- With infinite streams, we need to provide a condition to eventually terminate the processing. One common way of doing this is using limit()

### generate
```jsx
// Here, we pass Math::random() as a Supplier, which returns the next random number
Stream.generate(Math::random)
    .limit(5)
    .forEach(System.out::println);
```

### iterate
```jsx
// Here, we pass 2 as the seed value, which becomes the first element 
// of our stream. This value is passed as input to the lambda, which 
// returns 4. This value, in turn, is passed as input in the next 
// iteration.
Stream<Integer> evenNumStream = Stream.iterate(2, i -> i * 2);

List<Integer> collect = evenNumStream
    .limit(5)
    .collect(Collectors.toList());

assertEquals(collect, Arrays.asList(2, 4, 8, 16, 32));


// Java 9+
Stream.iterate(1, i -> i < 256, i -> i * 2)
      .forEach(System.out::println);
```


## forEach
- forEach is a terminal operation
```jsx
empList.stream().forEach(e -> e.salaryIncrement(10.0));
```

## map
- Converts a stream of TypeA to steam of TypeB

```jsx
Integer[] empIds = { 1, 2, 3 };

List<Employee> employees = Stream.of(empIds)
    .map(employeeRepository::findById)
    .collect(Collectors.toList());
```

## collect + toList
- collect is a terminal operation
```jsx
List<Employee> employees = empList.stream().collect(Collectors.toList());
```

## collect + joining
```jsx
String empNames = empList.stream()
    .map(Employee::getName)
    .collect(Collectors.joining(", "))
    .toString();

assertEquals(empNames, "Jeff Bezos, Bill Gates, Mark Zuckerberg");
```

## collect + toSet
```jsx
Set<String> empNames = empList.stream()
        .map(Employee::getName)
        .collect(Collectors.toSet());
```

## collect + toCollection
- Collectors.toCollection() to extract the elements into any other collection by passing in a Supplier\<Collection\>

```jsx
Vector<String> empNames = empList.stream()
        .map(Employee::getName)
        .collect(Collectors.toCollection(Vector::new));
```

## collect + summarizingDouble
- summarizingDouble() gives statistical information

```jsx
DoubleSummaryStatistics stats = empList.stream()
    .collect(Collectors.summarizingDouble(Employee::getSalary));

assertEquals(stats.getCount(), 3);
assertEquals(stats.getSum(), 600000.0, 0);
assertEquals(stats.getMin(), 100000.0, 0);
assertEquals(stats.getMax(), 300000.0, 0);
assertEquals(stats.getAverage(), 200000.0, 0);

// Option 2
DoubleSummaryStatistics stats = empList.stream()
    .mapToDouble(Employee::getSalary)
    .summaryStatistics();
```

## collect + partitioningBy
- partition a stream into two—based on whether the elements satisfy certain criteria or not

```jsx
List<Integer> intList = Arrays.asList(2, 4, 5, 6, 8);
Map<Boolean, List<Integer>> isEven = intList.stream().collect(
    Collectors.partitioningBy(i -> i % 2 == 0));

assertEquals(isEven.get(true).size(), 4);
assertEquals(isEven.get(false).size(), 1);
```

## collect + groupingBy
- groupingBy() offers advanced partitioning-where we can partition the stream into more than just two groups
- It groups elements of the stream with the use of a Map

```jsx
Map<Character, List<Employee>> groupByAlphabet = empList.stream().collect(
    Collectors.groupingBy(e -> new Character(e.getName().charAt(0))));

assertEquals(groupByAlphabet.get('B').get(0).getName(), "Bill Gates");
assertEquals(groupByAlphabet.get('J').get(0).getName(), "Jeff Bezos");
assertEquals(groupByAlphabet.get('M').get(0).getName(), "Mark Zuckerberg");
```

## collect + groupingBy + mapping
- mapping is like groupingBy but we can group other data into a type other than the element type

```jsx
Map<Character, List<Integer>> idGroupedByAlphabet = empList.stream().collect(
    Collectors.groupingBy(
        e -> new Character(e.getName().charAt(0)),
        Collectors.mapping(Employee::getId, Collectors.toList())
    )
);

assertEquals(idGroupedByAlphabet.get('B').get(0), new Integer(2));
assertEquals(idGroupedByAlphabet.get('J').get(0), new Integer(1));
assertEquals(idGroupedByAlphabet.get('M').get(0), new Integer(3));
```

## collect + reducing
- reducing() takes 3 parameters
    - identity: the identity value for the reduction (also, the value that is returned when there are no input elements)
    - mapper:  a mapping function to apply to each input value
    - op: a BinaryOperator used to reduce the mapped values

```jsx
Double percentage = 10.0;
Double salIncrOverhead = empList.stream().collect(Collectors.reducing(
    0.0,                                    // initial value
    e -> e.getSalary() * percentage / 100,  // get 10% of each salary
    (s1, s2) -> s1 + s2)                    // prev sum + new calc'd value
);

assertEquals(salIncrOverhead, 60000.0, 0);
```

## collect + groupingBy + reducing

```jsx
// Here, we group the employees based on the initial character of 
// their first name. Within each group, we find the employee with the 
// longest name.

Comparator<Employee> byNameLength = Comparator.comparing(Employee::getName);
    
Map<Character, Optional<Employee>> longestNameByAlphabet = empList.stream().collect(
    Collectors.groupingBy(
        e -> new Character(e.getName().charAt(0)),
        Collectors.reducing(BinaryOperator.maxBy(byNameLength))
    )
);

assertEquals(longestNameByAlphabet.get('B').get().getName(), "Bill Gates");
assertEquals(longestNameByAlphabet.get('J').get().getName(), "Jeff Bezos");
assertEquals(longestNameByAlphabet.get('M').get().getName(), "Mark Zuckerberg");
```

## filter
- filter produces a new stream that contains elements of the original stream that pass a given test (specified by a predicate).

```jsx
Integer[] empIds = { 1, 2, 3, 4 };
    
List<Employee> employees = Stream.of(empIds)
    .map(employeeRepository::findById)
    .filter(e -> e != null)
    .filter(e -> e.getSalary() > 200000)
    .collect(Collectors.toList());
```

## findFirst
- findFirst() returns an Optional for the first entry in the stream. The Optional can, of course, be empty

```jsx
Integer[] empIds = { 1, 2, 3, 4 };
    
Employee employee = Stream.of(empIds)
    .map(employeeRepository::findById)
    .filter(e -> e != null)
    .filter(e -> e.getSalary() > 100000)
    .findFirst()
    .orElse(null);
```

## toArray
- Get an array out of a stream
- toArray is a terminal operation

```jsx
Employee[] employees = empList.stream().toArray(Employee[]::new);
```

## flatMap
- A stream can hold complex data structures like Stream<List<String>>. In cases like this, flatMap() helps us to flatten the data structure.
- flatMap converts Stream<List<String>> to a simpler Stream<String>

```jsx
List<List<String>> namesNested = Arrays.asList( 
    Arrays.asList("Jeff", "Bezos"), 
    Arrays.asList("Bill", "Gates"), 
    Arrays.asList("Mark", "Zuckerberg"));

List<String> namesFlatStream = namesNested.stream()
    .flatMap(Collection::stream)
    .collect(Collectors.toList());
```

## peek
- peek is an intermediate operation
- peek is used to perform the specified operation on each element of the stream and returns a new stream

```jsx
Employee[] arrayOfEmps = {
    new Employee(1, "Jeff Bezos", 100000.0), 
    new Employee(2, "Bill Gates", 200000.0), 
    new Employee(3, "Mark Zuckerberg", 300000.0)
};

List<Employee> empList = Arrays.asList(arrayOfEmps);

empList.stream()
    .peek(e -> e.salaryIncrement(10.0))
    .peek(System.out::println)
    .collect(Collectors.toList());
```

## skip, limit
- short-circuiting operations
```jsx
Stream<Integer> infiniteStream = Stream.iterate(2, i -> i * 2);

List<Integer> collect = infiniteStream
    .skip(3)
    .limit(5)
    .collect(Collectors.toList());

assertEquals(collect, Arrays.asList(16, 32, 64, 128, 256));
```

## sorted
- sorted sorts the stream elements based on the comparator passed we pass into it
```jsx
List<Employee> employees = empList.stream()
      .sorted((e1, e2) -> e1.getName().compareTo(e2.getName()))
      .collect(Collectors.toList());
```

## min, max
- min() and max() return the minimum and maximum element in the stream respectively, based on a comparator. They return an Optional since a result may or may not exist

```jsx
Employee maxSalEmp = empList.stream()
      .max(Comparator.comparing(Employee::getSalary))
      .orElseThrow(NoSuchElementException::new);
```

## distinct
- distinct() does not take any argument and returns the distinct elements in the stream, eliminating duplicates. It uses the equals() method of the elements to decide whether two elements are equal or not

```jsx
List<Integer> intList = Arrays.asList(2, 5, 3, 2, 4, 3);
List<Integer> distinctIntList = intList.stream().distinct().collect(Collectors.toList());
```

## allMatch, anyMatch and noneMatch
- These operations all take a predicate and return a boolean. Short-circuiting is applied and processing is stopped as soon as the answer is determined
- allMatch() checks if the predicate is true for ALL the elements in the stream.
- anyMatch() checks if the predicate is true for any one element in the stream (short-circuting)
- noneMatch() checks if no elements are matching the predicate

```jsx
List<Integer> intList = Arrays.asList(2, 4, 5, 6, 8);

boolean allEven = intList.stream().allMatch(i -> i % 2 == 0);
boolean oneEven = intList.stream().anyMatch(i -> i % 2 == 0);
boolean noneMultipleOfThree = intList.stream().noneMatch(i -> i % 3 == 0);

assertEquals(allEven, false);
assertEquals(oneEven, true);
assertEquals(noneMultipleOfThree, false);
```

## reduce
- Most common form of reduce:
    - T reduce(T identity, BinaryOperator<T> accumulator)
    - identity is the starting value and accumulator is the binary operation we repeatedly apply

```jsx
Double sumSal = empList.stream()
    .map(Employee::getSalary)
    .reduce(0.0, Double::sum);
```

## takeWhile
- Java 9+
- Once the condition becomes false, the method stops and returns a new stream containing only the elements that match the predicate
- takeWhile *can* potentially be a more efficient way to *filter*

```jsx
// While both examples yield the same result in this scenario, the 
// difference lies in how takeWhile operates. It stops processing as 
// soon as the predicate is false, whereas filter evaluates the entire 
// stream
Stream.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
      .takeWhile(x -> x <= 5)
      .forEach(System.out::println);

Stream.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
      .filter(x -> x <= 5)
      .forEach(System.out::println);
```

## dropWhile
- Java 9+
- Instead of taking elements while a condition is true, dropWhile skips elements while the condition is true and starts returning elements once the condition becomes false.
```jsx
Stream.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 0, 9, 8, 7, 6, 5, 4, 3, 2, 1, 0)
      .dropWhile(x -> x <= 5)
      .forEach(System.out::println);

// Prints:
6
7
8
9
0
9
8
7
6
5
4
3
2
1
0
```

## concat
- Java 9+
- concat method merges two streams into a single stream.

```jsx
Stream<String> firstStream = Stream.of("A", "B", "C");
Stream<String> secondStream = Stream.of("D", "E", "F");

Stream<String> concatenatedStream = Stream.concat(firstStream, secondStream);
concatenatedStream.forEach(System.out::println);

// Same for IntStream, LongStream, DoubleStream:
IntStream concatenatedStream = IntStream.concat(firstStream, secondStream);
LongStream concatenatedStream = LongStream.concat(firstStream, secondStream);
DoubleStream concatenatedStream = DoubleStream.concat(firstStream, secondStream);
```


## mapToInt, mapToLong, mapToDouble
- These methods convert a stream's elements to an IntStream, LongStream or DoubleStream

```jsx
// IntStream
List<String> numbersAsString = Arrays.asList("10000000000", "20000000000");
IntStream intStream = numbersAsString.stream()
                                       .mapToInt(Integer::parseInt);

// LongStream
List<String> numbersAsString = Arrays.asList("10000000000", "20000000000");
LongStream longStream = numbersAsString.stream()
                                       .mapToLong(Long::parseLong);

// DoubleStream
List<String> numbersAsString = Arrays.asList("1.5", "2.5", "3.5");
DoubleStream doubleStream = numbersAsString.stream()
                                           .mapToDouble(Double::parseDouble);

```

## flatMapToInt, flatMapToLong, flatMapToDouble
- Very similar to mapToXXX, flatMapToXXX flatten the result streams into a single stream

```jsx
// flatMapToInt
Stream<String> strings = Stream.of("1,2,3", "4,5");
IntStream intStream = strings.flatMapToInt(s -> Arrays.stream(s.split(","))
                             .mapToInt(Integer::parseInt));

// flatMapToLong
Stream<String> strings = Stream.of("10000000000,20000000000", "30000000000");
LongStream longStream = strings.flatMapToLong(s -> Arrays.stream(s.split(","))
                               .mapToLong(Long::parseLong));       
                                            
// flatMapToDoble
Stream<String> strings = Stream.of("1.1,2.2", "3.3,4.4");
DoubleStream doubleStream = strings.flatMapToDouble(s -> Arrays.stream(s.split(","))
                                   .mapToDouble(Double::parseDouble));
```

