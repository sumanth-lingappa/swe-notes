---
layout: default
title: C++ basics
---

# C++ basics: quick reference

This note collects small but important C++ concepts for quick revision.

<details markdown="1">
<summary><strong>1. <code>std::string</code>: member-method reference</strong></summary>


`std::string` is the standard string type for byte/`char` text. The table uses `s = "hello"` unless a row says otherwise. Overloads are grouped under one method name; the examples show common forms rather than every possible signature.

Add this header:

```cpp
#include <string>
```

| Group | Method | Example input/use | Return value or result |
| --- | --- | --- | --- |
| Create/assign | `operator=` | `s = "hi";` | `s` becomes `"hi"`; returns `s` by reference. |
| Create/assign | `assign(...)` | `s.assign("hey");` | `s` becomes `"hey"`; returns `s` by reference. Can also assign a substring, count of characters, or iterator range. |
| Create/assign | `get_allocator()` | `auto a = s.get_allocator();` | Returns the allocator object used by `s`; usually only needed for advanced memory-management code. |
| Read characters | `at(pos)` | `s.at(1)` | `'e'`; checks bounds and throws `std::out_of_range` if `pos >= s.size()`. |
| Read characters | `operator[](pos)` | `s[1]` | `'e'`; does not perform normal bounds checking. |
| Read characters | `front()` | `s.front()` | First character, `'h'`. Do not call on an empty string. |
| Read characters | `back()` | `s.back()` | Last character, `'o'`. Do not call on an empty string. |
| Read characters | `data()` | `s.data()` | Pointer to the characters; in modern C++, the character storage is null-terminated. |
| Read characters | `c_str()` | `s.c_str()` | `const char*` view of `"hello\0"`, suitable for C APIs; do not modify it. |
| Iterate | `begin()`, `cbegin()` | `*s.begin()` | Iterator / const iterator to `'h'`. |
| Iterate | `end()`, `cend()` | `s.begin() != s.end()` | End iterator is one past the last character; never dereference it. |
| Iterate | `rbegin()`, `crbegin()` | `*s.rbegin()` | Reverse iterator / const reverse iterator to `'o'`. |
| Iterate | `rend()`, `crend()` | `s.rbegin() != s.rend()` | Reverse end iterator; one past the last character in reverse order. |
| Size/capacity | `empty()` | `s.empty()` | `false`; says whether `size() == 0`. |
| Size/capacity | `size()`, `length()` | `s.size()` | `5`; both return the number of `char` elements. |
| Size/capacity | `max_size()` | `s.max_size()` | Maximum size this string could theoretically hold; implementation-dependent. |
| Size/capacity | `capacity()` | `s.capacity()` | Allocated character capacity; at least `size()`, exact value is implementation-dependent. |
| Size/capacity | `reserve(n)` | `s.reserve(100);` | Requests capacity of at least `100`; does not change the string's size or text. |
| Size/capacity | `shrink_to_fit()` | `s.shrink_to_fit();` | Requests reduced capacity; the request is non-binding and text is unchanged. |
| Modify | `clear()` | `s.clear();` | `s` becomes empty. |
| Modify | `insert(...)` | `s.insert(1, "ey");` | `s` becomes `"heyello"`; inserts at a position. Overloads can insert characters, strings, counts, or ranges. |
| Modify | `erase(...)` | `s.erase(1, 2);` | `s` becomes `"hlo"`; removes up to 2 characters starting at index 1. |
| Modify | `push_back(ch)` | `s.push_back('!');` | `s` becomes `"hello!"`; appends one character. |
| Modify | `pop_back()` | `s.pop_back();` | `s` becomes `"hell"`; removes the last character. Do not call on an empty string. |
| Modify | `append(...)` | `s.append("!");` | `s` becomes `"hello!"`; appends text, a count of characters, or a range. |
| Modify | `operator+=` | `s += "!";` | `s` becomes `"hello!"`; appends a string, C string, or single character. |
| Modify | `replace(...)` | `s.replace(1, 2, "ipp");` | `s` becomes `"hipplo"`; replaces 2 characters starting at index 1. |
| Modify | `resize(n[, ch])` | `s.resize(3);` | `s` becomes `"hel"`; shrinking removes the end. Growing appends `ch` (or `\0` if omitted). |
| Modify | `copy(buffer, count, pos)` | `char out[4]; auto n = s.copy(out, 3, 1);` | Copies `"ell"` into `out`, returns `3`; does **not** append a null terminator. |
| Modify | `swap(other)` | `s.swap(other);` | Exchanges the contents of `s` and `other`; returns `void`. |
| Modify | `resize_and_overwrite(n, op)` (C++23) | `s.resize_and_overwrite(3, [](char* p, std::size_t n) { p[0]='c'; p[1]='a'; p[2]='t'; return n; });` | `s` becomes `"cat"`; lets a callback write into the string's writable buffer and return the resulting length. Advanced; follow the callback requirements carefully. |
| Modify ranges | `append_range(r)` (C++23) | `s.append_range(std::string("!"));` | Appends the range; `s` becomes `"hello!"`. |
| Modify ranges | `insert_range(pos, r)` (C++23) | `s.insert_range(s.begin() + 1, std::string("ey"));` | Inserts the range at the iterator; `s` becomes `"heyello"`. |
| Modify ranges | `replace_with_range(first, last, r)` (C++23) | `s.replace_with_range(s.begin() + 1, s.begin() + 3, std::string("ipp"));` | Replaces the selected range; `s` becomes `"hipplo"`. |
| Search | `find(value[, pos])` | `s.find("ll")` | `2`, the first match at or after `pos` (default `0`); returns `std::string::npos` if not found. |
| Search | `rfind(value[, pos])` | `s.rfind('l')` | `3`, the last matching position at or before `pos` (default: search from the end); or `npos`. |
| Search | `find_first_of(chars[, pos])` | `s.find_first_of("aeiou")` | `1`, first character in `s` that is any character from `chars`; or `npos`. |
| Search | `find_first_not_of(chars[, pos])` | `s.find_first_not_of("he")` | `2`, first character not present in `chars`; or `npos`. |
| Search | `find_last_of(chars[, pos])` | `s.find_last_of("lo")` | `4`, last character in `s` that is any character from `chars`; or `npos`. |
| Search | `find_last_not_of(chars[, pos])` | `s.find_last_not_of("lo")` | `1`, last character not present in `chars`; or `npos`. |
| Search | `starts_with(value)` (C++20) | `s.starts_with("he")` | `true`; checks whether the beginning matches. |
| Search | `ends_with(value)` (C++20) | `s.ends_with("lo")` | `true`; checks whether the end matches. |
| Search | `contains(value)` (C++23) | `s.contains("ll")` | `true`; checks whether the substring/character exists. Before C++23, use `find(...) != std::string::npos`. |
| Extract/compare | `substr(pos[, count])` | `s.substr(1, 3)` | A new string, `"ell"`; `count` means number of characters, not ending index. |
| Extract/compare | `compare(other)` | `s.compare("hello")` | `0` if equal, negative if this string sorts before `other`, positive if after. Prefer `==`, `<`, etc. for ordinary comparisons. |
| Static constant | `npos` | `s.find("x") == std::string::npos` | Special value meaning “no position found”; it is not a boolean. |

### Quick examples

```cpp
std::string text = "  hello  ";

const auto at = text.find("hello");
if (at != std::string::npos) {
    std::string word = text.substr(at, 5); // "hello"
}

const std::string whitespace = " \t\n\r";
const auto first = text.find_first_not_of(whitespace);
const auto last = text.find_last_not_of(whitespace);
std::string trimmed = (first == std::string::npos)
    ? ""
    : text.substr(first, last - first + 1); // "hello"
```

### Important notes

- The table groups overloads by method name; it is a practical reference, not a listing of every signature.
- `size()` counts `char` elements (bytes for `std::string`), not user-perceived Unicode characters. A UTF-8 character can occupy multiple bytes.
- `capacity()` is storage reserved by the string; `size()` is the number of characters currently in it.
- `std::string` has no built-in `trim()` or `toUpper()` member. Use standard algorithms or write a helper.
- `operator+`, comparisons (`==`, `<`), `std::getline`, `std::erase`, and `std::stoi` are related non-member functions/utilities, not `std::string` member methods.
- Methods marked C++20/C++23 require compiling with an appropriate standard; common interview environments may use C++17, where those rows won't compile.

The standard defines the complete overload set and version additions in [`basic_string`](https://eel.is/c%2B%2Bdraft/string.classes).

</details>

<details markdown="1">
<summary><strong>2. <code>std::ostringstream</code>: build and format strings</strong></summary>

`std::ostringstream` is an output stream that writes into an in-memory `std::string`. Use the familiar `<<` operator to combine text, numbers, and other printable values, then call `.str()` to get the completed string. It is a string-building tool, not a terminal-printing tool. It inherits formatting, output, and state-checking methods from `std::ostream`.

Include:

```cpp
#include <iomanip>  // std::fixed, std::setprecision, std::setw, std::setfill
#include <locale>   // std::locale, if changing locale
#include <sstream>  // std::ostringstream
#include <string>
```

### Basic use

```cpp
std::ostringstream out;
out << "Order #" << 42 << " costs $" << 12.5;

std::string message = out.str();
// message == "Order #42 costs $12.5"
```

The stream is a temporary string-building workspace. It does not print to the terminal; `std::cout` does that.

### Format a balance to two decimal places

```cpp
#include <iomanip>
#include <sstream>
#include <string>

std::string accountInfo(
    const std::string& ownerName,
    const std::string& accountNumber,
    double balance)
{
    std::ostringstream out;
    out << ownerName << " (" << accountNumber << ") | Balance: $"
        << std::fixed << std::setprecision(2) << balance;
    return out.str();
}

// accountInfo("Asha", "1234", 12.5)
// -> "Asha (1234) | Balance: $12.50"
```

`std::fixed` makes precision mean digits after the decimal point. With `std::fixed << std::setprecision(2)`, `12.5` becomes `12.50`. Without `std::fixed`, precision means significant digits instead.

### Useful stream operations and formatting tools

| Name | What it does | Example | Result / note |
| --- | --- | --- | --- |
| `operator<<` | Appends a value to the stream using its output formatting. | `out << "count=" << 3;` | Buffer contains `count=3`. Works with strings, characters, numbers, and types that define stream output. |
| `str()` | Gets a copy of the current buffer as a `std::string`. | `std::string text = out.str();` | Returns all text written so far. |
| `str(newText)` | Replaces the buffer contents. | `out.str("start");` | Buffer becomes `start`. This does not reset stream error flags. |
| `clear()` | Resets stream state flags such as `failbit`; it does **not** erase buffered text. | `out.clear();` | Stream state is usable again. To erase text too, call `out.str("");`. |
| `precision(n)` | Sets floating-point precision. | `out.precision(2);` | With default formatting, means significant digits; with `std::fixed`, means digits after the decimal. |
| `width()` / `width(n)` | Gets/sets the minimum width for the next formatted field. | `out.width(4); out << 7;` | Writes three leading spaces and `7`; width resets after the next formatted field. |
| `fill()` / `fill(ch)` | Gets/sets the padding character. | `out.fill('0'); out << std::setw(4) << 7;` | Writes `0007`; fill remains set until changed. |
| `flags()` / `setf()` / `unsetf()` | Reads or changes formatting flags. | `out.setf(std::ios::fixed);` | Enables fixed decimal formatting. Manipulators like `std::fixed` are usually easier to read. |
| `imbue(locale)` / `getloc()` | Sets or gets the stream's locale. | `out.imbue(std::locale(""));` | Locale can affect numeric formatting, such as decimal separators; requires `<locale>`. |
| `std::fixed` | Formats floating-point values in fixed decimal notation. | `out << std::fixed << 12.5;` | Writes `12.500000` by default (default precision is commonly 6). Pair with `std::setprecision`. |
| `std::setprecision(n)` | Sets precision for subsequent floating-point output. | `out << std::fixed << std::setprecision(2) << 12.5;` | Writes `12.50`. Requires `<iomanip>`. |
| `std::setw(n)` | Sets the minimum width of the **next** formatted field. | `out << std::setw(4) << 7;` | Writes three leading spaces, then `7`. Requires `<iomanip>`. |
| `std::setfill(ch)` | Chooses the padding character used by width formatting. | `out << std::setfill('0') << std::setw(4) << 7;` | Writes `0007`. Requires `<iomanip>`. |
| `std::left`, `std::right` | Chooses alignment when a field is wider than its contents. | `out << std::left << std::setw(6) << "Hi";` | Writes `Hi` followed by four spaces. Alignment setting remains active. |
| `std::boolalpha` | Prints booleans as words instead of `1` or `0`. | `out << std::boolalpha << true;` | Writes `true`; setting remains active. |
| `std::hex`, `std::oct`, `std::dec` | Chooses the base for integer output. | `out << std::hex << 255;` | Writes `ff`; use `std::dec` to return to decimal. |
| `view()` (C++20) | Returns a non-owning view of the current buffer. | `std::string_view v = out.view();` | Reads the current characters without copying. Include `<string_view>`; the view can be invalidated when the stream changes or is destroyed. |
| `put(ch)` | Writes one character. | `out.put('!');` | Appends `!`; equivalent in spirit to `out << '!'`. |
| `write(ptr, count)` | Writes exactly `count` characters from a buffer. | `out.write("hey!", 3);` | Appends `hey`; does not stop at a null character. |
| `tellp()` | Reports the current output position. | `auto position = out.tellp();` | Returns a stream position; can be used with `seekp` for positioning. |
| `seekp(position)` | Moves the output position. | `out.seekp(0);` | The next write starts at the requested position, if supported by the buffer. Usually unnecessary for basic string building. |
| `good()`, `fail()`, `bad()`, `eof()` | Check stream state. | `if (out.fail()) { /* handle error */ }` | Return `bool`. These are inherited stream methods; output streams rarely need explicit state checks in simple examples. |
| `rdstate()` | Gets the current state flags. | `auto state = out.rdstate();` | Returns a state bitmask such as `goodbit` or `failbit`; mostly for advanced stream handling. |
| `setstate(flags)` | Adds error/state flags. | `out.setstate(std::ios::failbit);` | Marks the stream as failed; normally the stream sets these flags itself. |
| `exceptions(mask)` | Chooses which state flags cause exceptions. | `out.exceptions(std::ios::failbit);` | Future failures matching the mask throw; advanced error handling. |
| `rdbuf()` | Gets the underlying string buffer. | `auto* buffer = out.rdbuf();` | Returns a pointer to the associated stream buffer; usually not needed for everyday formatting. |
| `swap(other)` | Swaps the stream state and underlying buffer with another string stream. | `out.swap(other);` | Exchanges their buffers and stream settings; an advanced operation. |
| `std::flush` | Requests that a stream flush pending output. | `out << std::flush;` | Usually unnecessary for an in-memory string stream; more relevant to terminal/file output streams. |

Formatting settings such as `fixed`, `boolalpha`, and integer base remain active for later insertions on the same stream. `setw` applies only to the next field.

### Reusing a stream

```cpp
std::ostringstream out;
out << "first";

out.str(""); // Erase the accumulated text.
out.clear(); // Reset error flags, if any.

out << "second";
std::string result = out.str(); // "second"
```

### Related string streams

| Type | Direction | Typical use |
| --- | --- | --- |
| `std::ostringstream` | Write values into a string. | Build a formatted message or report. |
| `std::istringstream` | Read/parse values from a string. | Parse fields from text with `>>`. |
| `std::stringstream` | Both read and write. | Use only when you need both directions on the same stream. |

Parsing example:

```cpp
#include <sstream>
#include <string>

std::istringstream input("42 3.5");
int count;
double price;

if (input >> count >> price) {
    // count == 42, price == 3.5
}
```

### When it is useful

- Build a return string from multiple fields and types without a long chain of `+` operations.
- Format numbers consistently in account summaries, receipts, logs, and reports.
- Control decimal places, padding, alignment, boolean words, or integer bases.
- Parse structured values from an existing string with `std::istringstream`.
- Write a helper that returns formatted text instead of printing it directly.

For one simple value, `std::to_string` may be enough. For exact monetary arithmetic, prefer storing cents in an integer (or use a decimal-money type): `ostringstream` formats a `double`, but it does not fix floating-point precision issues.

</details>

<details markdown="1">
<summary><strong>3. <code>explicit</code>: prevent surprising automatic conversions</strong></summary>

### The problem

A constructor that can be called with **one argument** can also become an automatic conversion path unless it is marked `explicit`.

```cpp
#include <string>

class EmailNotification {
public:
    // Without `explicit`, a std::string can silently become an EmailNotification.
    EmailNotification(const std::string& smtpServer)
        : smtpServer_(smtpServer) {}

private:
    std::string smtpServer_;
};
```

```cpp
std::string smtpServer = "smtp.gmail.com";

// Copy initialization. This is allowed above because C++ silently calls:
// EmailNotification(smtpServer)
EmailNotification notification = smtpServer;
```

That conversion might be valid, but it is easy to miss when reading code. A string and an `EmailNotification` are different concepts:

```text
std::string              EmailNotification
"smtp.gmail.com"   →    an object that knows how to send email
```

### The safer version

```cpp
#include <string>

class EmailNotification {
public:
    // `explicit` forbids implicit conversion from std::string.
    explicit EmailNotification(const std::string& smtpServer)
        : smtpServer_(smtpServer) {}

private:
    std::string smtpServer_;
};
```

```cpp
std::string smtpServer = "smtp.gmail.com";

// EmailNotification notification = smtpServer;
// Error: C++ will not silently convert std::string to EmailNotification.

// Direct initialization: clearly says “construct an EmailNotification”.
EmailNotification notification(smtpServer);

// Brace initialization: equally clear.
EmailNotification anotherNotification{smtpServer};
```

### Why it matters in functions

Without `explicit`, a function that expects a fully constructed notification could accidentally accept a string:

```cpp
void addNotificationChannel(EmailNotification notification);

std::string smtpServer = "smtp.gmail.com";

// Without `explicit`: allowed; C++ silently constructs EmailNotification.
addNotificationChannel(smtpServer);
```

With `explicit`, the caller must make the conversion visible:

```cpp
addNotificationChannel(EmailNotification(smtpServer));
```

This is clearer for both the compiler and the person reading the code.

### Direct initialization vs copy initialization

```cpp
std::string smtpServer = "smtp.gmail.com";

// Copy initialization: can use an implicit converting constructor.
EmailNotification a = smtpServer;

// Direct initialization: directly calls the constructor.
EmailNotification b(smtpServer);

// Direct-list initialization: directly calls the constructor.
EmailNotification c{smtpServer};
```

When the constructor is `explicit`:

| Code | Result when the constructor is `explicit` | Why |
| --- | --- | --- |
| `EmailNotification a = smtpServer;` | **Compiler error** | Copy initialization would need an implicit conversion. |
| `EmailNotification b(smtpServer);` | **Valid** | Direct initialization visibly calls the constructor. |
| `EmailNotification c{smtpServer};` | **Valid** | Direct-list initialization visibly calls the constructor. |

### Complete valid/invalid example

```cpp
#include <string>

class EmailNotification {
public:
    explicit EmailNotification(const std::string& smtpServer)
        : smtpServer_(smtpServer) {}

private:
    std::string smtpServer_;
};

void addNotificationChannel(EmailNotification notification) {
    // Store or use the complete EmailNotification object.
}

int main() {
    std::string smtpServer = "smtp.gmail.com";

    // INVALID: compiler error.
    // A std::string is not automatically an EmailNotification.
    // EmailNotification first = smtpServer;

    // VALID: direct construction is clear.
    EmailNotification second(smtpServer);

    // VALID: brace construction is clear too.
    EmailNotification third{smtpServer};

    // INVALID: compiler error.
    // The function asks for EmailNotification, not std::string.
    // addNotificationChannel(smtpServer);

    // VALID: the caller explicitly constructs the required type.
    addNotificationChannel(EmailNotification{smtpServer});
}
```

### Rule of thumb

> Mark a constructor `explicit` when it takes one argument, unless automatic conversion is clearly useful and natural.

For example, turning a server address into an `EmailNotification` should be intentional, so `explicit` is appropriate.

`explicit` is not required by Factory Method, Builder, or any other design pattern. It is a C++ language feature that makes APIs safer and clearer.

</details>

---

<details markdown="1">
<summary><strong>4. RAII and smart pointers: safe resource ownership</strong></summary>

### What is RAII?

**RAII** means **Resource Acquisition Is Initialization**.

It means: put a resource inside an object. The object's constructor acquires the resource, and its destructor releases it automatically when the object leaves scope.

```text
Enter scope                     Leave scope
    │                                │
    ▼                                ▼
object is created             object is destroyed
resource is acquired          resource is released automatically
```

Resources are not only memory. They include files, locks, database connections, sockets, and memory.

### RAII without pointers: prefer an ordinary object

```cpp
#include <iostream>
#include <string>

class File {
public:
    explicit File(const std::string& fileName) {
        std::cout << "Open: " << fileName << '\n';
    }

    ~File() {
        // Runs automatically when the File object leaves scope.
        std::cout << "Close file\n";
    }
};

int main() {
    {
        File file("notes.txt");
        // Use file here.
    } // file.~File() runs automatically here.
}
```

```text
Stack frame for main
┌─────────────────────────────────┐
│ File file("notes.txt")          │
│                                 │
│ file owns the open file resource │
└─────────────────────────────────┘
              │ scope ends
              ▼
File destructor closes the resource
```

This is the best default. No dynamic allocation is needed.

### Smart-pointer headers

```cpp
#include <memory>  // std::unique_ptr, std::shared_ptr, std::weak_ptr,
                    // std::make_unique, std::make_shared
```

`<memory>` is the main header for standard smart pointers.

---

### `std::unique_ptr`: exactly one owner

Use `std::unique_ptr<T>` when one object owns a dynamically allocated `T`.

```cpp
#include <memory>
#include <string>
#include <utility> // std::move

class EmailNotification {
public:
    explicit EmailNotification(std::string smtpServer)
        : smtpServer_(std::move(smtpServer)) {}

private:
    std::string smtpServer_;
};

int main() {
    // Creates EmailNotification on the heap.
    // unique_ptr is the one owner and will delete it automatically.
    auto notification = std::make_unique<EmailNotification>("smtp.gmail.com");

} // notification is destroyed, then EmailNotification is deleted.
```

```text
Stack                                Heap
┌───────────────────────────┐        ┌───────────────────────────┐
│ unique_ptr notification   │ ─────► │ EmailNotification object  │
│ the only owner            │        │ smtpServer = ...          │
└───────────────────────────┘        └───────────────────────────┘
            │ scope ends
            ▼
unique_ptr deletes the heap object
```

#### Valid and invalid `unique_ptr` operations

```cpp
auto first = std::make_unique<EmailNotification>("smtp.gmail.com");

// INVALID: copying would create two owners of one object.
// auto second = first;

// VALID: ownership moves from first to second.
auto second = std::move(first);

// first is now empty (it holds nullptr).
// second is now the only owner.
```

```text
Before std::move:       first ─────► object
                         second      empty

After std::move:        first        empty
                         second ─────► object
```

Use `std::move` only when you intentionally transfer ownership.

---

### `std::shared_ptr`: shared ownership

Use `std::shared_ptr<T>` only when multiple objects must jointly keep `T` alive.

```cpp
#include <memory>

auto config = std::make_shared<AppConfig>();

// Both variables share ownership of the same AppConfig object.
auto orderServiceConfig = config;
auto emailWorkerConfig = config;
```

```text
orderServiceConfig ──┐
                     ├──► AppConfig object
emailWorkerConfig ───┘       shared owner count: 2

When the last shared_ptr owner disappears:
AppConfig object is deleted.
```

Copying a `shared_ptr` is valid because it intentionally increases the shared-owner count.

```cpp
auto first = std::make_shared<AppConfig>();
auto second = first; // Valid: two shared owners.
```

Be careful: shared ownership can make object lifetime harder to reason about. Prefer `unique_ptr` unless ownership is truly shared.

---

### `std::weak_ptr`: observe without owning

`std::weak_ptr<T>` observes an object managed by `shared_ptr`, but does not keep it alive.

```cpp
#include <memory>

std::weak_ptr<AppConfig> observer;

{
    auto owner = std::make_shared<AppConfig>();
    observer = owner; // observer does not increase shared ownership.
} // owner is gone, so AppConfig is deleted.

if (auto config = observer.lock()) {
    // Valid only if the AppConfig still exists.
    // config is a temporary shared_ptr owner in this block.
} else {
    // The AppConfig no longer exists.
}
```

```text
shared_ptr owner ─────► AppConfig object
weak_ptr observer ─ ─ ─► observes it, but does not keep it alive
```

`weak_ptr` is commonly used to break circular ownership:

```text
BAD: shared_ptr A ─► B and shared_ptr B ─► A
     Neither owner count becomes zero, so neither object is deleted.

BETTER: shared_ptr A ─► B and weak_ptr B ─ ─► A
        A disappearing can now allow both objects to be cleaned up.
```

---

### Borrowing: references and raw pointers can still be correct

Smart pointers express **ownership**. A function that merely uses an object should usually not receive ownership.

```cpp
// The function borrows notification. It cannot be null.
void send(Notification& notification) {
    notification.send("Hello");
}

// The function borrows notification. nullptr is allowed and must be checked.
void sendIfPresent(Notification* notification) {
    if (notification != nullptr) {
        notification->send("Hello");
    }
}
```

```text
Owner:    unique_ptr<Notification> ─────► Notification
Borrower: Notification& or Notification* ─► same object

Borrower does NOT call delete.
Owner decides the lifetime.
```

### Raw `new` and `delete`: avoid for ordinary application code

```cpp
// Avoid: manual ownership is easy to get wrong.
EmailNotification* notification = new EmailNotification("smtp.gmail.com");

// If an exception or early return happens before this line, memory leaks.
delete notification;
```

```cpp
// Prefer: cleanup is automatic, including on exceptions and early returns.
auto notification = std::make_unique<EmailNotification>("smtp.gmail.com");
```

Use raw `new` / `delete` mainly in low-level code that implements memory management itself, such as object pools, custom allocators, or placement `new`.

### Quick ownership decision

| Need | Prefer |
| --- | --- |
| Object can live in the current scope | An ordinary object: `T object;` |
| One dynamic owner | `std::unique_ptr<T>` |
| Several real owners | `std::shared_ptr<T>` |
| Observe a `shared_ptr` object without keeping it alive | `std::weak_ptr<T>` |
| Borrow; null is not allowed | `T&` |
| Borrow; null is meaningful | `T*` |

> Memory rule: **Use a raw pointer to borrow, not to own. Use an ordinary object when possible; otherwise prefer `unique_ptr`.**

</details>

---

<details markdown="1">
<summary><strong>5. Type casting: converting one type to another</strong></summary>

A **cast** tells C++ to treat or convert a value as another type.

There are two broad kinds:

```text
Implicit conversion  → C++ performs it automatically.
Explicit conversion  → You clearly request it with syntax such as static_cast<T>(value).
```

Prefer the named C++ casts. Avoid C-style casts such as `(int)value` because they hide which kind of conversion is happening.

### 1. Implicit conversion

C++ automatically converts when it considers the conversion safe or natural enough.

```cpp
int count = 42;

// int becomes double automatically.
double average = count;

// char becomes int automatically.
int asciiCode = 'A'; // 65 in common character encodings.
```

```text
int 42 ──implicit conversion──► double 42.0
```

Be careful: automatic does not always mean lossless.

```cpp
double price = 19.99;

// VALID, but data after the decimal point is discarded.
int wholePrice = price; // 19
```

### 2. Direct initialization and explicit conversion

These both make the destination type visible:

```cpp
double price = 19.99;

int first = static_cast<int>(price); // Explicit conversion: 19
int second(price);                   // Direct initialization: 19
int third{static_cast<int>(price)};  // Brace initialization after explicit conversion: 19
```

For ordinary numeric conversion, prefer `static_cast` because it is easy to search for and shows intent.

```cpp
int completed = 3;
int total = 4;

// INVALID LOGIC: integer division happens first, so result is 0.
double wrongPercentage = completed / total * 100;

// VALID: convert one operand before division.
double percentage = static_cast<double>(completed) / total * 100; // 75.0
```

---

## Named C++ casts

| Cast | Main use | Runtime check? | Typical safety |
| --- | --- | --- | --- |
| `static_cast<T>` | Numeric conversion; safe/known inheritance conversion | No | Usually the default explicit cast |
| `dynamic_cast<T>` | Downcasting polymorphic base pointers/references | Yes | Safe runtime-checked inheritance cast |
| `const_cast<T>` | Add/remove `const` or `volatile` | No | Rare; dangerous if you modify an originally `const` object |
| `reinterpret_cast<T>` | Low-level bit/address reinterpretation | No | Very low-level; avoid in normal application code |

### 3. `static_cast<T>`

Use `static_cast` for conversions C++ can verify at compile time.

#### Numeric conversion

```cpp
double ratio = 2.75;
int roundedDown = static_cast<int>(ratio); // 2; decimal part is discarded.

int count = 500;
double exactEnough = static_cast<double>(count); // 500.0
```

#### Safe inheritance conversion: derived → base (upcast)

```cpp
class Animal {
public:
    virtual ~Animal() = default;
};

class Dog : public Animal {};

Dog dog;

// Always safe: every Dog is also an Animal.
Animal* animal = static_cast<Animal*>(&dog);
```

```text
Dog object
┌─────────────────────────┐
│ Animal part             │  ◄── Animal* can view this safely
│ Dog-specific part       │
└─────────────────────────┘
```

#### Risky inheritance conversion: base → derived (downcast)

```cpp
Dog realDog;
Animal* animal = &realDog;

// Compiles, but static_cast does NOT check the real object at runtime.
Dog* dog = static_cast<Dog*>(animal);
```

This is only safe when you already know `animal` truly points to a `Dog`.

```cpp
Animal plainAnimal;
Animal* animal = &plainAnimal;

// Compiles, but is dangerous: the real object is NOT a Dog.
Dog* dog = static_cast<Dog*>(animal);

// Using dog as a Dog is undefined behavior.
```

For an uncertain downcast, use `dynamic_cast` instead.

```text
             inheritance (“is-a”)
Dog ─────────────────────────────▷ Animal ◁──────────────────────────── Cat
                                      │
                                      │ Animal* / Animal& can refer to
                                      │ either derived object
                                      ▼
                           dynamic_cast<Dog*>
                           succeeds for Dog;
                           returns nullptr for Cat
```

> Rule: **Use `static_cast` for an upcast. Do not use it to “guess” a derived type.**

---

### 4. `dynamic_cast<T>`: checked downcasting

`dynamic_cast` asks at runtime: “Is this object really the requested derived type?”

The base class must be **polymorphic**—it needs at least one virtual function. A virtual destructor is enough.

```cpp
#include <iostream>

class Animal {
public:
    virtual ~Animal() = default; // Makes Animal polymorphic.
};

class Dog : public Animal {
public:
    void bark() { std::cout << "Woof\n"; }
};

class Cat : public Animal {};

void makeDogBark(Animal* animal) {
    // Pointer dynamic_cast returns nullptr if the object is not a Dog.
    if (Dog* dog = dynamic_cast<Dog*>(animal)) {
        dog->bark(); // Valid: C++ confirmed that it is a Dog.
    } else {
        std::cout << "This animal is not a Dog\n";
    }
}

int main() {
    Dog dog;
    Cat cat;

    makeDogBark(&dog); // Woof
    makeDogBark(&cat); // This animal is not a Dog
}
```

```text
Animal* ──► Dog object  → dynamic_cast<Dog*> returns a valid Dog*
Animal* ──► Cat object  → dynamic_cast<Dog*> returns nullptr
```

For references, failure throws `std::bad_cast` instead of returning `nullptr`:

```cpp
#include <typeinfo>

Animal& animal = cat;

try {
    Dog& dog = dynamic_cast<Dog&>(animal); // Throws: cat is not a Dog.
    dog.bark();
} catch (const std::bad_cast&) {
    std::cout << "Not a Dog\n";
}
```

Prefer the pointer form when failure is an expected possibility—it is easy to test for `nullptr`.

---

### 5. `const_cast<T>`: add or remove constness

`const_cast` changes only whether the compiler sees an object as `const`; it does not change the object's real nature.

```cpp
void legacyPrint(char* text); // Imagine an old API that promises not to modify text.

void printMessage(const char* message) {
    // Sometimes used to call a legacy API. Only safe if legacyPrint does not write.
    legacyPrint(const_cast<char*>(message));
}
```

The dangerous mistake:

```cpp
const int original = 10;

int* writable = const_cast<int*>(&original);

// INVALID BEHAVIOR: original was truly declared const.
// *writable = 20; // Undefined behavior.
```

This can be valid only when the original object was not `const`:

```cpp
int original = 10;                 // Not truly const.
const int* readOnly = &original;   // View it as const.

int* writable = const_cast<int*>(readOnly);
*writable = 20;                    // Valid: original itself was mutable.
```

> Rule: **Use `const_cast` only for a well-understood legacy/API boundary. Never use it to modify an object declared `const`.**

---

### 6. `reinterpret_cast<T>`: low-level representation conversion

`reinterpret_cast` can reinterpret an address or bits as an unrelated type. It is for low-level systems code, not ordinary application code.

```cpp
#include <cstdint>

int value = 42;

// Converts a pointer address to an integer-sized value.
std::uintptr_t address = reinterpret_cast<std::uintptr_t>(&value);

// Converts it back to the original pointer type.
int* sameValue = reinterpret_cast<int*>(address);
```

This does **not** safely convert an `int` into a different, unrelated object type.

```cpp
int number = 42;

// Avoid: treating int memory as a double object is undefined behavior.
// double* wrong = reinterpret_cast<double*>(&number);
// std::cout << *wrong;
```

If you need to copy raw bits between same-sized trivially copyable types in C++20, prefer `std::bit_cast` from `<bit>` rather than pointer tricks.

---

### 7. C-style casts: avoid them

```cpp
double price = 19.99;

// Avoid: it could behave like static_cast, const_cast, or reinterpret_cast.
int value = (int)price;

// Prefer: visible and searchable intent.
int betterValue = static_cast<int>(price);
```

Named casts help reviewers understand risk:

```text
static_cast       → ordinary conversion
dynamic_cast      → runtime-checked inheritance conversion
const_cast        → constness changed: inspect carefully
reinterpret_cast  → low-level conversion: inspect very carefully
```

</details>

<details markdown="1">
<summary><strong>6. Type-casting quick decision guide</strong></summary>

| You need to… | Prefer |
| --- | --- |
| Convert between numeric types | `static_cast<T>(value)` |
| Convert derived pointer/reference to base | Usually implicit conversion or `static_cast` |
| Convert base pointer/reference to derived, with uncertainty | `dynamic_cast<T*>` / `dynamic_cast<T&>` |
| Call a legacy API that wrongly lacks `const` | Rare, carefully justified `const_cast` |
| Work with raw addresses/bits in low-level code | Rare `reinterpret_cast`; consider `std::bit_cast` |
| Make code short using `(T)value` | Do not; use a named cast instead |

> Memory rule: **Default to `static_cast` for ordinary explicit conversion. Use `dynamic_cast` when you need C++ to verify the real derived type at runtime. Treat `const_cast` and `reinterpret_cast` as warning signs.**

</details>

---

<details markdown="1">
<summary><strong>7. STL quick reference (C++17)</strong></summary>

The Standard Template Library (STL) gives you ready-made containers, algorithms, iterators, and utilities. In problem-solving code, prefer these over implementing a data structure from scratch unless the problem asks you to.

### Containers and utilities

Each row shows the header, a common declaration/use, typical complexity, and a pitfall. Complexity is for the named core operation; sizes and allocator behavior can affect details.

| Type (header) | Quick example / best for | Typical complexity | Watch out for |
| --- | --- | --- | --- |
| `std::vector<T>` (`<vector>`) | `std::vector<int> v{3, 1, 2};` — default sequence; contiguous and fast random access. | Index `O(1)`; `push_back` amortized `O(1)`; insert/erase in middle `O(n)`. | Growth may reallocate and invalidate pointers, references, and iterators. `reserve(n)` can reduce reallocations. |
| `std::array<T, N>` (`<array>`) | `std::array<int, 3> a{3, 1, 2};` — fixed-size sequence known at compile time. | Index `O(1)`; iteration `O(n)`. | Size cannot grow or shrink; it is still an STL container, unlike a raw array. |
| `std::deque<T>` (`<deque>`) | `std::deque<int> d{2, 3}; d.push_front(1);` — efficient insertion/removal at both ends. | Index `O(1)`; push/pop at either end `O(1)`; middle insert/erase `O(n)`. | Not contiguous like `vector`; iterator invalidation rules differ. |
| `std::list<T>` (`<list>`) | `std::list<int> xs{1, 3}; auto position = xs.begin(); ++position; xs.insert(position, 2);` — stable iterators and frequent insertion/erasure when you already have an iterator. | Insert/erase at a known iterator `O(1)`; search `O(n)`. | No random indexing; `xs[i]` is invalid. Usually `vector` is a better default. |
| `std::forward_list<T>` (`<forward_list>`) | `std::forward_list<int> xs{1, 2};` — singly linked list, niche memory-constrained cases. | Insert/erase after a known iterator `O(1)`; search `O(n)`. | Forward-only; no `size()` member and no random indexing. |
| `std::stack<T>` (`<stack>`) | `std::stack<int> st; st.push(4); int x = st.top(); st.pop();` — last-in, first-out (LIFO). | `top`, `push`, `pop` typically `O(1)`. | `pop()` removes but does not return the value; read `top()` first. Calling `top()`/`pop()` when empty is invalid. |
| `std::queue<T>` (`<queue>`) | `std::queue<int> q; q.push(4); int x = q.front(); q.pop();` — first-in, first-out (FIFO), e.g. BFS. | `front`, `push`, `pop` typically `O(1)`. | `pop()` returns nothing; use `front()` first. Check `empty()` before access. |
| `std::priority_queue<T>` (`<queue>`) | `std::priority_queue<int> maxHeap;` — always access the largest item first by default. | `top` `O(1)`; `push`/`pop` `O(log n)`. | It is a max-heap by default. A min-heap is `std::priority_queue<int, std::vector<int>, std::greater<int>>`. `pop()` does not return the top. |
| `std::set<T>` / `std::multiset<T>` (`<set>`) | `std::set<int> unique{3, 1, 2};` — sorted keys; `multiset` allows duplicates. | Find/insert/erase `O(log n)`. | Iteration is sorted, not insertion order. Use `unordered_set` if ordering is unnecessary and average constant-time lookup is useful. |
| `std::map<K, V>` / `std::multimap<K, V>` (`<map>`) | `std::map<std::string, int> score; score["Ada"] = 10;` — sorted key/value lookup; `multimap` permits repeated keys. | Find/insert/erase `O(log n)`. | `map[key]` inserts a default value when the key is absent. Use `find` or `count` when you only want to check whether a key exists. |
| `std::unordered_set<T>` (`<unordered_set>`) | `std::unordered_set<int> seen; seen.insert(x);` — membership / deduplication without sorted order. | Find/insert/erase average `O(1)`, worst `O(n)`. | Order is unspecified; rehashing can invalidate iterators. Custom key types need a hash function and equality operation. |
| `std::unordered_map<K, V>` (`<unordered_map>`) | `std::unordered_map<int, int> frequency; ++frequency[x];` — frequency tables and fast key lookup. | Find/insert/erase average `O(1)`, worst `O(n)`. | Same hash/rehash caveats. Do not rely on iteration order. |
| `std::pair<A, B>` (`<utility>`) | `std::pair<int, std::string> p{1, "Ada"};` — carry two related values; use `p.first`, `p.second`. | Access either field `O(1)`. | Prefer a small named `struct` if fields have meaningful roles; `first`/`second` can hide intent. |
| `std::tuple<...>` (`<tuple>`) | `auto t = std::make_tuple(7, "Ada", true); auto [id, name, active] = t;` — group a few values of different types (structured bindings are C++17). | `std::get<I>(t)` `O(1)`. | `std::get<0>` is less readable than a named struct for domain data. |
| `std::optional<T>` (`<optional>`) | `std::optional<int> answer = 42; if (answer) use(*answer);` — represent “a value may be absent” (C++17). | Check/access `O(1)`. | Check `has_value()` or the boolean conversion before dereferencing; it is not a container for many values. |
| `std::string` (`<string>`) | `std::string name = "Ada";` — owned text. | See the [string method reference](#stdstring-member-method-reference). | `size()` counts bytes for UTF-8; string insert/erase in the middle is `O(n)`. |

### Algorithms

Most algorithms take an iterator range `[first, last)`: `first` is included and `last` is excluded. They work with many containers, so include `<algorithm>` for these examples and `<numeric>` for numeric algorithms.

| Algorithm (header) | Example / result | Typical complexity | Common pitfall |
| --- | --- | --- | --- |
| `std::sort` (`<algorithm>`) | `std::sort(v.begin(), v.end());` — ascending order. Use a comparator for custom order. | `O(n log n)`. | Requires random-access iterators; use with `vector`, `array`, or `deque`, not `list` (which has its own `.sort()`). Comparator must define a strict weak ordering; avoid `<=`. |
| `std::stable_sort` (`<algorithm>`) | `std::stable_sort(v.begin(), v.end(), byScore);` — equivalent elements keep their prior relative order. | `O(n log n)` comparisons; may use extra memory. | Stability matters only when equal-key items must preserve input order. |
| `std::find` (`<algorithm>`) | `auto it = std::find(v.begin(), v.end(), target);` | `O(n)`. | Not found means `it == v.end()`; don’t dereference `end()`. |
| `std::count` (`<algorithm>`) | `auto n = std::count(v.begin(), v.end(), target);` | `O(n)`. | Counts exact matches; use a predicate with `count_if` for conditions. |
| `std::count_if` (`<algorithm>`) | `auto n = std::count_if(v.begin(), v.end(), isEven);` | `O(n)`. | Predicate should return true for items to count. |
| `std::lower_bound` (`<algorithm>`) | `auto it = std::lower_bound(v.begin(), v.end(), x);` — first item `>= x`. | `O(log n)` comparisons; random-access range also has `O(log n)` iterator movement. | Range must already be sorted using a compatible ordering. In `map`/`set`, prefer the container's `.lower_bound(key)` member. |
| `std::upper_bound` (`<algorithm>`) | `auto it = std::upper_bound(v.begin(), v.end(), x);` — first item `> x`. | `O(log n)` comparisons; random-access range also has `O(log n)` iterator movement. | Requires a sorted range with compatible ordering. |
| `std::binary_search` (`<algorithm>`) | `bool exists = std::binary_search(v.begin(), v.end(), x);` | `O(log n)` comparisons. | Requires sorted data; returns only true/false, not the matching position. |
| `std::min_element` / `std::max_element` (`<algorithm>`) | `auto it = std::max_element(v.begin(), v.end());` | `O(n)`. | Returns an iterator; check for an empty range before dereferencing. |
| `std::reverse` (`<algorithm>`) | `std::reverse(v.begin(), v.end());` | `O(n)`. | Mutates the range in place. |
| `std::accumulate` (`<numeric>`) | `long long sum = std::accumulate(v.begin(), v.end(), 0LL);` | `O(n)`. | The initial value controls the accumulator type: use `0LL` for a `long long` sum to avoid `int` overflow. |
| `std::iota` (`<numeric>`) | `std::iota(v.begin(), v.end(), 0);` — fills with `0, 1, 2, ...`. | `O(n)`. | Mutates every element; the starting value determines the value type. |
| `std::all_of`, `any_of`, `none_of` (`<algorithm>`) | `bool anyNegative = std::any_of(v.begin(), v.end(), [](int x) { return x < 0; });` | `O(n)` worst case; may stop early. | Be clear whether the condition must hold for all, at least one, or none. |
| `std::transform` (`<algorithm>`) | `std::transform(v.begin(), v.end(), v.begin(), [](int x) { return x * 2; });` | `O(n)`. | Output range must have enough space; this example overwrites the existing vector. |
| `std::remove` + `vector::erase` | `v.erase(std::remove(v.begin(), v.end(), x), v.end());` — removes all `x` from a vector. | `O(n)`. | `std::remove` alone does not shrink the container; it moves kept values forward and returns a new logical end. |
| `std::unique` + `vector::erase` | `std::sort(v.begin(), v.end()); v.erase(std::unique(v.begin(), v.end()), v.end());` | `O(n log n)` including sort; `unique` itself `O(n)`. | `unique` removes only consecutive duplicates, so sort first if you want global deduplication. |
| `std::nth_element` (`<algorithm>`) | `std::nth_element(v.begin(), v.begin() + k, v.end());` — places the item for sorted index `k` there; partitions around it. | Average `O(n)`. | The entire vector is not sorted afterward. Ensure `k < v.size()`. |
| `std::next_permutation` (`<algorithm>`) | `std::next_permutation(v.begin(), v.end());` — advances to next lexicographic permutation. | `O(n)` per call. | To enumerate all permutations in order, sort first; it returns false after the last permutation and resets to the first. |

### Common interview recipes

**Frequency counting:**

```cpp
#include <unordered_map>
#include <vector>

std::unordered_map<int, int> frequency;
for (int value : values) {
    ++frequency[value];
}
```

**Sort, then remove duplicates:**

```cpp
#include <algorithm>
#include <vector>

std::sort(values.begin(), values.end());
values.erase(std::unique(values.begin(), values.end()), values.end());
```

**Count occurrences in sorted data:**

```cpp
#include <algorithm>
#include <vector>

auto first = std::lower_bound(values.begin(), values.end(), target);
auto afterLast = std::upper_bound(values.begin(), values.end(), target);
auto count = afterLast - first; // Works here because vector iterators are random access.
```

**Keep the largest `k` values seen:**

```cpp
#include <functional>
#include <queue>
#include <vector>

std::priority_queue<int, std::vector<int>, std::greater<int>> smallestFirst;
for (int value : values) {
    smallestFirst.push(value);
    if (smallestFirst.size() > k) {
        smallestFirst.pop(); // Remove the smallest; the heap retains the largest k.
    }
}
```

### Fast pitfalls checklist

- `map[key]` inserts a default value if `key` was absent; use `.find()` when you only want to check.
- `stack::pop()`, `queue::pop()`, and `priority_queue::pop()` remove an item but do not return it.
- Check `empty()` before reading `top()`, `front()`, or `back()`.
- A vector insertion/resize can invalidate iterators and references; do not keep them across a possible reallocation.
- `lower_bound`, `upper_bound`, and `binary_search` need sorted data.
- Generic `std::sort` cannot sort `std::list`; call `list.sort()` instead.
- An unordered container has no stable iteration order and can have worst-case linear operations.
- Prefer standard algorithms over handwritten loops when they directly express the operation; use a loop when it makes the logic clearer.

</details>
