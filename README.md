# SOLID

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

        // LSP Violation: Subclass breaks behavior
        PaymentMode payment = new CODPayment();
        payment.pay(); // This will throw exception

        // ISP Violation: Using interface with unneeded methods
        OrderActions actions = new DigitalOrderActions();
        actions.process();
        actions.returnOrder(); // Throws exception

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

## Refactored Code (SOLID Compliant)

```java
public class OrderProcessor {
    private final OrderValidator validator;
    private final OrderRepository repository;
    private final Notifier notifier;

    public OrderProcessor(OrderValidator validator, OrderRepository repository, Notifier notifier) {
        this.validator = validator;
        this.repository = repository;
        this.notifier = notifier;
    }

    public void processOrder(String itemType, PaymentStrategy paymentStrategy) {
        validator.validate(itemType);
        repository.save(itemType);
        paymentStrategy.pay();
        notifier.sendConfirmation();
    }
}
```

---


