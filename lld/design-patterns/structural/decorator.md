---
layout: default
title: Decorator
---

# Decorator: LLD quick reference

## Purpose

Decorator adds behavior to one object by wrapping it in another object that implements the same interface. Wrappers can be combined at runtime, avoiding a subclass for every feature combination.

```text
Client → Notifier interface
             ▲
             │ same interface
       LoggingDecorator ──wraps──► EmailNotifier
```

> Core idea: **Decorator wraps an object with the same interface and adds behavior before or after delegation.**

## C++: notifications with optional logging

```cpp
#include <iostream>
#include <memory>
#include <string>
#include <utility>

class Notifier {
public:
    virtual void send(const std::string& message) = 0;
    virtual ~Notifier() = default;
};

class EmailNotifier : public Notifier {
public:
    void send(const std::string& message) override {
        std::cout << "Email: " << message << '\n';
    }
};

// Decorator is also a Notifier and owns the wrapped Notifier.
class LoggingDecorator : public Notifier {
public:
    explicit LoggingDecorator(std::unique_ptr<Notifier> wrapped)
        : wrapped_(std::move(wrapped)) {}

    void send(const std::string& message) override {
        std::cout << "Log: sending notification\n"; // Added behavior.
        wrapped_->send(message);                    // Original behavior.
    }

private:
    std::unique_ptr<Notifier> wrapped_;
};

int main() {
    std::unique_ptr<Notifier> notifier =
        std::make_unique<LoggingDecorator>(std::make_unique<EmailNotifier>());
    notifier->send("Your order shipped");
}
```

## Python

```python
class Notifier:
    def send(self, message):
        raise NotImplementedError


class EmailNotifier(Notifier):
    def send(self, message):
        print(f"Email: {message}")


class LoggingDecorator(Notifier):
    def __init__(self, wrapped):
        self._wrapped = wrapped

    def send(self, message):
        print("Log: sending notification")
        self._wrapped.send(message)


notifier = LoggingDecorator(EmailNotifier())
notifier.send("Your order shipped")
```

## Go

```go
package main

import "fmt"

type Notifier interface {
	Send(message string)
}

type EmailNotifier struct{}

func (EmailNotifier) Send(message string) {
	fmt.Println("Email:", message)
}

// LoggingDecorator satisfies Notifier and wraps another Notifier.
type LoggingDecorator struct {
	wrapped Notifier
}

func (decorator LoggingDecorator) Send(message string) {
	fmt.Println("Log: sending notification")
	decorator.wrapped.Send(message)
}

func main() {
	var notifier Notifier = LoggingDecorator{wrapped: EmailNotifier{}}
	notifier.Send("Your order shipped")
}
```

## Class relationship and stacking

```text
Client → Notifier
            ▲
            │ implements same interface
     LoggingDecorator ──wraps──► Notifier
            ▲                         ▲
            │                         │
     RetryDecorator ─────────── wraps ┘
                                      │
                               EmailNotifier
```

You can stack logging, retry, metrics, compression, or encryption decorators. Order can matter: logging outside retry records one request; logging inside retry may record each attempt.

## Decorator versus inheritance

```text
Inheritance approach: EmailWithLoggingAndRetryNotifier
                      EmailWithMetricsNotifier
                      SmsWithLoggingAndRetryNotifier ...

Decorator approach:   Retry(Logging(EmailNotifier))
```

Decorator avoids a subclass for each combination. The tradeoff is more wrapper objects and a call path that can be harder to trace. Use it when optional behaviors need runtime composition; avoid it for one fixed, simple behavior.

## Other examples

- Coffee + milk + syrup, each wrapper adding to cost/description.
- Data stream wrapped with compression and encryption.
- HTTP client wrapped with retry, metrics, and tracing.
