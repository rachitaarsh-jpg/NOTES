# component.xml in Moqui Framework

## Overview

The **`component.xml`** file is the **descriptor (or manifest) file** of a Moqui component.

It tells the Moqui engine:

> **"This directory is a valid Moqui component, and here is how it should be loaded and configured."**

Without this file, Moqui will not recognize the folder as a proper component.

---

# Purpose of component.xml

The `component.xml` file acts as the **configuration blueprint** for a component.

It is responsible for:

- Registering the component
- Defining its identity
- Loading entities and seed data
- Managing dependencies
- Configuring web routing
- Controlling component initialization

---

# 1. Defines the Component Identity

Every component has a unique name defined using the `<component>` tag.

Example:

```xml
<component name="party">
```

Here:

- Component name = **party**

This name is used by the framework to identify the component.

Other components can also refer to this name when declaring dependencies.

---

# 2. Explicit Asset Loading

Although Moqui follows many conventions, `component.xml` allows you to explicitly specify which resources should be loaded.

Example:

```xml
<entity-facade-xml location="entity/PartyEntities.xml"/>
```

This tells Moqui to load entity definitions from:

```
entity/
    PartyEntities.xml
```

---

## Loading Entity Definitions

Example:

```xml
<entity-facade-xml location="entity/PartyEntities.xml"/>
```

Purpose:

- Loads entity (database schema) definitions.
- Creates database tables if necessary.
- Registers entities with the Entity Facade.

Think of it as loading the database structure.

---

## Loading Seed Data

Example:

```xml
<entity-facade-xml location="data/PartyTypeData.xml"/>

<entity-facade-xml location="data/ContactMechTypeData.xml"/>
```

Purpose:

Loads initial data into the database.

Examples include:

- Party Types
- Contact Mechanism Types
- Enumeration values
- Default records

This data is often called **seed data** because it initializes the application with essential values.

---

# Example Flow

```
component.xml
       │
       │
       ├──────────────► entity/PartyEntities.xml
       │                   │
       │                   ▼
       │            Database Schema
       │
       ├──────────────► data/PartyTypeData.xml
       │                   │
       │                   ▼
       │            Seed Data
       │
       └──────────────► data/ContactMechTypeData.xml
                           │
                           ▼
                     Initial Records
```

---

# 3. Declares Dependencies

A component may depend on another component.

Example:

```xml
<depends-on name="mantle-usl"/>
```

This tells Moqui:

> Load **mantle-usl** before loading this component.

---

## Why Dependencies Are Important

Suppose:

```
Component A
```

uses services or entities from

```
Component B
```

Then Component A cannot function unless Component B is already loaded.

Dependency declaration ensures:

- Correct loading order
- Proper classpath setup
- Entity availability
- Service availability

---

# Dependency Flow

```
mantle-usl
      │
      ▼
custom-component
```

Moqui loads:

1. mantle-usl
2. custom-component

---

# 4. Mounts Web Screens

If a component provides a web UI, `component.xml` can configure screen routing.

It tells Moqui:

- Which screens belong to this component
- Which URL path should display them

Example concept:

```
URL

/products

        │

        ▼

Screen File

component/screen/Product.xml
```

This is typically configured using the `<screen-facade>` element.

---

# Why component.xml Is Important

Without `component.xml`:

- The folder is not recognized as a Moqui component.
- Entities are not loaded.
- Seed data is not imported.
- Dependencies are ignored.
- Screens are not registered.
- The component cannot be initialized correctly.

---

# Typical Responsibilities

| Responsibility | Description |
|---------------|-------------|
| Register component | Makes the folder a valid Moqui component |
| Define component name | Gives the component a unique identity |
| Load entities | Loads database schema definitions |
| Load seed data | Imports default database records |
| Declare dependencies | Ensures components load in the correct order |
| Configure web screens | Maps URLs to screen definitions |

---

# Architecture Overview

```
                  component.xml
                        │
      ┌─────────────────┼─────────────────┐
      │                 │                 │
      ▼                 ▼                 ▼
 Component Name   Entity Definitions   Seed Data
      │                 │                 │
      ▼                 ▼                 ▼
 Registered        Database Schema   Initial Records
      │
      ▼
 Dependencies
      │
      ▼
 Web Screen Routing
```

---

# Example Structure

```
party/
│
├── component.xml
├── entity/
│      PartyEntities.xml
│
├── data/
│      PartyTypeData.xml
│      ContactMechTypeData.xml
│
├── screen/
│      PartyScreens.xml
│
└── service/
       PartyServices.xml
```

The `component.xml` acts as the entry point that tells Moqui how to load everything inside this component.

---

# Key Takeaways

- `component.xml` is the **manifest (descriptor)** of a Moqui component.
- It registers the component with the framework.
- It defines the component's identity.
- It loads entity definitions and seed data.
- It manages component dependencies.
- It configures web screen routing.
- It serves as the starting point for component initialization.

---

# Quick Revision

| XML Element | Purpose |
|-------------|---------|
| `<component>` | Defines the component name |
| `<entity-facade-xml>` | Loads entity definitions or seed data |
| `<depends-on>` | Declares component dependencies |
| `<screen-facade>` | Configures web screen routing |

---

## One-Line Summary

> **`component.xml` is the configuration blueprint (manifest) of a Moqui component that registers the component, loads its entities and seed data, manages dependencies, and configures how the framework initializes and uses the component.**
