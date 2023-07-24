# Assertions

### Assertion 메소드의 Static vs Non-Static 사용법

---

- PHPUnit의 assertion은 `PHPUnit\Framework\Assert`에서 구현된다. `PHPUnit\Framework\TestCase`는 `PHPUnit\Framework\Assert`에서 상속된다.
- assertion 메소드는 `static` 선언되며 `PHPUnit\Framework\TestCase`를 상속한 클래스 안의 모든 컨텍스트에서 예를 들어, `PHPUnit\Framework\Assert::assertTrue()`, `$this->assertTrue()`, `self::assertTrue()`로 호출할 수 있다. 심지어 `assertTrue()`같은 글로벌 함수 래퍼도 사용할 수 있다.
- PHPUnit을 처음 접하는 개발자들의 공통 질문은 예를 들어 `$this->assertTrue()`, `self::assertTrue()`가 assertion을 호출하는 "올바른 방법"인지 여부이다. 짧은 대답은: 올바른 방법은 없다는 것이다. 잘못된 방법도 없다. 개인 취향의 문제다.
- 많은 사람들은 테스트 메소드가 테스트 객체에 의해 호출되기 때문에 `$this->assertTrue()`를 쓰는게 옳은 것 같다고 생각한다. 사실은 assertion 메소드가 `static`으로 선언되어 있기 때문에 테스트 객체 범위 밖에서 사용될 수 있다. 마지막으로, 글로벌 함수 래퍼를 통해 개발자는 문자 수를 줄일 수 있다.(`$this->assertTrue()`, `self::assertTrue()` 대신 `assertTrue()`로)

### `assertArrayHasKey()`

---

- `assertArrayHasKey(mixed $key, array $array[, string $message = ''])`
- `$array`에 `$key`가 없는 경우 에러 `$message`를 보고한다.
- `assertArrayNotHasKey()`는 이 검증과 반대의 의미로, 같은 인수를 받는다.

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class ArrayHasKeyTest extends TestCase
{
    public function testFailure(): void
    {
        $this->assertArrayHasKey('foo', ['bar' => 'baz']);
    }
}
```

### `assertClassHasAttribute()`

---

- `assertClassHasAttribute(string $attributeName, string $className[, string $message = ''])`
- `$className::attributeName`이 존재하지 않을 경우, 에러 `$message`를 보고한다.
- `assertClassNotHasAttribute()`는 이 검증과 반대의 의미로, 같은 인수를 받는다.

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class ClassHasAttributeTest extends TestCase
{
    public function testFailure(): void
    {
        $this->assertClassHasAttribute('foo', stdClass::class);
    }
}
```

### `assertClassHasStaticAttribute()`

---

- `assertClassHasStaticAttribute(string $attributeName, string $className[, string $message = ''])`
- `$className::attributeName`이 존재하지 않을 경우, 에러 `$message`를 보고한다.
- `assertClassNotHasStaticAttribute()`는 이 검증과 반대의 의미로, 같은 인수를 받는다.

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class ClassHasStaticAttributeTest extends TestCase
{
    public function testFailure(): void
    {
        $this->assertClassHasStaticAttribute('foo', stdClass::class);
    }
}
```

### `assertContains()`

---

- `assertContains(mixed $needle, iterable $haystack[, string $message = ''])`
- `$needle`이 `$haystack`의 요소가 아닐 경우, 에러 `$message`를 보고한다.
- `assertNotContains()`는 이 검증과 반대의 의미로, 같은 인수를 받는다.

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class ContainsTest extends TestCase
{
    public function testFailure(): void
    {
        $this->assertContains(4, [1, 2, 3]);
    }
}
```

### `assertStringContainsString()`

---

- `assertStringContainsString(string $needle, string $haystack[, string $message = ''])`
- `$needle`이 `$haystack`문자열의 일부가 아닐 경우, 에러 `$message`를 보고한다.
- `assertStringNotContainsString()`는 이 검증과 반대의 의미로, 같은 인수를 받는다.

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class StringContainsStringTest extends TestCase
{
    public function testFailure(): void
    {
        $this->assertStringContainsString('foo', 'bar');
    }
}
```

### `assertStringContainsStringIgnoringCase()`

---

- `assertStringContainsStringIgnoringCase(string $needle, string $haystack[, string $message = ''])`
- `$needle`이 `$haystack`문자열의 일부가 아닐 경우, 에러 `$message`를 보고한다. 대소문자 차이는 무시된다.
- `assertStringNotContainsStringIgnoringCase()`는 이 검증과 반대의 의미로, 같은 인수를 받는다.

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class StringContainsStringIgnoringCaseTest extends TestCase
{
    public function testFailure(): void
    {
        $this->assertStringContainsStringIgnoringCase('foo', 'bar');
    }
}
```

### `assertContainsOnly()`

---

- `assertContainsOnly(string $type, iterable $haystack[, boolean $isNativeType = null, string $message = ''])`
- `$haystack`에 `$type` 이외의 형을 가지는 데이터가 들어있는 경우, 에러 `$message`를 보고한다. `$isNativeType`은 `$type`이 네이티브 PHP 유형인지 여부를 나타내는데 사용되는 플래그다.
- `assertNotContainsOnly()`는 이 검증과 반대의 의미로, 같은 인수를 받는다.

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class ContainsOnlyTest extends TestCase
{
    public function testFailure(): void
    {
        $this->assertContainsOnly('string', ['1', '2', 3]);
    }
}
```

### `assertContainsOnlyInstancesOf()`

---

- `assertContainsOnlyInstancesOf(string $classname, Traversable|array $haystack[, string $message = ''])`
- `$haystack`이 `$classname` 클래스의 인스턴스 이외를 포함한 경우, 에러 `$message`를 보고한다.

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class ContainsOnlyInstancesOfTest extends TestCase
{
    public function testFailure(): void
    {
        $this->assertContainsOnlyInstancesOf(
            Foo::class,
            [new Foo, new Bar, new Foo]
        );
    }
}
```

### `assertCount()`

---

- `assertCount($expectedCount, $haystack[, string $message = ''])`
- `$heystack`의 요소 숫자가 `$expectedCount`와 다른 경우, 에러 `$message`를 보고한다.
- `assertNotCount()`는 이 검증과 반대의 의미로, 같은 인수를 받는다.

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class CountTest extends TestCase
{
    public function testFailure(): void
    {
        $this->assertCount(0, ['foo']);
    }
}
```

### `assertDirectoryExists()`

---

- `assertDirectoryExists(string $directory[, string $message = ''])`
- `$directory`에서 지정한 디렉터리가 없는 경우, 에러 `$message`를 보고한다.
- `assertDirectoryDoesNotExist()`는 이 검증과 반대의 의미로, 같은 인수를 받는다.

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class DirectoryExistsTest extends TestCase
{
    public function testFailure(): void
    {
        $this->assertDirectoryExists('/path/to/directory');
    }
}
```

### `assertDirectoryIsReadable()`

---

- `assertDirectoryIsReadable(string $directory[, string $message = ''])`
- `$directory`에서 지정한 디렉터리가 디렉터리가 아니거나 읽을 수 없는 경우, 에러 `$message`를 보고한다.
- `assertDirectoryIsNotReadable()`는 이 검증과 반대의 의미로, 같은 인수를 받는다.

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class DirectoryIsReadableTest extends TestCase
{
    public function testFailure(): void
    {
        $this->assertDirectoryIsReadable('/path/to/directory');
    }
}
```

### `assertDirectoryIsWritable()`

---

- `assertDirectoryIsWritable(string $directory[, string $message = ''])`
- `$directory`에서 지정한 디렉터리가 디렉터리가 아니거나 쓸 수 없는 경우, 에러 `$message`를 보고한다.
- `assertDirectoryIsNotWritable()`는 이 검증과 반대의 의미로, 같은 인수를 받는다.

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class DirectoryIsWritableTest extends TestCase
{
    public function testFailure(): void
    {
        $this->assertDirectoryIsWritable('/path/to/directory');
    }
}
```

### `assertEmpty()`

---

- `assertEmpty(mixed $actual[, string $message = ''])`
- `$actual`이 비어 있지 않은 경우, 에러 `$message`를 보고한다.
- `assertNotEmpty()`는 이 검증과 반대의 의미로, 같은 인수를 받는다.

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class EmptyTest extends TestCase
{
    public function testFailure(): void
    {
        $this->assertEmpty(['foo']);
    }
}
```

### `assertEquals()`

---

- `assertEquals(mixed $expected, mixed $actual[, string $message = ''])`
- `$expected` 변수와 `$actual` 변수가 동일하지 않을 경우, 에러 `$message`를 보고한다.
- `$expected`와 `$actual` 두 객체의 속성 값이 동일하지 않을 경우, 에러 `$message`를 보고한다.
- `assertNotEquals()`는 이 검증과 반대의 의미로, 같은 인수를 받는다.

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class EqualsTest extends TestCase
{
    public function testFailure(): void
    {
        $this->assertEquals(1, 0);
    }

    public function testFailure2(): void
    {
        $this->assertEquals('bar', 'baz');
    }

    public function testFailure3(): void
    {
        $this->assertEquals("foo\nbar\nbaz\n", "foo\nbah\nbaz\n");
    }
}
```

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class EqualsTest extends TestCase
{
    public function testFailure(): void
    {
        $expected = new stdClass;
        $expected->foo = 'foo';
        $expected->bar = 'bar';

        $actual = new stdClass;
        $actual->foo = 'bar';
        $actual->baz = 'bar';

        $this->assertEquals($expected, $actual);
    }
}
```

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class EqualsTest extends TestCase
{
    public function testFailure(): void
    {
        $this->assertEquals(['a', 'b', 'c'], ['a', 'c', 'd']);
    }
}
```

### `assertEqualsCanonicalizing()`

---

- `assertEqualsCanonicalizing(mixed $expected, mixed $actual[, string $message = ''])`
- `$expected` 변수와 `$actual` 변수가 동일하지 않을 경우, 에러 `$message`를 보고한다.
- `$expected`, `$actual`의 내용은 비교되기 전에 표준화된다. 예를 들어, `$expected` 변수와 `$actual` 변수가 배열인 경우, 비교되기 전에 정렬된다. `$expected`, `$actual`이 객체인 경우, 각 객체는 모든 `private`, `protected`, `public` 속성을 포함하는 배열로 변환된다.
- `assertNotEqualsCanonicalizing()`는 이 검증과 반대의 의미로, 같은 인수를 받는다.

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class EqualsCanonicalizingTest extends TestCase
{
    public function testFailure()
    {
        $this->assertEqualsCanonicalizing([3, 2, 1], [2, 3, 0, 1]);
    }
}
```

### `assertEqualsIgnoringCase()`

---

- `assertEqualsIgnoringCase(mixed $expected, mixed $actual[, string $message = ''])`
- `$expected` 변수와 `$actual` 변수가 동일하지 않을 경우, 에러 `$message`를 보고한다. 대소문자 차이는 무시된다.
- `assertNotEqualsIgnoringCase()`는 이 검증과 반대의 의미로, 같은 인수를 받는다.

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class EqualsIgnoringCaseTest extends TestCase
{
    public function testFailure()
    {
        $this->assertEqualsIgnoringCase('foo', 'BAR');
    }
}
```

### `assertEqualsWithDelta()`

---

- `assertEqualsWithDelta(mixed $expected, mixed $actual, float $delta[, string $message = ''])`
- `$expected`와 `$actual`의 절대 차이가 `$delta`보다 클 경우, 에러 `$message`를 보고한다.
- `assertNotEqualsWithDelta()`는 이 검증과 반대의 의미로, 같은 인수를 받는다.

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class EqualsWithDeltaTest extends TestCase
{
    public function testFailure()
    {
        $this->assertEqualsWithDelta(1.0, 1.5, 0.1);
    }
}
```

### `assertObjectEquals()`

---

- `assertObjectEquals(object $expected, object $actual, string $method = 'equals', string $message = ''])`
- `$actual->$method($expected)`에 따라 `$actual`이 `$expected`와 같지 않은 경우, 에러 `$message`를 보고한다.
- 커스텀 comparator를 등록하지 않고 `assertEquals()`를 `object`에 쓰는 것은 좋지 않다. 보통 객체들은 `equals(self $other): bool`메소드를 갖고 있다. `assertObjectEquals()`를 사용하면 다음과 같은 일반적인 사용 사례에 편리하게 객체를 비교할 수 있다:

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class SomethingThatUsesEmailTest extends TestCase
{
    public function testSomething(): void
    {
        $a = new Email('user@example.org');
        $b = new Email('user@example.org');
        $c = new Email('user@example.com');

        // This passes
        $this->assertObjectEquals($a, $b);

        // This fails
        $this->assertObjectEquals($a, $c);
    }
}
```

```php
<?php declare(strict_types=1);
final class Email
{
    private string $email;

    public function __construct(string $email)
    {
        $this->ensureIsValidEmail($email);

        $this->email = $email;
    }

    public function asString(): string
    {
        return $this->email;
    }

    public function equals(self $other): bool
    {
        return $this->asString() === $other->asString();
    }

    private function ensureIsValidEmail(string $email): void
    {
        // ...
    }
}
```

- 이름이 `$method`인 메소드가 `$actual` 객체에 있어야 한다.
- 메소드는 정확히 하나의 인수를 가져야 한다.
- 각 매개 변수에는 타입이 선언되어야 한다.
- `$expected` 객체는 이 선언된 타입과 호환되어야 한다.
- 메소드는 `bool` 반환 타입이 선언되어야 한다.

→ 위에서 언급된 가정 중 하나라도 충족되지 않거나 `$actual->$method($expected)`가 거짓을 반환하면 assertion이 실패한다.

### `assertFalse()`

---

- `assertFalse(bool $condition[, string $message = ''])`
- `$condition`이 참인 경우, 에러 `$message`를 보고한다.
- `assertNotFalse()`는 이 검증과 반대의 의미로, 같은 인수를 받는다.

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class FalseTest extends TestCase
{
    public function testFailure(): void
    {
        $this->assertFalse(true);
    }
}
```

### `assertFileEquals()`

---

- `assertFileEquals(string $expected, string $actual[, string $message = ''])`
- `$expected`로 지정한 파일과 `$actual`로 지정한 파일의 내용이 다른 경우, 에러 `$message`를 보고한다.
- `assertFileNotEquals()`는 이 검증과 반대 의미로, 같은 인수를 받는다.

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class FileEqualsTest extends TestCase
{
    public function testFailure(): void
    {
        $this->assertFileEquals('/home/sb/expected', '/home/sb/actual');
    }
}
```

### `assertFileExists()`

---

- `assertFileExists(string $filename[, string $message = ''])`
- 파일 `$filename`이 존재하지 않을 경우, 에러 `$message`를 보고한다.
- `assertFileNotExists()`는 이 검증과 반대 의미로, 같은 인수를 받는다.

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class FileExistsTest extends TestCase
{
    public function testFailure(): void
    {
        $this->assertFileExists('/path/to/file');
    }
}
```

### `assertFileIsReadable()`

---

- `assertFileIsReadable(string $filename[, string $message = ''])`
- `$filename`으로 지정된 파일이 파일이 아니거나 읽을 수 없는 경우, 에러 `$message`를 보고한다.
- `assertFileIsNotReadable()`는 이 검증과 반대 의미로, 같은 인수를 받는다.

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class FileIsReadableTest extends TestCase
{
    public function testFailure(): void
    {
        $this->assertFileIsReadable('/path/to/file');
    }
}
```

### `assertFileIsWritable()`

---

- `assertFileIsWritable(string $filename[, string $message = ''])`
- `$filename`으로 지정된 파일이 파일이 아니거나 쓸 수 없는 경우, 에러 `$message`를 보고한다.
- `assertFileIsNotWritable()`는 이 검증과 반대 의미로, 같은 인수를 받는다.

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class FileIsWritableTest extends TestCase
{
    public function testFailure(): void
    {
        $this->assertFileIsWritable('/path/to/file');
    }
}
```

### `assertGreaterThan()`

---

- `assertGreaterThan(mixed $expected, mixed $actual[, string $message = ''])`
- `$actual`값이 `$expected`보다 크지 않을 경우, 에러 `$message`를 보고한다.

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class GreaterThanTest extends TestCase
{
    public function testFailure(): void
    {
        $this->assertGreaterThan(2, 1);
    }
}
```

### `assertGreaterThanOrEqual()`

---

- `assertGreaterThanOrEqual(mixed $expected, mixed $actual[, string $message = ''])`
- `$actual`값이 `$expected`보다 크거나 같지 않은 경우, 에러 `$message`를 보고한다.

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class GreatThanOrEqualTest extends TestCase
{
    public function testFailure(): void
    {
        $this->assertGreaterThanOrEqual(2, 1);
    }
}
```

### `assertInfinite()`

---

- `assertInfinite(mixed $variable[, string $message = ''])`
- `$variable`이 `INF`가 아닌 경우, 에러 `$message`를 보고한다.
- `assertFinite()`는 이 검증과 반대 의미로, 같은 인수를 받는다.

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class InfiniteTest extends TestCase
{
    public function testFailure(): void
    {
        $this->assertInfinite(1);
    }
}
```

### `assertInstanceOf()`

---

- `assertInstanceOf($expected, $actual[, $message = ''])`
- `$actual`이 `$expected`의 인스턴스가 아닐 경우, 에러 `$message`를 보고한다.
- `assertNotInstanceOf()`는 이 검증과 반대 의미로, 같은 인수를 받는다.

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class InstanceOfTest extends TestCase
{
    public function testFailure(): void
    {
        $this->assertInstanceOf(RuntimeException::class, new Exception);
    }
}
```

### `assertIsArray()`

---

- `assertIsArray($actual[, $message = ''])`
- `$actual`이 배열 타입이 아닌 경우, 에러 `$message`를 보고한다.
- `assertIsNotArray()`는 이 검증과 반대 의미로, 같은 인수를 받는다.

```php
<?php
use PHPUnit\Framework\TestCase;

class ArrayTest extends TestCase
{
    public function testFailure()
    {
        $this->assertIsArray(null);
    }
}
```

### `assertIsBool()`

---

- `assertIsBool($actual[, $message = ''])`
- `$actual`이 `bool` 타입이 아닌 경우, 에러 `$message`를 보고한다.
- `assertIsNotBool()`는 이 검증과 반대 의미로, 같은 인수를 받는다.

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class BoolTest extends TestCase
{
    public function testFailure(): void
    {
        $this->assertIsBool(null);
    }
}
```

### `assertIsCallable()`

---

- `assertIsCallable($actual[, $message = ''])`
- `$actual`이 `callable` 타입이 아닌 경우, 에러 `$message`를 보고한다.
- `assertIsNotCallable()`는 이 검증과 반대 의미로, 같은 인수를 받는다.

```php
<?php
use PHPUnit\Framework\TestCase;

class CallableTest extends TestCase
{
    public function testFailure()
    {
        $this->assertIsCallable(null);
    }
}
```

### `assertIsFloat()`

---

- `assertIsFloat($actual[, $message = ''])`
- `$actual`이 `float` 타입이 아닌 경우, 에러 `$message`를 보고한다.
- `assertIsNotFloat()`는 이 검증과 반대 의미로, 같은 인수를 받는다.

```php
<?php
use PHPUnit\Framework\TestCase;

class FloatTest extends TestCase
{
    public function testFailure()
    {
        $this->assertIsFloat(null);
    }
}
```

### `assertIsInt()`

---

- `assertIsInt($actual[, $message = ''])`
- `$actual`이 `int` 타입이 아닌 경우, 에러 `$message`를 보고한다.
- `assertIsNotInt()`는 이 검증과 반대 의미로, 같은 인수를 받는다.

```php
<?php
use PHPUnit\Framework\TestCase;

class IntTest extends TestCase
{
    public function testFailure()
    {
        $this->assertIsInt(null);
    }
}
```

### `assertIsIterable()`

---

- `assertIsIterable($actual[, $message = ''])`
- `$actual`이 `iterable` 타입이 아닌 경우, 에러 `$message`를 보고한다.
- `assertIsNotIterable()`는 이 검증과 반대 의미로, 같은 인수를 받는다.

```php
<?php
use PHPUnit\Framework\TestCase;

class IterableTest extends TestCase
{
    public function testFailure()
    {
        $this->assertIsIterable(null);
    }
}
```

### `assertIsNumeric()`

---

- `assertIsNumeric($actual[, $message = ''])`
- `$actual`이 `numeric` 타입이 아닌 경우, 에러 `$message`를 보고한다.
- `assertIsNotNumeric()`는 이 검증과 반대 의미로, 같은 인수를 받는다.

```php
<?php
use PHPUnit\Framework\TestCase;

class NumericTest extends TestCase
{
    public function testFailure()
    {
        $this->assertIsNumeric(null);
    }
}
```

### `assertIsObject()`

---

- `assertIsObject($actual[, $message = ''])`
- `$actual`이 `object` 타입이 아닌 경우, 에러 `$message`를 보고한다.
- `assertIsNotObject()`는 이 검증과 반대 의미로, 같은 인수를 받는다.

```php
<?php
use PHPUnit\Framework\TestCase;

class ObjectTest extends TestCase
{
    public function testFailure()
    {
        $this->assertIsObject(null);
    }
}
```

### `assertIsResource()`

---

- `assertIsResource($actual[, $message = ''])`
- `$actual`이 `resource` 타입이 아닌 경우, 에러 `$message`를 보고한다.
- `assertIsNotResource()`는 이 검증과 반대 의미로, 같은 인수를 받는다.

```php
<?php
use PHPUnit\Framework\TestCase;

class ResourceTest extends TestCase
{
    public function testFailure()
    {
        $this->assertIsResource(null);
    }
}
```

### `assertIsScalar()`

---

- `assertIsScalar($actual[, $message = ''])`
- `$actual`이 `scalar` 타입이 아닌 경우, 에러 `$message`를 보고한다.
- `assertIsNotScalar()`는 이 검증과 반대 의미로, 같은 인수를 받는다.

```php
<?php
use PHPUnit\Framework\TestCase;

class ScalarTest extends TestCase
{
    public function testFailure()
    {
        $this->assertIsScalar(null);
    }
}
```

### `assertIsString()`

---

- `assertIsString($actual[, $message = ''])`
- `$actual`이 `string` 타입이 아닌 경우, 에러 `$message`를 보고한다.
- `assertIsNotString()`는 이 검증과 반대 의미로, 같은 인수를 받는다.

```php
<?php
use PHPUnit\Framework\TestCase;

class StringTest extends TestCase
{
    public function testFailure()
    {
        $this->assertIsString(null);
    }
}
```

### `assertIsReadable()`

---

- `assertIsReadable(string $filename[, string $message = ''])`
- `$filename`으로 지정된 파일 또는 디렉터리를 읽을 수 없는 경우, 에러 `$message`를 보고한다.
- `assertIsNotReadable()`는 이 검증과 반대 의미로, 같은 인수를 받는다.

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class IsReadableTest extends TestCase
{
    public function testFailure(): void
    {
        $this->assertIsReadable('/path/to/unreadable');
    }
}
```

### `assertIsWritable()`

---

- `assertIsWritable(string $filename[, string $message = ''])`
- `$filename`으로 지정된 파일 또는 디렉터리를 쓸 수 없는 경우, 에러 `$message`를 보고한다.
- `assertIsNotWritable()`는 이 검증과 반대 의미로, 같은 인수를 받는다.

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class IsWritableTest extends TestCase
{
    public function testFailure(): void
    {
        $this->assertIsWritable('/path/to/unwritable');
    }
}
```

### `assertJsonFileEqualsJsonFile()`

---

- `assertJsonFileEqualsJsonFile(mixed $expectedFile, mixed $actualFile[, string $message = ''])`
- `$actualFile`의 값이 `$expectedFile`의 값과 일치하지 않는 경우, 에러 `$message`를 보고한다.

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class JsonFileEqualsJsonFileTest extends TestCase
{
    public function testFailure(): void
    {
        $this->assertJsonFileEqualsJsonFile(
          'path/to/fixture/file', 'path/to/actual/file');
    }
}
```

### `assertJsonStringEqualsJsonFile()`

---

- `assertJsonStringEqualsJsonFile(mixed $expectedFile, mixed $actualJson[, string $message = ''])`
- `$actualJson`의 값이 `$expectedFile`의 값과 일치하지 않는 경우, 에러 `$message`를 보고한다.

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class JsonStringEqualsJsonFileTest extends TestCase
{
    public function testFailure(): void
    {
        $this->assertJsonStringEqualsJsonFile(
            'path/to/fixture/file', json_encode(['Mascot' => 'ux'])
        );
    }
}
```

### `assertJsonStringEqualsJsonString()`

---

- `assertJsonStringEqualsJsonString(mixed $expectedJson, mixed $actualJson[, string $message = ''])`
- `$actualJson`의 값이 `$expectedJson`의 값과 일치하지 않는 경우, 에러 `$message`를 보고한다.

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class JsonStringEqualsJsonStringTest extends TestCase
{
    public function testFailure(): void
    {
        $this->assertJsonStringEqualsJsonString(
            json_encode(['Mascot' => 'Tux']),
            json_encode(['Mascot' => 'ux'])
        );
    }
}
```

### `assertLessThan()`

---

- `assertLessThan(mixed $expected, mixed $actual[, string $message = ''])`
- `$actual`의 값이 `$expected`의 값보다 작지 않은 경우, 에러 `$message`를 보고한다.

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class LessThanTest extends TestCase
{
    public function testFailure(): void
    {
        $this->assertLessThan(1, 2);
    }
}
```

### `assertLessThanOrEqual()`

---

- `assertLessThanOrEqual(mixed $expected, mixed $actual[, string $message = ''])`
- `$actual`의 값이 `$expected`의 값 이하가 아닌 경우 (보다 큰 경우), 에러 `$message`를 보고한다.

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class LessThanOrEqualTest extends TestCase
{
    public function testFailure(): void
    {
        $this->assertLessThanOrEqual(1, 2);
    }
}
```

### `assertNan()`

---

- `assertNan(mixed $variable[, string $message = ''])`
- `$variable`이 `NAN`이 아닌 경우, 에러 `$message`를 보고한다.

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class NanTest extends TestCase
{
    public function testFailure(): void
    {
        $this->assertNan(1);
    }
}
```

### `assertNull()`

---

- `assertNull(mixed $variable[, string $message = ''])`
- `$variable`이 `null`이 아닌 경우, 에러 `$message`를 보고한다.
- `assertNotNull()`는 이 검증과 반대 의미로, 같은 인수를 받는다.

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class NullTest extends TestCase
{
    public function testFailure(): void
    {
        $this->assertNull('foo');
    }
}
```

### `assertObjectHasAttribute()`

---

- `assertObjectHasAttribute(string $attributeName, object $object[, string $message = ''])`
- `$object->attributeName`이 존재하지 않는 경우, 에러 `$message`를 보고한다.
- `assertObjectNotHasAttribute()`는 이 검증과 반대 의미로, 같은 인수를 받는다.

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class ObjectHasAttributeTest extends TestCase
{
    public function testFailure(): void
    {
        $this->assertObjectHasAttribute('foo', new stdClass);
    }
}
```

### `assertMatchesRegularExpression()`

---

- `assertMatchesRegularExpression(string $pattern, string $string[, string $message = ''])`
- `$string`이 `$pattern` 정규식과 일치하지 않을 경우, 에러 `$message`를 보고한다.
- `assertDoesNotMatchRegularExpression()`는 이 검증과 반대 의미로, 같은 인수를 받는다.

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class RegExpTest extends TestCase
{
    public function testFailure(): void
    {
        $this->assertMatchesRegularExpression('/foo/', 'bar');
    }
}
```

### `assertStringMatchesFormat()`

---

- `assertStringMatchesFormat(string $format, string $string[, string $message = ''])`
- `$string`이 `$format` 문자열과 일치하지 않을 경우, 에러 `$message`를 보고한다.
- `assertStringNotMatchesFormat()`는 이 검증과 반대 의미로, 같은 인수를 받는다.

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class StringMatchesFormatTest extends TestCase
{
    public function testFailure(): void
    {
        $this->assertStringMatchesFormat('%i', 'foo');
    }
}
```

- format 문자열에는 다음이 포함될 수 있다:
    - `%e`: 디렉토리 구분자를 나타낸다, for example `/` on Linux.
    - `%s`: One or more of anything (character or white space) except the end of line character.
    - `%S`: Zero or more of anything (character or white space) except the end of line character.
    - `%a`: One or more of anything (character or white space) including the end of line character.
    - `%A`: Zero or more of anything (character or white space) including the end of line character.
    - `%w`: Zero or more white space characters.
    - `%i`: A signed integer value, for example `+3142`, `3142`.
    - `%d`: An unsigned integer value, for example `123456`.
    - `%x`: One or more hexadecimal character. That is, characters in the range `0-9`, `a-f`, `A-F`.
    - `%f`: A floating point number, for example: `3.142`, `3.142`, `3.142E-10`, `3.142e+10`.
    - `%c`: A single character of any sort.
    - `%%`: A literal percent character: `%`.

### `assertStringMatchesFormatFile()`

---

- `assertStringMatchesFormatFile(string $formatFile, string $string[, string $message = ''])`
- `$string`이 `$formatFile`의 내용과 일치하지 않을 경우, 에러 `$message`를 보고한다.
- `assertStringNotMatchesFormatFile()`는 이 검증과 반대 의미로, 같은 인수를 받는다.

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class StringMatchesFormatFileTest extends TestCase
{
    public function testFailure(): void
    {
        $this->assertStringMatchesFormatFile('/path/to/expected.txt', 'foo');
    }
}
```

### `assertSame()`

---

- `assertSame(mixed $expected, mixed $actual[, string $message = ''])`
- `$expected` 및 `$actual` 두 변수의 타입과 값이 동일하지 않은 경우, 에러 `$message`를 보고한다.
- `$expected` 및 `$actual` 두 변수가 동일한 객체를 참조하지 않는 경우, 에러 `$message`를 보고한다.
- `assertNotSame()`는 이 검증과 반대 의미로, 같은 인수를 받는다.

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class SameTest extends TestCase
{
    public function testFailure(): void
    {
        $this->assertSame('2204', 2204);
    }
}
```

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class SameTest extends TestCase
{
    public function testFailure(): void
    {
        $this->assertSame(new stdClass, new stdClass);
    }
}
```

### `assertSameSize()`

---

- `assertSameSize($expected, $actual, string $message = '')`
- `$actual`과 `$expected`의 크기가 동일하지 않을 경우, 에러 `$message`를 보고한다.
- `assertNotSameSize()`는 이 검증과 반대 의미로, 같은 인수를 받는다.

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class SameSizeTest extends TestCase
{
    public function testFailure(): void
    {
        $this->assertSameSize([1, 2], [1]);
    }
}
```

### `assertStringEndsWith()`

---

- `assertStringEndsWith(string $suffix, string $string[, string $message = ''])`
- `$string`이 `$suffix`로 끝나지 않는 경우, 에러 `$message`를 보고한다.
- `assertStringEndsNotWith()`는 이 검증과 반대 의미로, 같은 인수를 받는다.

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class StringEndsWithTest extends TestCase
{
    public function testFailure(): void
    {
        $this->assertStringEndsWith('suffix', 'foo');
    }
}
```

### `assertStringEqualsFile()`

---

- `assertStringEqualsFile(string $expectedFile, string $actualString[, string $message = ''])`
- `$expectedFile`로 지정된 파일에 `$actualString`이 내용으로 포함되지 않은 경우, 에러 `$message`를 보고한다.
- `assertStringNotEqualsFile()`는 이 검증과 반대 의미로, 같은 인수를 받는다.

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class StringEqualsFileTest extends TestCase
{
    public function testFailure(): void
    {
        $this->assertStringEqualsFile('/home/sb/expected', 'actual');
    }
}
```

### `assertStringStartsWith()`

---

- `assertStringStartsWith(string $prefix, string $string[, string $message = ''])`
- `$string`이 `$prefix`로 시작하지 않는 경우, 에러 `$message`를 보고한다.
- `assertStringStartsNotWith()`는 이 검증과 반대 의미로, 같은 인수를 받는다.

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class StringStartsWithTest extends TestCase
{
    public function testFailure(): void
    {
        $this->assertStringStartsWith('prefix', 'foo');
    }
}
```

### `assertThat()`

---

- `PHPUnit\Framework\Constraint` 클래스를 사용하여 보다 복잡한 `assertion`을 공식화할 수 있다. `assertThat()` 메소드를 사용하여 평가될 수 있다.
- `assertThat(mixed $value, PHPUnit\Framework\Constraint $constraint[, $message = ''])`
- `$value`가 `$constraint`와 일치하지 않을 경우, 에러 `$message`를 보고한다.
- ex> `logicalNot()`, `equalTo()`로 `assertNotEquals()` 표현
    
    ```php
    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;
    
    final class BiscuitTest extends TestCase
    {
        public function testEquals(): void
        {
            $theBiscuit = new Biscuit('Ginger');
            $myBiscuit  = new Biscuit('Ginger');
    
            $this->assertThat(
              $theBiscuit,
              $this->logicalNot(
                $this->equalTo($myBiscuit)
              )
            );
        }
    }
    ```
    

- `PHPUnit\Framework\Constraint`
    
     -
    | 제약 | 의미 |
    | --- | --- |
    | PHPUnit\Framework\Constraint\IsAnything anything() | 모든 입력 값을 허용하는 제약 조건 |
    | PHPUnit\Framework\Constraint\ArrayHasKey arrayHasKey(mixed $key) | 배열에 지정된 키가 있다고 주장하는 제약 조건 |
    | PHPUnit\Framework\Constraint\TraversableContains contains(mixed $value) | Iterator 인터페이스를 구현하는 배열 또는 객체에 지정된 값이 포함되어 있다고 주장하는 제약 조건 |
    | PHPUnit\Framework\Constraint\TraversableContainsOnly containsOnly(string $type) | Iterator 인터페이스를 구현하는 배열 또는 객체에 지정된 타입의 값만 포함되어 있다고 주장하는 제약 조건 |
    | PHPUnit\Framework\Constraint\TraversableContainsOnly containsOnlyInstancesOf(string $classname) | Iterator 인터페이스를 구현하는 배열 또는 객체에 지정된 클래스 이름의 인스턴스만 포함되어 있다고 주장하는 제약 조건 |
    | PHPUnit\Framework\Constraint\IsEqual equalTo($value, $delta = 0, $maxDepth = 10) | 한 값이 다른 값과 동일한지 확인하는 제약 조건 |
    | PHPUnit\Framework\Constraint\DirectoryExists directoryExists() | 디렉터리가 있는지 확인하는 제약 조건 |
    | PHPUnit\Framework\Constraint\FileExists fileExists() | 파일(이름)이 있는지 확인하는 제약 조건 |
    | PHPUnit\Framework\Constraint\IsReadable isReadable() | 파일(이름)을 읽을 수 있는지 확인하는 제약 조건 |
    | PHPUnit\Framework\Constraint\IsWritable isWritable() | 파일(이름)이 쓰기 가능한지 확인하는 제약 조건 |
    | PHPUnit\Framework\Constraint\GreaterThan greaterThan(mixed $value) | 값이 지정된 값보다 크다고 주장하는 제약 조건 |
    | PHPUnit\Framework\Constraint\LogicalOr greaterThanOrEqual(mixed $value) | 값이 지정된 값보다 크거나 같다고 주장하는 제약 조건 |
    | PHPUnit\Framework\Constraint\ClassHasAttribute classHasAttribute(string $attributeName) | 클래스에 지정된 속성이 있다고 주장하는 제약 조건 |
    | PHPUnit\Framework\Constraint\ClassHasStaticAttribute classHasStaticAttribute(string $attributeName) | 클래스에 지정된 정적 속성이 있다고 주장하는 제약 조건 |
    | PHPUnit\Framework\Constraint\ObjectHasAttribute objectHasAttribute(string $attributeName) | 객체에 지정된 속성이 있다고 주장하는 제약 조건 |
    | PHPUnit\Framework\Constraint\IsIdentical identicalTo(mixed $value) | 한 값이 다른 값과 동일하다고 주장하는 제약 조건 |
    | PHPUnit\Framework\Constraint\IsFalse isFalse() | 값이 거짓이라고 주장하는 제약 조건 |
    | PHPUnit\Framework\Constraint\IsInstanceOf isInstanceOf(string $className) | 객체가 지정된 클래스의 인스턴스라고 주장하는 제약 조건 |
    | PHPUnit\Framework\Constraint\IsNull isNull() | 값이 null이라고 주장하는 제약 조건 |
    | PHPUnit\Framework\Constraint\IsTrue isTrue() | 값이 참이라고 주장하는 제약 조건 |
    | PHPUnit\Framework\Constraint\IsType isType(string $type) | 값이 지정된 타입이라고 주장하는 제약 조건 |
    | PHPUnit\Framework\Constraint\LessThan lessThan(mixed $value) | 값이 지정된 값보다 작다고 주장하는 제약 조건 |
    | PHPUnit\Framework\Constraint\LogicalOr lessThanOrEqual(mixed $value) | 값이 지정된 값보다 작거나 같다고 주장하는 제약 조건 |
    | logicalAnd() | 논리적 AND. |
    | logicalNot(PHPUnit\Framework\Constraint $constraint) | 논리적 NOT. |
    | logicalOr() | 논리적 OR. |
    | logicalXor() | 논리적 XOR. |
    | PHPUnit\Framework\Constraint\PCREMatch matchesRegularExpression(string $pattern) | 문자열이 정규식과 일치한다고 주장하는 제약 조건 |
    | PHPUnit\Framework\Constraint\StringContains stringContains(string $string, bool $case) | 문자열에 지정된 문자열이 포함되어 있다고 주장하는 제약 조건 |
    | PHPUnit\Framework\Constraint\StringEndsWith stringEndsWith(string $suffix) | 문자열이 지정된 접미사로 끝나도록 하는 제약 조건 |
    | PHPUnit\Framework\Constraint\StringStartsWith stringStartsWith(string $prefix) | 문자열이 지정된 접두사로 시작한다고 주장하는 제약 조건 |

### `assertTrue()`

---

- `assertTrue(bool $condition[, string $message = ''])`
- `$condition`이 `false`인 경우, 에러 `$message`를 보고한다.
- `assertNotTrue()`는 이 검증과 반대 의미로, 같은 인수를 받는다.

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class TrueTest extends TestCase
{
    public function testFailure(): void
    {
        $this->assertTrue(false);
    }
}
```

### `assertXmlFileEqualsXmlFile()`

---

- `assertXmlFileEqualsXmlFile(string $expectedFile, string $actualFile[, string $message = ''])`
- `$actualFile`의 XML 문서가 `$expectedFile`의 XML 문서와 동일하지 않을 경우, 에러 `$message`를 보고한다.
- `assertXmlFileNotEqualsXmlFile()`는 이 검증과 반대 의미로, 같은 인수를 받는다.

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class XmlFileEqualsXmlFileTest extends TestCase
{
    public function testFailure(): void
    {
        $this->assertXmlFileEqualsXmlFile(
          '/home/sb/expected.xml', '/home/sb/actual.xml');
    }
}
```

### `assertXmlStringEqualsXmlFile()`

---

- `assertXmlStringEqualsXmlFile(string $expectedFile, string $actualXml[, string $message = ''])`
- `$actualXml`의 XML 문서가 `$expectedFile`의 XML 문서와 동일하지 않을 경우, 에러 `$message`를 보고한다.
- `assertXmlStringNotEqualsXmlFile()`는 이 검증과 반대 의미로, 같은 인수를 받는다.

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class XmlStringEqualsXmlFileTest extends TestCase
{
    public function testFailure(): void
    {
        $this->assertXmlStringEqualsXmlFile(
          '/home/sb/expected.xml', '<foo><baz/></foo>');
    }
}
```

### `assertXmlStringEqualsXmlString()`

---

- `assertXmlStringEqualsXmlString(string $expectedXml, string $actualXml[, string $message = ''])`
- `$actualXml`의 XML 문서가 `$expectedXml`의 XML 문서와 동일하지 않을 경우, 에러 `$message`를 보고한다.
- `assertXmlStringNotEqualsXmlString()`는 이 검증과 반대 의미로, 같은 인수를 받는다.

```php
<?php declare(strict_types=1);
use PHPUnit\Framework\TestCase;

final class XmlStringEqualsXmlStringTest extends TestCase
{
    public function testFailure(): void
    {
        $this->assertXmlStringEqualsXmlString(
          '<foo><bar/></foo>', '<foo><baz/></foo>');
    }
}
```

- 참고
    - [https://docs.phpunit.de/en/9.6/writing-tests-for-phpunit.html](https://docs.phpunit.de/en/9.6/assertions.html)