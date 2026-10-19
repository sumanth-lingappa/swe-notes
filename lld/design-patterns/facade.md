---
layout: default
title: Facade
---

# Facade: LLD interview quick reference

## What problem does Facade solve?

One business operation can require several subsystem calls in a specific order.

For a deployment, a client may need to:

```text
pull source code → build artifact → run tests → deploy artifact
```

Without a Facade, every caller must know all subsystem classes, call order, configuration, and error handling.

```text
Client ──► VersionControlSystem
       ──► BuildSystem
       ──► TestingFramework
       ──► DeploymentTarget
```

A Facade provides one high-level operation and coordinates the subsystems internally.

```text
Client ──► DeploymentFacade::deployApplication()
                     │
                     ├──► VersionControlSystem
                     ├──► BuildSystem
                     ├──► TestingFramework
                     └──► DeploymentTarget
```

> Interview sentence: **Facade provides a simple, unified interface to a complex subsystem.**

---

## Is a Facade required to be a class?

No. Facade is a design idea, not a requirement to create a class.

Use either:

1. A **function** when one stateless workflow is small and clear.
2. A **Facade class/object** when several related operations share the same dependencies, configuration, or state.

Both hide subsystem coordination from the caller.

---

## C++

### Option A: free function Facade

Good for one small workflow. The caller supplies all dependencies for every call.

```cpp
#include <iostream>
#include <string>

class VersionControlSystem {
public:
    void pullLatestCode(const std::string& branch) {
        std::cout << "Pulled " << branch << '\n';
    }
};

class BuildSystem {
public:
    std::string build() {
        std::cout << "Built application\n";
        return "app.jar";
    }
};

class TestingFramework {
public:
    bool runAllTests() {
        std::cout << "Tests passed\n";
        return true;
    }
};

class DeploymentTarget {
public:
    void deploy(const std::string& artifact, const std::string& server) {
        std::cout << "Deployed " << artifact << " to " << server << '\n';
    }
};

// This free function is a Facade: callers make one high-level request.
bool deployApplication(
    VersionControlSystem& vcs,
    BuildSystem& buildSystem,
    TestingFramework& tests,
    DeploymentTarget& target,
    const std::string& branch,
    const std::string& server)
{
    vcs.pullLatestCode(branch);
    std::string artifact = buildSystem.build();

    if (!tests.runAllTests()) {
        return false; // Do not deploy a failing build.
    }

    target.deploy(artifact, server);
    return true;
}

int main() {
    VersionControlSystem vcs;
    BuildSystem buildSystem;
    TestingFramework tests;
    DeploymentTarget target;

    // Valid: one high-level call, but dependencies are repeated as arguments.
    deployApplication(vcs, buildSystem, tests, target, "main", "production");
}
```

### Option B: `DeploymentFacade` class

Good when several operations use the same dependencies.

```cpp
#include <string>

class DeploymentFacade {
public:
    // Dependencies are injected once when the facade is created.
    // References mean the facade borrows them; it does not own or delete them.
    DeploymentFacade(
        VersionControlSystem& vcs,
        BuildSystem& buildSystem,
        TestingFramework& tests,
        DeploymentTarget& target)
        : vcs_(vcs), buildSystem_(buildSystem), tests_(tests), target_(target) {}

    bool deployApplication(const std::string& branch, const std::string& server) {
        vcs_.pullLatestCode(branch);
        std::string artifact = buildSystem_.build();

        if (!tests_.runAllTests()) {
            return false;
        }

        target_.deploy(artifact, server);
        return true;
    }

    // A second high-level operation can reuse the same subsystems.
    bool deployHotfix(const std::string& branch, const std::string& server) {
        // Could run a smaller, expedited test suite here.
        return deployApplication(branch, server);
    }

private:
    VersionControlSystem& vcs_;
    BuildSystem& buildSystem_;
    TestingFramework& tests_;
    DeploymentTarget& target_;
};

// Usage:
// DeploymentFacade deployment(vcs, buildSystem, tests, target);
// deployment.deployApplication("main", "production");
// deployment.deployHotfix("release/urgent-fix", "production");
```

### C++: valid and invalid thinking

```cpp
// VALID: client only knows the high-level operation.
deployment.deployApplication("main", "production");

// Avoid: every client repeats the low-level sequence.
vcs.pullLatestCode("main");
std::string artifact = buildSystem.build();
if (tests.runAllTests()) {
    target.deploy(artifact, "production");
}
```

The second code can compile, but it is poor design if several clients repeat it. A future quality scan, rollback rule, or notification must then be added everywhere.

### C++ choice guide

| Choose | When |
| --- | --- |
| Free function | One short, stateless workflow; parameters remain readable. |
| Facade class | Multiple operations share dependencies/configuration; you want dependency injection and a stable API. |

---

## Python

Python can use either a module-level function or an object. The idea is the same.

### Option A: function Facade

```python
class VersionControlSystem:
    def pull_latest_code(self, branch):
        print(f"Pulled {branch}")


class BuildSystem:
    def build(self):
        print("Built application")
        return "app.jar"


class TestingFramework:
    def run_all_tests(self):
        print("Tests passed")
        return True


class DeploymentTarget:
    def deploy(self, artifact, server):
        print(f"Deployed {artifact} to {server}")


# A function can be a Facade.
def deploy_application(vcs, build_system, tests, target, branch, server):
    vcs.pull_latest_code(branch)
    artifact = build_system.build()

    if not tests.run_all_tests():
        return False

    target.deploy(artifact, server)
    return True


# Dependencies are created by the application/composition root.
vcs = VersionControlSystem()
build_system = BuildSystem()
tests = TestingFramework()
target = DeploymentTarget()

deploy_application(vcs, build_system, tests, target, "main", "production")
```

### Option B: class Facade

```python
class DeploymentFacade:
    # Dependencies are stored once instead of passed to every call.
    def __init__(self, vcs, build_system, tests, target):
        self._vcs = vcs
        self._build_system = build_system
        self._tests = tests
        self._target = target

    def deploy_application(self, branch, server):
        self._vcs.pull_latest_code(branch)
        artifact = self._build_system.build()

        if not self._tests.run_all_tests():
            return False

        self._target.deploy(artifact, server)
        return True

    def deploy_hotfix(self, branch, server):
        return self.deploy_application(branch, server)


# Created once in the composition root, then used by clients.
deployment = DeploymentFacade(vcs, build_system, tests, target)
deployment.deploy_application("main", "production")
```

### Python pros and cons

| Option | Pros | Cons |
| --- | --- | --- |
| Function | Small and direct; no extra object | Dependency list can grow; related workflows are scattered across functions. |
| Class | Groups related operations and dependencies; easy to replace in tests | More code; unnecessary for one tiny workflow. |

---

## Go

Go often uses a struct that holds dependencies, with methods as the Facade. A function is also valid for a small workflow.

### Option A: function Facade

```go
package main

import "fmt"

type VersionControlSystem struct{}

func (v VersionControlSystem) PullLatestCode(branch string) {
	fmt.Println("Pulled", branch)
}

type BuildSystem struct{}

func (b BuildSystem) Build() string {
	fmt.Println("Built application")
	return "app.jar"
}

type TestingFramework struct{}

func (t TestingFramework) RunAllTests() bool {
	fmt.Println("Tests passed")
	return true
}

type DeploymentTarget struct{}

func (d DeploymentTarget) Deploy(artifact string, server string) {
	fmt.Println("Deployed", artifact, "to", server)
}

// A function can be a Facade.
func DeployApplication(
	vcs VersionControlSystem,
	buildSystem BuildSystem,
	tests TestingFramework,
	target DeploymentTarget,
	branch string,
	server string,
) bool {
	vcs.PullLatestCode(branch)
	artifact := buildSystem.Build()

	if !tests.RunAllTests() {
		return false
	}

	target.Deploy(artifact, server)
	return true
}

func main() {
	vcs := VersionControlSystem{}
	buildSystem := BuildSystem{}
	tests := TestingFramework{}
	target := DeploymentTarget{}

	DeployApplication(vcs, buildSystem, tests, target, "main", "production")
}
```

### Option B: struct Facade

```go
type DeploymentFacade struct {
	// Dependencies are stored once in the facade.
	vcs         VersionControlSystem
	buildSystem BuildSystem
	tests       TestingFramework
	target      DeploymentTarget
}

func NewDeploymentFacade(
	vcs VersionControlSystem,
	buildSystem BuildSystem,
	tests TestingFramework,
	target DeploymentTarget,
) *DeploymentFacade {
	return &DeploymentFacade{vcs, buildSystem, tests, target}
}

func (facade *DeploymentFacade) DeployApplication(branch string, server string) bool {
	facade.vcs.PullLatestCode(branch)
	artifact := facade.buildSystem.Build()

	if !facade.tests.RunAllTests() {
		return false
	}

	facade.target.Deploy(artifact, server)
	return true
}

func (facade *DeploymentFacade) DeployHotfix(branch string, server string) bool {
	return facade.DeployApplication(branch, server)
}

// Usage:
// deployment := NewDeploymentFacade(vcs, buildSystem, tests, target)
// deployment.DeployApplication("main", "production")
```

### Go interview note

For production code, define small interfaces such as `CodePuller`, `Builder`, `TestRunner`, and `Deployer` for the dependencies. Then a test can provide fake implementations. The Facade still coordinates them in one high-level method.

---

## Function Facade vs class/object Facade

| Question | Function | Class/object |
| --- | --- | --- |
| Does it still count as a Facade? | Yes | Yes |
| Where do dependencies live? | Function parameters | Object fields |
| Best for | One simple workflow | Several related workflows |
| Configuration/state | Must be passed in or global | Stored once in the object |
| Testing | Pass fake parameters | Inject fakes at construction; optionally mock the facade itself |
| Cost | Minimal boilerplate | More structure and code |

## Facade pros and cons

### Benefits

- Client code is simpler: one meaningful call instead of many subsystem calls.
- Clients are less coupled to subsystem APIs and call order.
- Workflow changes happen in one place.
- Error handling, logging, metrics, retries, and validation can be centralized.
- Subsystems remain available for advanced callers when needed.

### Costs and risks

- A giant facade can become a “god object” with too many unrelated methods.
- Hiding every subsystem API may limit advanced use cases.
- A facade does not replace good subsystem design; it only simplifies common use.
- A one-line facade around one method adds no value.

## When to use—or avoid—Facade

Use it when a common task coordinates several classes and clients should not need to learn the details.

Avoid it when the client truly needs detailed subsystem control, or when the “facade” merely forwards one call without reducing complexity.

> Memory rule: **Facade hides a workflow, not necessarily every subsystem. A function is enough for one simple workflow; use a class when related workflows share dependencies.**

## Other common Facade examples

| Facade | Subsystems it coordinates | High-level methods |
| --- | --- | --- |
| Home theater | Projector, lights, amplifier, streaming player | `watchMovie()`, `endMovie()` |
| Checkout | Inventory, payment, shipping, email | `placeOrder()` |
| Travel booking | Flights, hotels, payments, itinerary | `bookTrip()` |
| Video publishing | Upload, compression, storage, metadata, notifications | `publishVideo()` |
