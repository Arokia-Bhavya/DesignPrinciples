# SOLID Violations in OrderProcessor

## Overview

This README analyzes how the original `OrderProcessor` class violates each of the SOLID principles and how to refactor the code to follow best design practices.

---

## Original Code (with Violations)

```java
// Interface with too many responsibilities (ISP Violation)
interface OrderActions {
    void process();
    void refund();
    void returnOrder();
    void validate();
    void sendConfirmation();
}

// Abstract base class for payment (LSP Violation with CODPayment)
abstract class PaymentMode {
    abstract void pay();
}

// Concrete payment that works fine
class CreditCardPayment extends PaymentMode {
    @Override
    void pay() {
        System.out.println("Processing credit card...");
    }
}

// LSP Violation: Subclass violates the expected contract
class CODPayment extends PaymentMode {
    @Override
    void pay() {
        throw new UnsupportedOperationException("COD payment not supported");
    }
}

// ISP Violation: Forced to implement unsupported methods
class DigitalOrderActions implements OrderActions {
    public void process() {
        System.out.println("Processing digital order...");
    }

    public void refund() {
        System.out.println("Refunding digital order...");
    }

    public void returnOrder() {
        throw new UnsupportedOperationException("Return not supported for digital products");
    }

    public void validate() {
        System.out.println("Validating digital order...");
    }

    public void sendConfirmation() {
        System.out.println("Sending confirmation email...");
    }
}

// DIP Violation: Depends directly on concrete implementation
class EmailSender {
    public void sendConfirmation() {
        System.out.println("Sending confirmation email...");
    }
}

// SRP, OCP, LSP, ISP, DIP violations all in one place
public class OrderProcessor {
    public void processOrder(String itemType, String paymentMode) {
        // SRP Violation: Multiple responsibilities
        System.out.println("Validating order...");
        System.out.println("Saving order to database...");

        // OCP Violation: Hardcoded payment logic
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

        // DIP Violation: Directly dependent on a concrete class
        EmailSender emailSender = new EmailSender();
        emailSender.sendConfirmation();
    }
}```

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


public interface PaymentStrategy {
    void pay();
}


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



