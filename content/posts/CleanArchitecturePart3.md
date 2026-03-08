---
title: "Clean Architecture Part 3: Why a Tech Lead Must Care About SOLID"
date: 2025-02-20T09:03:20-05:00
draft: false
---

In the previous parts of this series, we explored what Clean Architecture is and how SOLID principles shape the design of individual components. In this post, we go one level up: why should a **Tech Lead** specifically care about SOLID? And when the pressure of deadlines and business demands pushes back, why should the answer almost always be to protect the architecture?

This post draws heavily from Robert C. Martin's book *Clean Architecture: A Craftsman's Guide to Software Structure and Design*.

---

## The Two Values of Software

Martin opens *Clean Architecture* with a powerful distinction: software has two values — **behavior** and **structure**.

- **Behavior** is what the software does today: the features, the bug fixes, the things stakeholders can see and touch.
- **Structure** is how the software is built: how easy it is to change, extend, and maintain over time.

Most organizations only measure behavior. Deadlines are set around features. Bugs are tracked by their user impact. Structure is invisible — until it isn't, and by then it's too late.

A Tech Lead's most important job is to **keep structure on the table** when everyone else is only talking about behavior.

---

## Why SOLID Is an Architectural Concern

SOLID principles are often taught as coding guidelines — things individual developers apply when writing a class or a module. But Martin argues they operate at a higher level too: they define **how components and systems should relate to each other**.

Here is a quick recap of the five principles:

- **S — Single Responsibility Principle (SRP):** A module should have one, and only one, reason to change.
- **O — Open/Closed Principle (OCP):** A system should be open for extension but closed for modification.
- **L — Liskov Substitution Principle (LSP):** Components that depend on an abstraction should work correctly with any implementation of that abstraction.
- **I — Interface Segregation Principle (ISP):** Do not force components to depend on things they do not use.
- **D — Dependency Inversion Principle (DIP):** High-level policy (business rules) should not depend on low-level details (databases, frameworks, UI).

When a Tech Lead ignores these principles, the codebase slowly becomes a system where **every change is dangerous**, because everything depends on everything else.

---

## The Trade-offs a Tech Lead Faces

Let's look at real situations where SOLID comes under pressure — and what the right call usually is.

---

### Trade-off 1: "Just add it to the existing class, it's faster"

**The situation:** A new requirement arrives. A developer proposes adding the new logic to an existing class because it's already handling something similar.

**The SOLID tension:** This violates SRP. The class now has two reasons to change — the original responsibility and the new one. Over time, this class becomes a "god object" that everyone is afraid to touch.

**The right lean:** A Tech Lead should push for a new, focused class or module, even if it takes an extra hour. The cost of that hour is trivial compared to the cost of debugging a bloated class six months later.

---

### Trade-off 2: "We need this feature shipped by Friday, skip the abstraction"

**The situation:** The business wants a feature fast. Someone suggests hardcoding the logic directly against a specific database or third-party API to save time.

**The SOLID tension:** This violates DIP. The business logic (high-level policy) becomes tightly coupled to an external tool (low-level detail). When that tool changes — and it always does — the business logic breaks too.

**The right lean:** Define a simple interface that the business logic depends on, and implement the concrete connection separately. This does not take much longer and keeps the door open for change. Martin calls this the **Plugin Architecture** — your core system should not know or care what database, framework, or service it is connected to.

---

### Trade-off 3: "Let's just modify this base class, it's quicker than extending it"

**The situation:** A new use case is slightly different from an existing one. The temptation is to add a conditional inside the existing logic to handle both cases.

**The SOLID tension:** This violates OCP. Every time you modify existing, working code to handle a new case, you risk breaking the behavior it already had. Tests that were passing may now fail. Systems that were stable become unpredictable.

**The right lean:** Extend through new implementations or subclasses, not by modifying existing ones. Stable code should stay stable. New behavior should live in new code.

---

### Trade-off 4: "Let's reuse this interface, it already has most of what we need"

**The situation:** An existing interface covers 80% of what a new component needs. Someone suggests just implementing the full interface and leaving the unused methods empty or throwing exceptions.

**The SOLID tension:** This violates ISP. Components that implement interfaces they do not fully use create false contracts. Callers may depend on methods that do not actually work, leading to runtime failures that are hard to trace.

**The right lean:** Create a smaller, purpose-built interface for the new component. Interfaces are cheap to define. The confusion caused by a broken contract is expensive to debug.

---

### Trade-off 5: "Our performance benchmarks require tight coupling"

**The situation:** A performance optimization is proposed that requires one component to call another directly, bypassing abstractions. "We cannot afford the overhead of an interface here."

**The SOLID tension:** This is one of the hardest trade-offs because performance is real and measurable. But in most systems, the performance difference between a direct call and an interface-based call is negligible.

**The right lean:** Measure first. Premature optimization is one of the most common reasons architecture gets sacrificed for no real gain. If the measurement confirms a real bottleneck, then make the trade-off consciously and document it as a known exception, not a general pattern.

---

## Why You Should Almost Always Lean Toward Structure

Martin makes a strong argument in *Clean Architecture*: **the urgency of behavior is almost always overstated, and the importance of structure is almost always underestimated**.

Features that are "urgent" today may be replaced or removed in six months. But the structural damage done by cutting corners accumulates. Every shortcut increases what Martin calls **technical debt** — the extra work required in the future because of the shortcuts taken today.

There is a useful mental model for this: imagine the codebase as a house. Features are the furniture — you can always move furniture. Structure is the walls and the foundation. If you start tearing down walls every time you need a new room, eventually the house becomes uninhabitable.

A Tech Lead's job is to keep the house standing while the team keeps rearranging the furniture.

---

## Key Takeaways

- SOLID is not just a coding guideline — it is an architectural philosophy that keeps systems changeable over time.
- A Tech Lead is responsible for maintaining the **structural value** of software, even when business pressure focuses only on behavioral value.
- Most "faster" shortcuts violate one or more SOLID principles and create compounding costs down the road.
- When in doubt, lean toward structure. Features can be delayed. A crumbling architecture cannot be easily rebuilt.

---

*This is Part 3 of the Clean Architecture series. In Part 1, we covered the role of a Tech Lead. In Part 2, we explored SOLID principles in depth.*