# SOLID Violations in OrderProcessor - README

## Overview

This README analyzes how the original `OrderProcessor` class violates each of the SOLID principles and how to refactor the code to follow best design practices.

---

## Original Code (with Violations)

```java
public class OrderProcessor {
    public void processOrder(String itemType, String paymentMode) {
        // 1. SRP Violation: Too many responsibilities
        System.out.println("Validating order...");
        System.out.println("Saving order to database...");

        // 2. OCP Violation: Can't add new payment methods without modifying
        if (paymentMode.equals("credit")) {
            System.out.println("Processing credit card...");
        } else if (paymentMode.equals("paypal")) {
            System.out.println("Processing PayPal...");
        }

        // 3. LSP Violation: Would break if PaymentMode subclasses misbehaved

        // 4. ISP Violation: Suppose we had a giant interface OrderActions with many unused methods

        // 5. DIP Violation: Direct dependency on concrete logic
        EmailSender emailSender = new EmailSender();
        emailSender.sendConfirmation();
    }
}
```

---

## ✅ Violation Breakdown with Explanations

### 1. SRP – Single Responsibility Principle

**Violation:**

- `OrderProcessor` handles validation, DB persistence, payment logic, and notification.

**Fix:**

- Extract to:
  - `OrderValidator`
  - `OrderRepository`
  - `PaymentStrategy`
  - `Notifier`

---

### 2. OCP – Open/Closed Principle

**Violation:**

- Payment types are hardcoded. New payment modes require modifying `if-else` block.

**Fix:**

- Use Strategy Pattern:

```java
public interface PaymentStrategy {
    void pay();
}
```

Inject concrete strategy like `CreditCardPayment`, `PayPalPayment`, etc.

---

### 3. LSP – Liskov Substitution Principle

**Violation:**

- Suppose you create a subclass `CODPayment` that throws exception in `pay()`.
- Replacing `Payment p = new CODPayment()` breaks the expected behavior.

**Fix:**

- Ensure subclasses honor base class contract.
- Prefer interface segregation or appropriate abstraction.

---

### 4. ISP – Interface Segregation Principle

**Violation:**

- If `OrderActions` interface has methods like `returnOrder()` or `refund()` which `CODOrderService` doesn’t need, you're forced to implement unused methods.

**Fix:**

- Split interfaces:

```java
interface Validatable { void validate(); }
interface Refundable { void refund(); }
```

Only implement what is needed.

---

### 5. DIP – Dependency Inversion Principle

**Violation:**

- `OrderProcessor` directly depends on `EmailSender`, a concrete class.

**Fix:**

- Depend on abstraction:

```java
public interface Notifier {
    void sendConfirmation();
}
```

- Inject via constructor:

```java
public class OrderProcessor {
    private final Notifier notifier;

    public OrderProcessor(Notifier notifier) {
        this.notifier = notifier;
    }
}
```

---

## Summary for Interviews

> "In my initial OrderProcessor, I noticed it violated SRP by mixing validation, payment, persistence, and notification. It also violated OCP since new payment types required changing existing logic. LSP was broken because subclasses like CODPayment couldn’t substitute safely. ISP was violated by bloated interfaces, and DIP was broken by hardcoding dependencies. I refactored the code using strategy pattern, dependency injection, and interface segregation to make it modular, extensible, and testable."

---

Let me know if you'd like the refactored code version or visual class diagrams to complement this!

