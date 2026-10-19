---
layout: default
title: Observer
---

# Observer: LLD quick reference

## Purpose

Observer defines a one-to-many relationship: when one object (the **subject/publisher**) changes, it notifies its registered **observers/subscribers**. The subject does not need to know the concrete observer classes; it only depends on their shared notification contract.

Example: when an order changes status, email, SMS, and analytics listeners can each react.

> Core idea: **Observers subscribe to a subject and are notified when its state changes.**

## UML class diagram — order status subscribers

```text
┌─────────────────┐  subscribe(observer)  ┌────────────────────────────┐
│     Client      │ ─────────────────────►│       OrderSubject         │
├─────────────────┤                       ├────────────────────────────┤
│ + main()        │                       │ - status: string           │
└─────────────────┘                       │ - observers: collection     │
                                          │ + subscribe(observer)     │
                                          │ + unsubscribe(observer)   │
                                          │ + setStatus(status)       │
                                          │ - notify()                │
                                          └──────────────┬─────────────┘
                                                         │ notifies 0..*
                                                         ▼
                                          ┌────────────────────────────┐
                                          │       OrderObserver        │
                                          ├────────────────────────────┤
                                          │ + onStatusChanged(status) │
                                          └──────────────▲─────────────┘
                                                         │ implements
                                      ┌──────────────────┴──────────────────┐
                                      │                                     │
                         ┌────────────┴─────────┐             ┌────────────┴─────────┐
                         │    EmailObserver     │             │  AnalyticsObserver   │
                         ├──────────────────────┤             ├──────────────────────┤
                         │ + onStatusChanged()  │             │ + onStatusChanged()  │
                         └──────────────────────┘             └──────────────────────┘
```

The subject stores references to observers, notifies them when its state changes, and can usually add/remove them. Concrete observers implement the same callback but perform different actions.

## C++: order status notifications

```cpp
#include <algorithm>
#include <iostream>
#include <string>
#include <utility>
#include <vector>

// Observer contract: every subscriber must respond to a status change.
class OrderObserver {
public:
    virtual void onStatusChanged(const std::string& status) = 0;
    virtual ~OrderObserver() = default;
};

class EmailObserver : public OrderObserver {
public:
    void onStatusChanged(const std::string& status) override {
        std::cout << "Email: order is " << status << '\n';
    }
};

class AnalyticsObserver : public OrderObserver {
public:
    void onStatusChanged(const std::string& status) override {
        std::cout << "Analytics recorded: " << status << '\n';
    }
};

// Subject/publisher owns no observers here; it stores non-owning pointers.
class OrderSubject {
public:
    void subscribe(OrderObserver& observer) {
        observers_.push_back(&observer);
    }

    void unsubscribe(OrderObserver& observer) {
        observers_.erase(
            std::remove(observers_.begin(), observers_.end(), &observer),
            observers_.end());
    }

    void setStatus(std::string status) {
        status_ = std::move(status);
        notify(); // Every current subscriber hears about the state change.
    }

private:
    void notify() const {
        for (OrderObserver* observer : observers_) {
            observer->onStatusChanged(status_);
        }
    }

    std::string status_ = "created";
    std::vector<OrderObserver*> observers_; // Observers must outlive subscriptions.
};

int main() {
    OrderSubject order;
    EmailObserver email;
    AnalyticsObserver analytics;

    order.subscribe(email);
    order.subscribe(analytics);
    order.setStatus("shipped");
    // Email: order is shipped
    // Analytics recorded: shipped

    order.unsubscribe(email); // Email no longer receives later updates.
}
```

### C++ lifetime note

This example uses non-owning observer pointers for brevity. Each observer must remain alive while subscribed and must unsubscribe before it is destroyed. Larger systems often use `std::weak_ptr` subscriptions or a scoped connection/token so a publisher cannot call a destroyed observer.

## Python

```python
class OrderSubject:
    def __init__(self):
        self._observers = []
        self._status = "created"

    def subscribe(self, observer):
        self._observers.append(observer)

    def unsubscribe(self, observer):
        self._observers.remove(observer)

    def set_status(self, status):
        self._status = status
        for observer in self._observers:
            observer.on_status_changed(status)


class EmailObserver:
    def on_status_changed(self, status):
        print(f"Email: order is {status}")


class AnalyticsObserver:
    def on_status_changed(self, status):
        print(f"Analytics recorded: {status}")


order = OrderSubject()
email = EmailObserver()
analytics = AnalyticsObserver()

order.subscribe(email)
order.subscribe(analytics)
order.set_status("shipped")
# Email: order is shipped
# Analytics recorded: shipped
order.unsubscribe(email)
```

Python commonly uses duck typing: any object with `on_status_changed` can be an observer; a base class is optional.

## Go

```go
package main

import "fmt"

// Observer is the callback contract expected by the subject.
type Observer interface {
	OnStatusChanged(status string)
}

type EmailObserver struct{}

func (EmailObserver) OnStatusChanged(status string) {
	fmt.Println("Email: order is", status)
}

type AnalyticsObserver struct{}

func (AnalyticsObserver) OnStatusChanged(status string) {
	fmt.Println("Analytics recorded:", status)
}

type OrderSubject struct {
	status    string
	observers []Observer
}

func (order *OrderSubject) Subscribe(observer Observer) {
	order.observers = append(order.observers, observer)
}

func (order *OrderSubject) SetStatus(status string) {
	order.status = status
	for _, observer := range order.observers {
		observer.OnStatusChanged(status)
	}
}

func main() {
	order := OrderSubject{status: "created"}
	order.Subscribe(EmailObserver{})
	order.Subscribe(AnalyticsObserver{})
	order.SetStatus("shipped")
	// Email: order is shipped
	// Analytics recorded: shipped
}
```

This compact Go example omits unsubscribe; production code can remove a subscriber by ID/token or use a subscription object with an explicit `Close` method.

## When to use—or avoid—Observer

Use Observer when one event should trigger several independent reactions, and the publisher should not depend on their concrete classes. It is common in UI events, domain events, notifications, and change listeners.

Avoid it when a direct function call is simpler, or when hidden notification chains make execution order difficult to understand. Decide whether notification is synchronous (as above) or asynchronous, and define what happens if one observer fails.

## Common mistakes

- Forgetting to unsubscribe or otherwise manage lifetimes; this can leave stale subscriptions or cause use-after-free in C++.
- Putting concrete observer knowledge in the subject, such as `if (emailObserver) ...`; that defeats loose coupling.
- Expecting observers to run in parallel. These examples notify synchronously, one after another.
- Creating a circular chain where observers update the subject and trigger repeated notifications.

## Other examples

- A stock-price subject notifies chart, alert, and portfolio observers.
- A UI button notifies click handlers when pressed.
- A news publisher notifies subscribers when an article is published.
- A file watcher notifies cache invalidation and indexing services when a file changes.
