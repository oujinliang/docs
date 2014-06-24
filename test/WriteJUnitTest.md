## Write better JUnit tests

### Assertions

除了基本的 `assertTrue`, `assertEquals` 等外，可以利用`matcher`来做各种断言。 首先需要 import 这些 matcher (包括junit的matchers 和 hamcrest 的，它们都包含在 junit 的包中):

    ```java
    import static org.hamcrest.CoreMatchers.allOf;
    import static org.hamcrest.CoreMatchers.anyOf;
    import static org.hamcrest.CoreMatchers.equalTo;
    import static org.hamcrest.CoreMatchers.not;
    import static org.hamcrest.CoreMatchers.sameInstance;
    import static org.hamcrest.CoreMatchers.startsWith;
    import static org.junit.Assert.assertThat;
    import static org.junit.matchers.JUnitMatchers.both;
    import static org.junit.matchers.JUnitMatchers.containsString;
    import static org.junit.matchers.JUnitMatchers.everyItem;
    import static org.junit.matchers.JUnitMatchers.hasItems;
     ```