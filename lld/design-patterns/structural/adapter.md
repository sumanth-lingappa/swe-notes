---
layout: default
title: Adapter
---

# Adapter: LLD quick reference

## Purpose

Adapter lets code that expects one interface use a class that has a different interface. It translates the request; the client can keep using the interface it already knows.

Example: our checkout expects `PaymentProcessor.pay(amountInCents)`, but a third-party gateway has `makePayment(amountInDollars)`.

```text
Client → PaymentProcessor interface ← PaymentAdapter → LegacyGateway
         pay(cents)                     converts cents    makePayment(dollars)
```

> Core idea: **Adapter translates one interface into another that the client expects.**

## C++: object adapter

```cpp
#include <iostream>
#include <string>

// The interface the application expects.
class PaymentProcessor {
public:
    virtual void pay(int amountInCents) = 0;
    virtual ~PaymentProcessor() = default;
};

// Existing third-party class. We cannot or do not want to change its API.
class LegacyGateway {
public:
    void makePayment(double amountInDollars) {
        std::cout << "Charged $" << amountInDollars << '\n';
    }
};

// Adapter implements the expected interface and wraps the incompatible class.
class PaymentAdapter : public PaymentProcessor {
public:
    explicit PaymentAdapter(LegacyGateway& gateway) : gateway_(gateway) {}

    void pay(int amountInCents) override {
        double dollars = amountInCents / 100.0; // Translate units.
        gateway_.makePayment(dollars);         // Forward to adaptee.
    }

private:
    LegacyGateway& gateway_; // Borrowed dependency; adapter does not own it.
};

int main() {
    LegacyGateway gateway;
    PaymentAdapter processor(gateway);
    PaymentProcessor& checkout = processor; // Client sees expected interface.
    checkout.pay(1299); // Charges $12.99
}
```

### Class relationship

```text
             implements
PaymentAdapter ───────────▷ PaymentProcessor
      │                         ▲
      │ wraps / delegates       │ client calls pay(cents)
      ▼                         │
LegacyGateway ◁────────────── Client
       makePayment(dollars)
```

## Python

```python
class PaymentProcessor:
    def pay(self, amount_in_cents):
        raise NotImplementedError


class LegacyGateway:
    def make_payment(self, amount_in_dollars):
        print(f"Charged ${amount_in_dollars:.2f}")


class PaymentAdapter(PaymentProcessor):
    def __init__(self, gateway):
        self._gateway = gateway  # Wrapped incompatible object.

    def pay(self, amount_in_cents):
        dollars = amount_in_cents / 100.0
        self._gateway.make_payment(dollars)  # Translate and delegate.


checkout_processor = PaymentAdapter(LegacyGateway())
checkout_processor.pay(1299)  # Charged $12.99
```

## Go

```go
package main

import "fmt"

// Interface expected by checkout.
type PaymentProcessor interface {
	Pay(amountInCents int)
}

// Existing API with a different method and unit.
type LegacyGateway struct{}

func (LegacyGateway) MakePayment(amountInDollars float64) {
	fmt.Printf("Charged $%.2f\n", amountInDollars)
}

// Adapter satisfies PaymentProcessor and wraps LegacyGateway.
type PaymentAdapter struct {
	gateway LegacyGateway
}

func (adapter PaymentAdapter) Pay(amountInCents int) {
	dollars := float64(amountInCents) / 100.0
	adapter.gateway.MakePayment(dollars)
}

func main() {
	var processor PaymentProcessor = PaymentAdapter{gateway: LegacyGateway{}}
	processor.Pay(1299) // Charged $12.99
}
```

## Common mistake and tradeoff

```cpp
// Poor: change every checkout caller to know the vendor API.
gateway.makePayment(amount / 100.0);
```

That spreads vendor details and unit conversion through clients. Adapter centralizes the translation. It adds a small wrapper class, which is useful when interfaces genuinely differ; avoid it when the APIs already match.

## Other examples

- Old logger with `writeLog(text)` adapted to an application `Logger.log(level, message)` interface.
- Celsius sensor adapted to an API that expects Fahrenheit.
- New storage API adapted to a legacy file-storage library.
