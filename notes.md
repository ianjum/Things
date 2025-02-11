Got it! I'll restructure this example so you can **introduce Mockito step by step** while explaining each concept. Here's the breakdown:

---

## **Step 1: Start with the Calculator Service Interface**
First, define the service that the `Calculator` class will depend on.

```java
public interface CalculatorService {
    int add(int a, int b);
    int subtract(int a, int b);
    int multiply(int a, int b);
    int divide(int a, int b);
}
```
- This is the dependency we will **mock** later.
- Explain to students that this represents an **external service** (e.g., API, database call) that we don't want to call in tests.

---

## **Step 2: Introduce the Calculator Class**
Next, create the `Calculator` class that uses the service.

```java
public class Calculator {
    private CalculatorService service;

    public Calculator(CalculatorService service) {
        this.service = service;
    }

    public int performAddition(int a, int b) {
        return service.add(a, b);
    }

    public int performSubtraction(int a, int b) {
        return service.subtract(a, b);
    }

    public int performMultiplication(int a, int b) {
        return service.multiply(a, b);
    }

    public int performDivision(int a, int b) {
        return service.divide(a, b);
    }
}
```
- Explain **dependency injection**: the service is passed via the constructor.
- The class **delegates operations** to the service.

---

## **Step 3: Write the First Test Without Mocks**
At this stage, students might **write direct tests** without using Mockito.

```java
import static org.junit.jupiter.api.Assertions.*;

import org.junit.jupiter.api.Test;

class CalculatorBasicTest {
    @Test
    void testAddition() {
        CalculatorService service = new CalculatorService() {
            @Override
            public int add(int a, int b) {
                return a + b;
            }
            @Override
            public int subtract(int a, int b) {
                return a - b;
            }
            @Override
            public int multiply(int a, int b) {
                return a * b;
            }
            @Override
            public int divide(int a, int b) {
                return a / b;
            }
        };

        Calculator calculator = new Calculator(service);
        assertEquals(30, calculator.performAddition(10, 20));
    }
}
```
- **This works**, but manually implementing `CalculatorService` is **not scalable**.

---

## **Step 4: Introduce Mockito with a Simple Mock**
Now, introduce **Mockito** to replace the manual service implementation.

### **Add Mockito Dependencies (Maven)**
Ensure `pom.xml` has:

```xml
<dependencies>
    <dependency>
        <groupId>org.mockito</groupId>
        <artifactId>mockito-core</artifactId>
        <version>5.7.0</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-api</artifactId>
        <version>5.9.2</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

---

### **Now, Modify the Test to Use a Mock**
```java
import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.*;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;

class CalculatorMockTest {
    private Calculator calculator;

    @Mock
    private CalculatorService service;

    @BeforeEach
    void setUp() {
        MockitoAnnotations.openMocks(this); // Initializes mocks
        calculator = new Calculator(service); // Inject mock service
    }

    @Test
    void testAdditionWithMock() {
        when(service.add(10, 20)).thenReturn(30); // Define behavior

        assertEquals(30, calculator.performAddition(10, 20));

        verify(service).add(10, 20); // Ensure method was called
    }
}
```
### **Key Points to Explain Here**
1. **`@Mock private CalculatorService service;`** → Creates a mock object.
2. **`MockitoAnnotations.openMocks(this);`** → Initializes mocks before each test.
3. **`when(service.add(10, 20)).thenReturn(30);`** → Defines mock behavior.
4. **`verify(service).add(10, 20);`** → Ensures method was called.

---

## **Step 5: Extend with More Tests**
Now, add more tests for subtraction, multiplication, and division.

```java
@Test
void testSubtractionWithMock() {
    when(service.subtract(30, 10)).thenReturn(20);
    assertEquals(20, calculator.performSubtraction(30, 10));
    verify(service).subtract(30, 10);
}

@Test
void testMultiplicationWithMock() {
    when(service.multiply(5, 4)).thenReturn(20);
    assertEquals(20, calculator.performMultiplication(5, 4));
    verify(service).multiply(5, 4);
}

@Test
void testDivisionWithMock() {
    when(service.divide(20, 5)).thenReturn(4);
    assertEquals(4, calculator.performDivision(20, 5));
    verify(service).divide(20, 5);
}
```
- Students can now **see the pattern** of mocking.

---

## **Step 6: Handle Exception Cases**
Introduce **exception handling** for division by zero.

```java
@Test
void testDivisionByZero() {
    when(service.divide(anyInt(), eq(0))).thenThrow(new ArithmeticException("Cannot divide by zero"));
    
    assertThrows(ArithmeticException.class, () -> calculator.performDivision(10, 0));
    
    verify(service).divide(10, 0);
}
```
- This demonstrates **mocking exceptions** and testing edge cases.

---

## **Step 7: Introduce Spy for Partial Mocks**
If you want to **keep real implementations but spy on behavior**, introduce a `spy`.

```java
import static org.mockito.Mockito.*;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.Spy;

class CalculatorSpyTest {
    @Spy
    private CalculatorService realService = new CalculatorService() {
        @Override
        public int add(int a, int b) {
            return a + b;
        }
        @Override
        public int subtract(int a, int b) {
            return a - b;
        }
        @Override
        public int multiply(int a, int b) {
            return a * b;
        }
        @Override
        public int divide(int a, int b) {
            return a / b;
        }
    };

    private Calculator calculator;

    @BeforeEach
    void setUp() {
        MockitoAnnotations.openMocks(this);
        calculator = new Calculator(realService); // Use spy
    }

    @Test
    void testSpy() {
        doReturn(100).when(realService).subtract(10, 5); // Mock only subtract

        assertEquals(100, calculator.performSubtraction(10, 5)); // Uses mock
        assertEquals(30, calculator.performAddition(10, 20)); // Calls real method

        verify(realService).subtract(10, 5);
    }
}
```
- This shows **mixing real and mocked behavior**.

---

## **Final Steps**
1. **Start with the basic class and interface.**
2. **Write tests without mocks.**
3. **Introduce Mockito for mocking.**
4. **Add more test cases.**
5. **Show handling exceptions.**
6. **Introduce spies.**

This **step-by-step** approach should make it much easier to explain each concept in class! Let me know if you'd like further refinements. 🚀
