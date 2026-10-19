# SWE Interview Notes

> A growing, practical reference for software engineering interview preparation—designed for fast revision on desktop or mobile.

## Start here

| Track | What it covers | Notes |
| --- | --- | --- |
| 🧩 [Low-Level Design](lld/) | C++ fundamentals, OOP, SOLID, and design patterns | [Open LLD notes](lld/) |
| 🧠 [Data Structures & Algorithms](dsa/) | Data structures, problem-solving patterns, and algorithms | Coming soon |
| 🏗️ [High-Level Design](hld/) | System-design fundamentals, building blocks, and case studies | Coming soon |
| 🔗 [Resources](resources/) | Helpful articles, videos, and revision links | Coming soon |

---

## 🧩 Low-Level Design

### C++ basics

- [`explicit`, RAII, and smart pointers](lld/cpp-basics/cpp-basics.md) — safe conversions, ownership, and lifetime management.

### Design patterns

| Pattern | One-line memory aid | Reference |
| --- | --- | --- |
| Singleton | One controlled, shared instance | [Read →](lld/design-patterns/singleton.md) |
| Builder | Construct a complex object step by step | [Read →](lld/design-patterns/builder.md) |
| Factory Method | Let subclasses choose which product to create | [Read →](lld/design-patterns/factory-method.md) |
| Facade | Simplify a complex subsystem behind one entry point | [Read →](lld/design-patterns/facade.md) |

---

## Revision workflow

1. Pick one topic from the table above.
2. Read the **what problem does it solve?** section first.
3. Study the commented code and the valid/invalid examples.
4. Say the interview summary aloud without looking.
5. Implement a small version from memory.

---

## Repository map

```text
swe-interview-notes/
├── lld/
│   ├── cpp-basics/
│   └── design-patterns/
├── dsa/
├── hld/
│   ├── fundamentals/
│   ├── building-blocks/
│   └── system-designs/
└── resources/
```

## Guiding principles

- Prefer understanding over memorizing code.
- Keep notes short enough to revise before an interview.
- Include the purpose, valid examples, common mistakes, and one rule to remember.
- Add practical examples in C++, Python, and Go when comparison helps.
