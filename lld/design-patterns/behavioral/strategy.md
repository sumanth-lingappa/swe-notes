---
layout: default
title: Strategy
---

# Strategy: LLD quick reference

## Purpose

Strategy puts interchangeable algorithms behind one interface. A context uses the interface; the caller supplies or selects the algorithm. This replaces growing conditionals when behaviors need to vary.

Example: a checkout can calculate payment using card, UPI, or bank transfer strategies.

```text
Checkout ──has-a──► PaymentStrategy interface
                         ▲
              ┌──────────┼──────────┐
              │          │          │
          CardPay      UpiPay    BankPay
```

> Core idea: **Encapsulate a family of algorithms and make them interchangeable.**

## C++

```cpp
#include <iostream>
#include <memory>
#include <string>
#include <utility>

class PaymentStrategy {
public:
    virtual void pay(int amountInCents) const = 0;
    virtual ~PaymentStrategy() = default;
};

class CardPayment : public PaymentStrategy {
public:
    void pay(int amountInCents) const override {
        std::cout << "Card charged " << amountInCents << " cents\n";
    }
};

class UpiPayment : public PaymentStrategy {
public:
    void pay(int amountInCents) const override {
        std::cout << "UPI paid " << amountInCents << " cents\n";
    }
};

class Checkout { // Context: delegates the algorithm to its strategy.
public:
    explicit Checkout(std::unique_ptr<PaymentStrategy> strategy)
        : strategy_(std::move(strategy)) {}

    void completeOrder(int amountInCents) const {
        strategy_->pay(amountInCents);
    }

private:
    std::unique_ptr<PaymentStrategy> strategy_;
};

int main() {
    Checkout checkout(std::make_unique<UpiPayment>());
    checkout.completeOrder(1299);
}
```

## Python

```python
class CardPayment:
    def pay(self, amount_in_cents):
        print(f"Card charged {amount_in_cents} cents")


class UpiPayment:
    def pay(self, amount_in_cents):
        print(f"UPI paid {amount_in_cents} cents")


class Checkout:
    def __init__(self, payment_strategy):
        self._payment_strategy = payment_strategy

    def complete_order(self, amount_in_cents):
        # Context delegates; it does not contain payment-specific branching.
        self._payment_strategy.pay(amount_in_cents)


checkout = Checkout(UpiPayment())
checkout.complete_order(1299)
```

## Go

```go
package main

import "fmt"

type PaymentStrategy interface {
	Pay(amountInCents int)
}

type CardPayment struct{}

func (CardPayment) Pay(amountInCents int) {
	fmt.Println("Card charged", amountInCents, "cents")
}

type UpiPayment struct{}

func (UpiPayment) Pay(amountInCents int) {
	fmt.Println("UPI paid", amountInCents, "cents")
}

// Checkout is the context and delegates to the selected strategy.
type Checkout struct {
	strategy PaymentStrategy
}

func (checkout Checkout) CompleteOrder(amountInCents int) {
	checkout.strategy.Pay(amountInCents)
}

func main() {
	checkout := Checkout{strategy: UpiPayment{}}
	checkout.CompleteOrder(1299)
}
```

## Class relationship and selection

```text
Client chooses strategy
       │
       ▼
┌──────────────┐       uses       ┌─────────────────┐
│   Checkout   │ ───────────────► │ PaymentStrategy │
│   (context)  │                  └────────▲────────┘
└──────────────┘                           │ implements
                              ┌────────────┴──────────┐
                              │                       │
                         CardPayment              UpiPayment
```

```cpp
// Valid: change behavior by supplying another strategy.
Checkout cardCheckout(std::make_unique<CardPayment>());
Checkout upiCheckout(std::make_unique<UpiPayment>());

// Avoid: put every algorithm in Checkout and grow this conditional forever.
// if (type == "card") { ... } else if (type == "upi") { ... }
```

## When to use—or avoid—Strategy

Use it when multiple algorithms satisfy the same job and the choice can vary by configuration, request, or runtime conditions. It improves extension and testing because each algorithm is isolated.

Avoid it when there is only one stable algorithm or when selecting among two tiny behaviors is clearer as a simple conditional. Strategy adds interface/classes and moves selection responsibility to a caller or factory.

## Other examples

- Route planning: fastest, shortest, or avoid-tolls route.
- Pricing: regular, holiday, or membership discount.
- Compression: ZIP, gzip, or no compression.
- Sorting/filtering: different comparison or ranking rules.
