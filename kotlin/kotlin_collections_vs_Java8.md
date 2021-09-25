# kotlin.collections vs Java 8

- Kotlin stdlib 에는 average, count, distinct, filtering, finding, grouping, joining, mapping, min, max, partitioning, slicing, sorting, summing, to/from arrays, to/from lists, to/from maps, union, co-iteration, 모든 기능 패러다임 등에 대한 함수가 있다. 따라서 이러한 기능을 사용하여 작은 1-라인을 생성할 수 있으며 Java 8의 더 복잡한 구문을 사용할 필요가 없다.

~~내 생각에 기본 제공되는 Java 8 'Collectors' 클래스에서 빠진 것은 summarization 밖에 없는 것 같다 ([이 질문에 대한 또 다른 답변](https://stackoverflow.com/a/34661483/3679676) → 이 간단한 해결책입니다)~~

~~One thing missing from both is batching by count, which is seen in [another Stack Overflow answer](https://stackoverflow.com/a/34504231/3679676) and has a simple answer as well. Another interesting case is this one also from Stack Overflow: [Idiomatic way to spilt sequence into three lists using Kotlin](https://stackoverflow.com/questions/32574783/idiomatic-way-to-spilt-sequence-into-three-lists-using-kotlin). And if you want to create something like `Stream.collect` for another purpose, see [Custom Stream.collect in Kotlin](https://stackoverflow.com/questions/34639208)~~

**EDIT 11.08.2017:** Chunked/windowed collection operations were added in kotlin 1.2 M2, see [https://blog.jetbrains.com/kotlin/2017/08/kotlin-1-2-m2-is-out/](https://blog.jetbrains.com/kotlin/2017/08/kotlin-1-2-m2-is-out/)

---

It is always good to explore the [API Reference for kotlin.collections](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/index.html) as a whole before creating new functions that might already exist there.

Here are some conversions from Java 8 `Stream.collect` examples to the equivalent in Kotlin:

### Accumulate names into a List

---

```java
// Java:
List<String> list = people.stream().map(Person::getName).collect(Collectors.toList());
```

```kotlin
// Kotlin:
val list = people.map { it.name }  // toList() not needed
```

### Convert elements to strings and concatenate them, separated by commas

---

```java
// Java:
String joined = things.stream()
                       .map(Object::toString)
                       .collect(Collectors.joining(", "));
```

```kotlin
// Kotlin:
val joined = things.joinToString(", ")
```

### Compute sum of salaries of employee

---

```java
// Java:
int total = employees.stream()
                      .collect(Collectors.summingInt(Employee::getSalary)));
```

```kotlin
// Kotlin:
val total = employees.sumBy { it.salary }
```

### Group employees by department

---

```java
// Java:
Map<Department, List<Employee>> byDept
     = employees.stream()
                .collect(Collectors.groupingBy(Employee::getDepartment));
```

```kotlin
// Kotlin:
val byDept = employees.groupBy { it.department }
```

### Compute sum of salaries by department

---

```java
// Java:
Map<Department, Integer> totalByDept
     = employees.stream()
                .collect(Collectors.groupingBy(Employee::getDepartment,
                     Collectors.summingInt(Employee::getSalary)));
```

```kotlin
// Kotlin:
val totalByDept = employees.groupBy { it.dept }.mapValues { it.value.sumBy { it.salary }}
```

### Partition students into passing and failing

---

```java
// Java:
Map<Boolean, List<Student>> passingFailing =
     students.stream()
             .collect(Collectors.partitioningBy(s -> s.getGrade() >= PASS_THRESHOLD));
```

```kotlin
// Kotlin:
val passingFailing = students.partition { it.grade >= PASS_THRESHOLD }
```

### Names of male members

---

```java
// Java:
List<String> namesOfMaleMembers = roster
    .stream()
    .filter(p -> p.getGender() == Person.Sex.MALE)
    .map(p -> p.getName())
    .collect(Collectors.toList());
```

```kotlin
// Kotlin:
val namesOfMaleMembers = roster.filter { it.gender == Person.Sex.MALE }.map { it.name }
```

### Group names of members in roster by gender

---

```java
// Java:
Map<Person.Sex, List<String>> namesByGender =
      roster.stream().collect(
        Collectors.groupingBy(
            Person::getGender,
            Collectors.mapping(
                Person::getName,
                Collectors.toList())));
```

```kotlin
// Kotlin:
val namesByGender = roster.groupBy { it.gender }.mapValues { it.value.map { it.name } }
```

### Filter a list to another list

---

```java
// Java:
List<String> filtered = items.stream()
    .filter( item -> item.startsWith("o") )
    .collect(Collectors.toList());
```

```kotlin
// Kotlin:
val filtered = items.filter { it.startsWith('o') }
```

### Finding shortest string a list

---

```java
// Java:
String shortest = items.stream()
    .min(Comparator.comparing(item -> item.length()))
    .get();
```

```kotlin
// Kotlin:
val shortest = items.minBy { it.length }
```

### Counting items in a list after filter is applied

---

```java
// Java:
long count = items.stream().filter( item -> item.startsWith("t")).count();
```

```kotlin
// Kotlin:
val count = items.filter { it.startsWith('t') }.size
// but better to not filter, but count with a predicate
val count = items.count { it.startsWith('t') }
```

and on it goes... In all cases, no special fold, reduce, or other functionality was required to mimic `Stream.collect`. If you have further use cases, add them in comments and we can see!


## About laziness

If you want to lazy process a chain, you can convert to a `Sequence` using `asSequence()` before the chain. At the end of the chain of functions, you usually end up with a `Sequence` as well. Then you can use `toList()`, `toSet()`, `toMap()` or some other function to materialize the `Sequence` at the end.

```kotlin
// switch to and from lazy
val someList = items.asSequence().filter { ... }.take(10).map { ... }.toList()

// switch to lazy, but sorted() brings us out again at the end
val someList = items.asSequence().filter { ... }.take(10).map { ... }.sorted()
```


## Why are there no Types?!?

You will notice the Kotlin examples do not specify the types. This is because Kotlin has full type inference and is completely type safe at compile time. More so than Java because it also has nullable types and can help prevent the dreaded NPE. So this in Kotlin:

```kotlin
val someList = people.filter { it.age <= 30 }.map { it.name }
```

is the same as:

```kotlin
val someList: List<String> = people.filter { it.age <= 30 }.map { it.name }
```

Because Kotlin knows what `people` is, and that `people.age` is `Int` therefore the filter expression only allows comparison to an `Int`, and that `people.name` is a `String` therefore the `map` step produces a `List<String>` (readonly `List` of `String`).

Now, if `people` were possibly `null`, as-in a `List<People>?` then:

```kotlin
val someList = people?.filter { it.age <= 30 }?.map { it.name }
```

Returns a `List<String>?` that would need to be null checked (*or use one of the other Kotlin operators for nullable values, see this [Kotlin idiomatic way to deal with nullable values](https://stackoverflow.com/questions/34498562/in-kotlin-what-is-the-idiomatic-way-to-deal-with-nullable-values-referencing-o) and also [Idiomatic way of handling nullable or empty list in Kotlin](https://stackoverflow.com/questions/26341225/idiomatic-way-of-handling-nullable-or-empty-list-in-kotlin)*)


## See also:

- API Reference for [extension functions for Iterable](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/kotlin.-iterable/index.html)
- API reference for [extension functions for Array](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/kotlin.-array/index.html)
- API reference for [extension functions for List](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/kotlin.-list/index.html)
- API reference for [extension functions to Map](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/kotlin.-map/index.html)

- 참고
    - [https://stackoverflow.com/questions/34642254/what-java-8-stream-collect-equivalents-are-available-in-the-standard-kotlin-libr](https://stackoverflow.com/questions/34642254/what-java-8-stream-collect-equivalents-are-available-in-the-standard-kotlin-libr)


- [Notion link](https://www.notion.so/kotlin-collections-vs-Java-8-b2921865a61247ca94edf911b76ad4fb)
