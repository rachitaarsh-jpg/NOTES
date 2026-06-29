# Facade Design Pattern in Moqui Framework

## Overview

A **Facade** (pronounced *fa-sād*) is a **software design pattern** that provides a **simple, unified interface** to a larger and more complex subsystem.

Instead of interacting directly with complicated internal components, developers interact with a single, easy-to-use interface called the **Facade**.

---

# Why Do We Need a Facade?

Large software systems often contain many complex components working together.

Without a facade:

- Developers must understand the internal implementation.
- Code becomes difficult to write and maintain.
- Interacting with the system becomes complicated.

A facade hides this complexity and exposes only the operations that developers need.

---

# Real-Life Analogy: Car Dashboard

Imagine driving a car.

You only interact with:

- Steering wheel
- Accelerator
- Brake pedal
- Gear lever

You **do not** directly interact with:

- Engine
- Fuel injection system
- Transmission
- Pistons
- Cooling system

The dashboard acts as a **Facade**.

```
Driver
   │
   ▼
Dashboard (Facade)
   │
   ▼
Engine + Transmission + Fuel System + Electronics
```

You press the accelerator, and the dashboard translates your action into complex operations behind the scenes.

Similarly, a software facade hides complex implementation details.

---

# Facade in Moqui Framework

Moqui is built around the **Facade Design Pattern**.

The framework contains many complex subsystems:

- Database management
- Services
- Screen rendering
- Transactions
- Security
- Caching

Instead of requiring developers to interact with these systems directly, Moqui provides specialized **Facades**.

Each facade acts as a **gateway** to one subsystem.

---

# Major Facades in Moqui

## 1. EntityFacade

### Purpose

Provides a simple interface for working with the database.

Instead of writing SQL manually, developers use the Entity Facade.

Typical operations include:

- Find records
- Create records
- Update records
- Delete records

Common CRUD operations:

```
Create
Read
Update
Delete
```

---

## EntityFacade Architecture

```
Developer
      │
      ▼
EntityFacade
      │
      ▼
Entity Engine
      │
      ▼
Database
```

The developer never communicates directly with the database.

The Entity Facade handles:

- SQL generation
- Database connections
- Transactions
- Entity mapping

---

## entity-facade-xml

In `component.xml`, you may see:

```xml
<entity-facade-xml location="entity/PartyEntities.xml"/>
```

This tells the **Entity Facade**:

> "Load these entity definitions and database configurations."

Similarly,

```xml
<entity-facade-xml location="data/PartyTypeData.xml"/>
```

tells the Entity Facade to load seed data into the database.

---

# 2. ServiceFacade

### Purpose

Provides access to business logic.

Instead of directly calling Java classes, developers invoke services through the Service Facade.

Examples:

- Order processing
- Payment handling
- Email sending
- Inventory updates
- Scheduled jobs
- REST APIs

---

## ServiceFacade Architecture

```
Developer
      │
      ▼
ServiceFacade
      │
      ▼
Business Services
      │
      ▼
Database / External Systems
```

The Service Facade manages:

- Service execution
- Transactions
- Authorization
- Error handling
- Scheduling

---

# 3. ScreenFacade

### Purpose

Handles everything related to the user interface.

Responsibilities include:

- Rendering web pages
- URL routing
- User sessions
- Screen transitions
- Templates
- Widgets

---

## ScreenFacade Architecture

```
Browser
    │
    ▼
ScreenFacade
    │
    ▼
Screens
Templates
Widgets
```

Developers simply define screens, while the Screen Facade handles rendering and navigation.

---

# 4. ExecutionFacade (ec)

### Purpose

Acts as the **master facade** in Moqui.

It provides access to all other facades through a single object.

Usually referenced as:

```groovy
ec
```

---

## Common Access Pattern

```
ec.entity
ec.service
ec.screen
ec.user
ec.message
ec.logger
```

Instead of creating different objects, developers access everything through `ec`.

---

## ExecutionFacade Architecture

```
                ExecutionFacade (ec)
                      │
      ┌───────────────┼───────────────┐
      │               │               │
      ▼               ▼               ▼
 EntityFacade   ServiceFacade   ScreenFacade
      │               │               │
      ▼               ▼               ▼
 Database       Business Logic      UI
```

The Execution Facade acts as a central entry point to the entire framework.

---

# Benefits of Using Facades

## 1. Simplicity

Developers interact with a simple API instead of complex internal systems.

---

## 2. Encapsulation

Internal implementation details remain hidden.

Developers don't need to know:

- SQL generation
- Connection management
- Rendering engine
- Transaction handling

---

## 3. Maintainability

Changes inside the framework do not affect application code as long as the facade interface remains the same.

---

## 4. Reusability

Multiple applications can use the same facade without worrying about implementation details.

---

## 5. Loose Coupling

Application code depends on the facade rather than internal classes, making the system easier to extend and maintain.

---

# How Facades Work Together

```
Developer
      │
      ▼
ExecutionFacade (ec)
      │
 ┌────┼─────┬─────────┐
 │    │     │         │
 ▼    ▼     ▼         ▼
Entity Service Screen User
Facade  Facade Facade Facade
 │       │      │
 ▼       ▼      ▼
DB   Business   UI
```

---

# Example Workflow

Suppose a user places an order.

```
User Clicks "Place Order"

        │

        ▼

ScreenFacade
        │

        ▼

ServiceFacade
        │

        ▼

EntityFacade
        │

        ▼

Database
```

Each facade handles only its specific responsibility while hiding the underlying complexity.

---

# Key Takeaways

- A **Facade** is a design pattern that provides a simple interface to a complex subsystem.
- Moqui uses facades extensively to simplify framework interactions.
- Developers work with facades instead of internal implementation details.
- The major facades are:
  - **EntityFacade** → Database operations
  - **ServiceFacade** → Business logic and services
  - **ScreenFacade** → User interface and web screens
  - **ExecutionFacade (`ec`)** → Central access point to all facades

---

# Quick Revision

| Facade | Responsibility |
|---------|----------------|
| **EntityFacade** | Database operations (CRUD) |
| **ServiceFacade** | Business logic and services |
| **ScreenFacade** | UI rendering and web navigation |
| **ExecutionFacade (`ec`)** | Provides access to all other facades |

---

# One-Line Summary

> **A Facade in Moqui is a simplified gateway that hides the complexity of framework subsystems, allowing developers to interact with databases, services, screens, and other features through clean and unified interfaces like `EntityFacade`, `ServiceFacade`, `ScreenFacade`, and `ExecutionFacade (ec)`.**
