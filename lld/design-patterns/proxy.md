---
layout: default
title: Proxy
---

# Proxy: LLD quick reference

## What problem does Proxy solve?

Proxy places a lightweight object in front of a real object. The proxy has the **same interface** as the real object, so clients can use either one without changing their code.

```text
Client → Subject interface ← RealSubject
                  ↑
                Proxy
                  │
                  └── decides whether, when, or how to call RealSubject
```

> Core idea: **Proxy is a stand-in for a real object that controls access while preserving the same interface.**

## When to use it

- Defer expensive loading until it is needed (lazy/virtual proxy).
- Check permission before access (protection proxy).
- Cache expensive results (caching proxy).
- Add logging, metrics, retries, or rate limits without editing the real class (smart proxy).
- Make a remote service look like a local object (remote proxy).

The real class being unmodifiable is a common reason to use Proxy, but it is not required. Proxy also keeps policy—such as loading, authorization, or caching—separate from the real object's business job.

---

## C++: lazy-loading image proxy

```cpp
#include <iostream>
#include <memory>
#include <string>
#include <utility>

// SUBJECT: both the real object and its proxy implement this contract.
class Image {
public:
    virtual void display() = 0;
    virtual const std::string& getFileName() const = 0;
    virtual ~Image() = default;
};

// REAL SUBJECT: expensive construction loads image data.
class HighResolutionImage : public Image {
public:
    explicit HighResolutionImage(std::string fileName)
        : fileName_(std::move(fileName)) {
        std::cout << "Loading " << fileName_ << " from disk\n";
    }

    void display() override {
        std::cout << "Displaying " << fileName_ << '\n';
    }

    const std::string& getFileName() const override {
        return fileName_;
    }

private:
    std::string fileName_;
};

// PROXY: lightweight at first; creates the real object only when required.
class ImageProxy : public Image {
public:
    explicit ImageProxy(std::string fileName)
        : fileName_(std::move(fileName)) {}

    // Cheap operation: the proxy already knows the filename.
    const std::string& getFileName() const override {
        return fileName_;
    }

    // Expensive operation: create the real image once, then delegate.
    void display() override {
        if (!realImage_) {
            realImage_ = std::make_unique<HighResolutionImage>(fileName_);
        }
        realImage_->display();
    }

private:
    std::string fileName_;
    std::unique_ptr<HighResolutionImage> realImage_;
};

int main() {
    std::unique_ptr<Image> image = std::make_unique<ImageProxy>("photo.jpg");

    std::cout << image->getFileName() << '\n'; // Valid: no image is loaded.
    image->display();                           // Loads image, then displays it.
    image->display();                           // Reuses the already loaded image.
}
```

```text
At construction:  ImageProxy("photo.jpg") only

getFileName():    proxy answers itself; no disk I/O

first display():  proxy → creates HighResolutionImage → loads → displays
second display(): proxy → existing HighResolutionImage → displays
```

### C++: protection proxy

```cpp
class Document {
public:
    virtual void read() = 0;
    virtual ~Document() = default;
};

class SecretDocument : public Document {
public:
    void read() override { std::cout << "Top-secret content\n"; }
};

class ProtectedDocumentProxy : public Document {
public:
    explicit ProtectedDocumentProxy(bool isAdmin) : isAdmin_(isAdmin) {}

    void read() override {
        if (!isAdmin_) {
            std::cout << "Access denied\n";
            return; // Proxy blocks the request.
        }

        if (!realDocument_) {
            realDocument_ = std::make_unique<SecretDocument>();
        }
        realDocument_->read(); // Proxy forwards an allowed request.
    }

private:
    bool isAdmin_;
    std::unique_ptr<SecretDocument> realDocument_;
};
```

### C++: other useful proxies

| Type | Same interface method | Proxy behavior |
| --- | --- | --- |
| Caching | `weather.get(city)` | Return cache hit; otherwise call real API and save result. |
| Logging | `payment.charge(amount)` | Log before/after forwarding to the real gateway. |
| Rate limiting | `sms.send(number, text)` | Block when the caller exceeds the allowed request count. |
| Remote | `inventory.getStock(sku)` | Convert method call to HTTP/gRPC, then return the response. |

---

## Python: lazy, protection, and caching proxy

```python
from abc import ABC, abstractmethod


class Image(ABC):
    # SUBJECT interface.
    @abstractmethod
    def display(self):
        pass

    @abstractmethod
    def get_file_name(self):
        pass


class HighResolutionImage(Image):
    # REAL SUBJECT: expensive loading happens during construction.
    def __init__(self, file_name):
        self._file_name = file_name
        print(f"Loading {file_name} from disk")

    def display(self):
        print(f"Displaying {self._file_name}")

    def get_file_name(self):
        return self._file_name


class ImageProxy(Image):
    # PROXY: does not create HighResolutionImage immediately.
    def __init__(self, file_name):
        self._file_name = file_name
        self._real_image = None

    def get_file_name(self):
        # Cheap request handled without the real image.
        return self._file_name

    def display(self):
        # Create real object only on the first expensive request.
        if self._real_image is None:
            self._real_image = HighResolutionImage(self._file_name)
        self._real_image.display()


image = ImageProxy("photo.jpg")
print(image.get_file_name())  # photo.jpg; image is not loaded yet.
image.display()               # Loads, then displays.
image.display()               # Displays the cached real object.
```

```python
# PROTECTION PROXY: same read() interface, but checks access first.
class SecretDocument:
    def read(self):
        print("Top-secret content")


class ProtectedDocumentProxy:
    def __init__(self, is_admin):
        self._is_admin = is_admin
        self._real_document = None

    def read(self):
        if not self._is_admin:
            print("Access denied")
            return

        if self._real_document is None:
            self._real_document = SecretDocument()
        self._real_document.read()
```

```python
# CACHING PROXY: same get_weather() method as a real weather service.
class CachingWeatherProxy:
    def __init__(self, real_weather_service):
        self._real_weather_service = real_weather_service
        self._cache = {}

    def get_weather(self, city):
        if city not in self._cache:
            self._cache[city] = self._real_weather_service.get_weather(city)
        return self._cache[city]
```

---

## Go: lazy, protection, and caching proxy

```go
package main

import "fmt"

// SUBJECT interface.
type Image interface {
	Display()
	GetFileName() string
}

// REAL SUBJECT.
type HighResolutionImage struct {
	fileName string
}

func NewHighResolutionImage(fileName string) *HighResolutionImage {
	fmt.Println("Loading", fileName, "from disk")
	return &HighResolutionImage{fileName: fileName}
}

func (image *HighResolutionImage) Display() {
	fmt.Println("Displaying", image.fileName)
}

func (image *HighResolutionImage) GetFileName() string {
	return image.fileName
}

// PROXY: realImage starts as nil, so loading is delayed.
type ImageProxy struct {
	fileName  string
	realImage *HighResolutionImage
}

func NewImageProxy(fileName string) *ImageProxy {
	return &ImageProxy{fileName: fileName}
}

func (proxy *ImageProxy) GetFileName() string {
	return proxy.fileName // Cheap request: no real object required.
}

func (proxy *ImageProxy) Display() {
	if proxy.realImage == nil {
		proxy.realImage = NewHighResolutionImage(proxy.fileName)
	}
	proxy.realImage.Display()
}

func main() {
	var image Image = NewImageProxy("photo.jpg")

	fmt.Println(image.GetFileName()) // No load.
	image.Display()                   // Load and display.
	image.Display()                   // Reuse real image.
}
```

```go
// PROTECTION PROXY: checks a condition before forwarding.
type Document interface {
	Read()
}

type ProtectedDocumentProxy struct {
	isAdmin bool
	real    Document
}

func (proxy *ProtectedDocumentProxy) Read() {
	if !proxy.isAdmin {
		fmt.Println("Access denied")
		return
	}
	proxy.real.Read()
}
```

```go
// CACHING PROXY: interface omitted for brevity; it mirrors GetWeather.
type WeatherService interface {
	GetWeather(city string) string
}

type CachingWeatherProxy struct {
	real  WeatherService
	cache map[string]string
}

func (proxy *CachingWeatherProxy) GetWeather(city string) string {
	if result, found := proxy.cache[city]; found {
		return result
	}

	result := proxy.real.GetWeather(city)
	proxy.cache[city] = result
	return result
}
```

In production Go, protect a shared cache with a mutex if multiple goroutines can use it at once.

---

## Proxy types at a glance

| Type | Question the proxy answers | Example |
| --- | --- | --- |
| Virtual | “Do I need to create/load this yet?” | Large image, video, PDF, database record. |
| Protection | “May this caller do this?” | Admin report, bank account, confidential document. |
| Caching | “Do I already know this result?” | Weather API, product catalog, feature flags. |
| Logging / smart | “What should happen around this call?” | Payments, audit trail, metrics, retries. |
| Rate limiting | “Has this caller used too many requests?” | Login, SMS, API gateway. |
| Remote | “How do I communicate with the remote object?” | Inventory or payment service over HTTP/gRPC. |

## Proxy vs Facade vs Decorator

| Pattern | Interface compared with real object | Main purpose |
| --- | --- | --- |
| Proxy | Same interface | Control access: lazy loading, permission, cache, remote call. |
| Facade | Usually a new, simpler interface | Hide a complex multi-object workflow. |
| Decorator | Same interface | Add behavior/combine features, usually transparently and repeatedly. |

## When to use—or avoid—Proxy

Use Proxy when the client should see the same API but access to the real object needs a policy or optimization.

Avoid it when direct access is already cheap, safe, and clear. Do not add a proxy merely to forward every call without controlling anything.

> Memory rule: **Proxy looks like the real object, but sits in front of it to decide whether, when, or how a call reaches it.**
