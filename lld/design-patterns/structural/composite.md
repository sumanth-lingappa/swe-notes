---
layout: default
title: Composite
---

# Composite: LLD quick reference

## Purpose

Composite represents a part-whole tree. A leaf (one item) and a composite (a group of items) share an interface, so clients can call the same operation on either.

Filesystem example: a file returns its own size; a folder adds the sizes of its children, including nested folders. Creating files/folders is not the pattern's main purpose; the focus is uniform operations on leaves and groups.

```text
FileSystemItem (component)
├── File (leaf): returns its own size
└── Folder (composite): contains FileSystemItems, sums child sizes
```

> Core idea: **Treat one object and a tree of objects uniformly through a shared interface.**

## C++: in-memory filesystem tree

```cpp
#include <iostream>
#include <cstdint>
#include <memory>
#include <string>
#include <utility>
#include <vector>

class FileSystemItem {
public:
    virtual std::uintmax_t getSize() const = 0;
    virtual void print(int depth = 0) const = 0;
    virtual ~FileSystemItem() = default;
};

class File : public FileSystemItem { // Leaf: no children.
public:
    File(std::string name, std::uintmax_t size)
        : name_(std::move(name)), size_(size) {}

    std::uintmax_t getSize() const override { return size_; }

    void print(int depth = 0) const override {
        std::cout << std::string(depth * 2, ' ') << name_ << " (" << size_ << " bytes)\n";
    }

private:
    std::string name_;
    std::uintmax_t size_;
};

class Folder : public FileSystemItem { // Composite: contains components.
public:
    explicit Folder(std::string name) : name_(std::move(name)) {}

    void add(std::unique_ptr<FileSystemItem> item) {
        children_.push_back(std::move(item));
    }

    std::uintmax_t getSize() const override {
        std::uintmax_t total = 0;
        for (const auto& child : children_) {
            total += child->getSize(); // Polymorphism handles file or folder.
        }
        return total;
    }

    void print(int depth = 0) const override {
        std::cout << std::string(depth * 2, ' ') << name_ << "/\n";
        for (const auto& child : children_) {
            child->print(depth + 1); // Recursively prints nested contents.
        }
    }

private:
    std::string name_;
    std::vector<std::unique_ptr<FileSystemItem>> children_;
};

int main() {
    auto root = std::make_unique<Folder>("root");
    root->add(std::make_unique<File>("report.pdf", 200));

    auto docs = std::make_unique<Folder>("docs");
    docs->add(std::make_unique<File>("notes.txt", 50));
    root->add(std::move(docs));

    root->print();
    std::cout << "Total: " << root->getSize() << " bytes\n"; // 250
}
```

## Python

```python
from abc import ABC, abstractmethod


class FileSystemItem(ABC):
    @abstractmethod
    def get_size(self):
        pass

    @abstractmethod
    def print(self, depth=0):
        pass


class File(FileSystemItem):  # Leaf
    def __init__(self, name, size):
        self.name = name
        self.size = size

    def get_size(self):
        return self.size

    def print(self, depth=0):
        print("  " * depth + f"{self.name} ({self.size} bytes)")


class Folder(FileSystemItem):  # Composite
    def __init__(self, name):
        self.name = name
        self.children = []

    def add(self, item):
        self.children.append(item)

    def get_size(self):
        return sum(child.get_size() for child in self.children)

    def print(self, depth=0):
        print("  " * depth + self.name + "/")
        for child in self.children:
            child.print(depth + 1)


root = Folder("root")
root.add(File("report.pdf", 200))
docs = Folder("docs")
docs.add(File("notes.txt", 50))
root.add(docs)
root.print()
print("Total:", root.get_size(), "bytes")  # 250
```

## Go

```go
package main

import "fmt"

// Component contract shared by leaves and composites.
type FileSystemItem interface {
	GetSize() uint64
	Print(depth int)
}

type File struct { // Leaf
	name string
	size uint64
}

func (file File) GetSize() uint64 { return file.size }

func (file File) Print(depth int) {
	fmt.Printf("%s%s (%d bytes)\n", indent(depth), file.name, file.size)
}

type Folder struct { // Composite
	name     string
	children []FileSystemItem
}

func (folder *Folder) Add(item FileSystemItem) {
	folder.children = append(folder.children, item)
}

func (folder Folder) GetSize() uint64 {
	var total uint64
	for _, child := range folder.children {
		total += child.GetSize() // Child decides its own behavior.
	}
	return total
}

func (folder Folder) Print(depth int) {
	fmt.Printf("%s%s/\n", indent(depth), folder.name)
	for _, child := range folder.children {
		child.Print(depth + 1)
	}
}

func indent(depth int) string {
	result := ""
	for i := 0; i < depth; i++ {
		result += "  "
	}
	return result
}

func main() {
	root := &Folder{name: "root"}
	root.Add(File{name: "report.pdf", size: 200})
	docs := &Folder{name: "docs"}
	docs.Add(File{name: "notes.txt", size: 50})
	root.Add(docs)

	root.Print(0)
	fmt.Println("Total:", root.GetSize(), "bytes") // 250
}
```

## Class relationship

```text
                 ┌───────────────┐
                 │ FileSystemItem│  component interface
                 │ getSize/print │
                 └───────▲───────┘
                         │ implements
               ┌─────────┴──────────┐
          ┌────┴────┐          ┌────┴─────┐
          │  File   │          │  Folder  │
          │  leaf   │          │composite │
          └─────────┘          └────┬─────┘
                                    │ contains zero or more
                                    └── FileSystemItem children
```

## Common mistake and tradeoff

```cpp
// Avoid type-specific client branching:
if (isFile(item)) size += fileSize(item);
else if (isFolder(item)) /* recursively inspect folder */;
```

With Composite, a folder simply calls the same `getSize()` operation on each child. The tradeoff is that a shared component interface may expose operations that do not make sense for every node; keep the interface focused on truly common operations.

## Other examples

- UI: a button is a leaf; a panel contains buttons and nested panels; render either uniformly.
- Organization: employee is a leaf; manager contains employees/managers; calculate subtree headcount.
- Graphics: shape is a leaf; group contains shapes/groups; move or draw the whole group.
