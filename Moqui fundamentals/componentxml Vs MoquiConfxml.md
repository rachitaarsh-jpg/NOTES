# Why Are `component.xml` and `MoquiConf.xml` Separate?

## Overview

A common question when learning Moqui is:

> **Why are some configurations placed in `component.xml` while others (like `<screen-facade>`) are placed in `MoquiConf.xml`?**

The answer lies in the different responsibilities of these two files.

- **`component.xml`** describes the component itself.
- **`MoquiConf.xml`** customizes or overrides the behavior of the entire Moqui framework.

Keeping them separate follows the **Separation of Concerns** principle.

---

# 1. `component.xml` – Component Manifest (Descriptor)

Think of `component.xml` as the **identity card** or **manifest** of a component.

It contains only information that belongs to that specific component.

Typical information includes:

- Component name
- Dependencies
- Entity definitions
- Seed data
- Component metadata

Example:

```xml
<component name="party">

    <depends-on name="mantle-usl"/>

    <entity-facade-xml location="entity/PartyEntities.xml"/>

</component>
```

This tells Moqui:

- My name is **party**
- I depend on **mantle-usl**
- These entities belong to me

Everything here is **local to the component**.

---

# Responsibilities of `component.xml`

```
component.xml

        │

        ├── Component Name

        ├── Dependencies

        ├── Entity Definitions

        └── Seed Data
```

Notice that everything describes **the component itself**.

---

# 2. `MoquiConf.xml` – Global Framework Configuration

`MoquiConf.xml` serves a completely different purpose.

Instead of describing one component, it customizes the behavior of the **entire Moqui framework**.

It is used to:

- Override framework settings
- Configure global behavior
- Inject screens
- Configure caches
- Configure databases
- Configure security
- Configure web routing

---

# Example: Screen Injection

Suppose you write:

```xml
<screen location="component://webroot/screen/webroot/apps.xml">

    <subscreens-item name="partyScreen" .../>

</screen>
```

This does **not** simply affect your own component.

Instead, you are telling Moqui:

> "When the framework loads the global `apps.xml` screen from the `webroot` component, insert my `partyScreen` into its menu."

This changes the behavior of another component.

Therefore, it belongs in **`MoquiConf.xml`**, not `component.xml`.

---

# Local vs Global

## `component.xml`

```
My Component

      │

      ▼

Name

Dependencies

Entities

Seed Data
```

Only describes **itself**.

---

## `MoquiConf.xml`

```
Entire Moqui Framework

          │

          ▼

Routing

Menus

Caches

Database Settings

Security

Screens

Framework Behavior
```

Affects the **whole application**.

---

# Moqui Startup Process

When Moqui starts, it performs several steps.

---

## Step 1 – Discover Components

```
Moqui Starts

      │

      ▼

Reads component.xml files

      │

      ▼

Registers Components

Loads Entities

Loads Seed Data
```

At this stage, Moqui simply learns:

- Which components exist
- Their dependencies
- Their entities

---

## Step 2 – Load Default Framework Configuration

Moqui loads its built-in configuration:

```
MoquiDefaultConf.xml
```

This contains the framework's default settings.

---

## Step 3 – Search for `MoquiConf.xml`

Moqui then searches every component for a:

```
MoquiConf.xml
```

Example:

```
Component A
    │
    └── MoquiConf.xml

Component B
    │
    └── MoquiConf.xml

Component C
    │
    └── MoquiConf.xml
```

---

## Step 4 – Merge Configurations

Moqui combines:

```
MoquiDefaultConf.xml
          +
Component A MoquiConf.xml
          +
Component B MoquiConf.xml
          +
Component C MoquiConf.xml
          │
          ▼
Master Configuration
```

The merged configuration becomes the framework's runtime configuration.

This allows every component to contribute global settings without modifying the framework itself.

---

# Why Not Put Everything in `component.xml`?

Imagine if `component.xml` contained:

- Component identity
- Database configuration
- Cache configuration
- Security rules
- Routing
- Global menus
- Screen overrides

It would become a large file with mixed responsibilities.

This violates the **Single Responsibility Principle (SRP)**.

---

# Separation of Concerns

Moqui keeps the files separate so that each has one clear purpose.

| File | Responsibility |
|------|----------------|
| `component.xml` | Defines **what the component is** |
| `MoquiConf.xml` | Defines **how the framework should behave** |

---

# Schema Validation

Another reason for separation is XML validation.

### `component.xml`

Validated using:

```
component-3.xsd
```

This schema only allows component-specific elements such as:

- Component name
- Dependencies
- Entity loading

---

### `MoquiConf.xml`

Validated using a different configuration schema.

It supports many framework-level settings, including:

- Screen routing
- Cache configuration
- Database configuration
- Security
- Transactions
- Framework overrides

Keeping separate schemas makes validation simpler and prevents invalid configurations.

---

# Architecture Overview

```
                Moqui Framework

                      │

          ┌───────────┴───────────┐

          │                       │

   component.xml             MoquiConf.xml

          │                       │

          ▼                       ▼

   Component Info         Framework Configuration

          │                       │

          ▼                       ▼

Name                  Screen Routing

Dependencies          Cache Settings

Entities              Database Config

Seed Data             Security

                       Global Overrides
```

---

# Real-Life Analogy

Imagine a company.

Every employee has an **ID card**.

The ID card contains:

- Name
- Employee ID
- Department

It does **not** contain company-wide policies.

The company's policy handbook contains:

- Office timings
- Security rules
- Leave policy
- Building access

Similarly:

```
Employee ID Card
        =
component.xml

Company Policy Manual
        =
MoquiConf.xml
```

One describes the individual.

The other controls the organization.

---

# Key Takeaways

- `component.xml` is the **manifest** of a component.
- It contains only component-specific information.
- `MoquiConf.xml` configures or overrides the global framework.
- Screen routing belongs in `MoquiConf.xml` because it changes application-wide behavior.
- During startup:
  1. Moqui loads all `component.xml` files.
  2. Loads `MoquiDefaultConf.xml`.
  3. Finds every component's `MoquiConf.xml`.
  4. Merges them into one master configuration.
- Keeping the two files separate follows the **Separation of Concerns** and **Single Responsibility Principle**, making the framework modular, maintainable, and easier to validate.

---

# Quick Revision

| `component.xml` | `MoquiConf.xml` |
|-----------------|-----------------|
| Component manifest | Framework configuration |
| Defines component identity | Overrides framework behavior |
| Dependencies | Screen routing |
| Entity definitions | Cache settings |
| Seed data | Database configuration |
| Component-specific | Global application behavior |

---

## One-Line Summary

> **`component.xml` defines what a component is, while `MoquiConf.xml` defines how the overall Moqui framework should behave. During startup, Moqui loads all component manifests first, then merges every `MoquiConf.xml` with the default framework configuration to create a single master runtime configuration.**
