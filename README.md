---
layout: default
title: SWE Notes
---

# SWE Notes

> A growing, practical reference for software engineering learning and revision—designed for fast revision on desktop or mobile.

## Start here

| Track | What it covers | Notes |
| --- | --- | --- |
| 🧩 Low-Level Design | C++ fundamentals, OOP, SOLID, and design patterns | [Open C++ basics](lld/cpp-basics/cpp-basics.md) |
| 🧠 Data Structures & Algorithms | Data structures, problem-solving patterns, and algorithms | Coming soon |
| 🏗️ High-Level Design | System-design fundamentals, building blocks, and case studies | Coming soon |
| 🔗 Resources | Helpful articles, videos, and revision links | Coming soon |

---

## 🧩 Low-Level Design

### C++ basics

- [C++ basics quick reference](lld/cpp-basics/cpp-basics.md) — currently covers:
  - `explicit` constructors and implicit conversions
  - RAII and resource lifetime
  - Smart pointers: `unique_ptr`, `shared_ptr`, and `weak_ptr`
  - Borrowing with references and raw pointers
  - Type casting: implicit conversions, `static_cast`, `dynamic_cast`, `const_cast`, and `reinterpret_cast`

### Design patterns

#### Creational

| Pattern | One-line memory aid | Reference |
| --- | --- | --- |
| Singleton | One controlled, shared instance | [Read →](lld/design-patterns/creational/singleton.md) |
| Builder | Construct a complex object step by step | [Read →](lld/design-patterns/creational/builder.md) |
| Factory Method | Let creator subclasses choose which product to create | [Read →](lld/design-patterns/creational/factory-method.md) |

#### Structural

| Pattern | One-line memory aid | Reference |
| --- | --- | --- |
| Adapter | Translate one interface into another the client expects | [Read →](lld/design-patterns/structural/adapter.md) |
| Decorator | Wrap an object to add behavior through the same interface | [Read →](lld/design-patterns/structural/decorator.md) |
| Composite | Treat an item and a group of items uniformly | [Read →](lld/design-patterns/structural/composite.md) |
| Facade | Simplify a complex subsystem behind one entry point | [Read →](lld/design-patterns/structural/facade.md) |
| Proxy | Control access to a real object through the same interface | [Read →](lld/design-patterns/structural/proxy.md) |

#### Behavioral

| Pattern | One-line memory aid | Reference |
| --- | --- | --- |
| Strategy | Make interchangeable algorithms selectable | [Read →](lld/design-patterns/behavioral/strategy.md) |

---

## Revision workflow

1. Pick one topic from the table above.
2. Read the **what problem does it solve?** section first.
3. Study the commented code and the valid/invalid examples.
4. Say the summary aloud without looking.
5. Implement a small version from memory.

---

## Repository map

```text
swe-notes/
├── lld/
│   ├── cpp-basics/
│   └── design-patterns/
│       ├── creational/
│       ├── structural/
│       └── behavioral/
├── dsa/
├── hld/
│   ├── fundamentals/
│   ├── building-blocks/
│   └── system-designs/
└── resources/
```

## Guiding principles

- Prefer understanding over memorizing code.
- Keep notes short enough to revise quickly.
- Include the purpose, valid examples, common mistakes, and one rule to remember.
- Add practical examples in C++, Python, and Go when comparison helps.
