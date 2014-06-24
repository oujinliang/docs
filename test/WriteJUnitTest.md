## Write better JUnit tests

### Assertions

除了基本的 `assertTrue`, `assertEquals` 等外，可以利用`matcher`来做各种断言。 首先需要 import 这些 matcher (包括junit的matchers 和 hamcrest 的，它们都包含在 junit 的包中):

```java
    import static org.hamcrest.CoreMatchers.*;
    import static org.junit.Assert.*;
    import static org.junit.matchers.JUnitMatchers.*
```    
