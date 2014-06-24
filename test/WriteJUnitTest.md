## Write better JUnit tests

### Assertions

除了基本的 `assertTrue`, `assertEquals` 等外，可以利用`matcher`来做各种断言。 <br/>
首先需要 import 这些 matcher (包括junit的matchers 和 hamcrest 的，它们都包含在 junit 的包中):

```java
    import static org.hamcrest.CoreMatchers.*;
    import static org.junit.Assert.*;
    import static org.junit.matchers.JUnitMatchers.*
```

这里我使用` import static` 来简化实际的代码。有了这些，就可以写下直观的断言：

```java
    @Test
    public void myTest() {
        assertThat("hello", both(containsString("h")).and(containsString("o")));

        List<String> list = Arrays.asList("one", "two", "hello");

        assertThat(list, hasItems("one", "two"));
        assertThat(list, everyItem(containsString("o")));

        assertThat("actualResult", startsWith("actual"));
        assertThat(new Object(), not(sameInstance(new Object())));
    }

```
更多 matcher 请参考： https://github.com/junit-team/junit/wiki/Matchers-and-assertthat


### Exception Testing

在JUnit 3.x 中，expected exception 使用如下的 try/catch 来测试，这种方法不够直观：

```java
    @Test
    public void testException() {
        try {
            ((String) null).substring(1);
            fail("Expected an exception.");
        } catch (NullPointerException e) {
        }
    }

```

在JUnit 4 中，我们可以使用 `@Test(expected = Xxx.class)`:

```java
    @Test(expected = NullPointerException.class)
    public void nullTest() {
        ((String) null).substring(1);
    }
```

或者使用 ExpectedException Rule:

```java
    @Rule
    public ExpectedException thrown = ExpectedException.none();

    @Test
    public void shouldTestExceptionMessage() throws IndexOutOfBoundsException {
        thrown.expect(IndexOutOfBoundsException.class);
        thrown.expectMessage("Index: 0, Size: 0");

        new ArrayList<Object>().get(0); // execution will never get past this line
    }

```

### Parameterized tests
