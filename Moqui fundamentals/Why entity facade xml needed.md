# Why `<entity-facade-xml>` Is Required in `component.xml`

## Overview

The `<entity-facade-xml>` element tells the **Entity Facade** which **Entity XML** and **Data XML** files should be loaded when the application starts.

If these entries are removed from `component.xml`, Moqui **does not load** those files.

As a result, the application will fail whenever it tries to use the entities or seed data defined in them.

---

# What Happens During Startup?

When Moqui starts, it performs the following steps:

```
Application Starts
        │
        ▼
Reads component.xml
        │
        ▼
Looks for <entity-facade-xml> entries
        │
        ▼
Loads Entity XML files
        │
        ▼
Registers Entities
        │
        ▼
Loads Seed Data
```

Only the files explicitly listed in `component.xml` are processed.

---

# Why Is `<entity-facade-xml>` Necessary?

Unlike some frameworks that automatically scan folders for database models, **Moqui requires explicit registration of Entity XML files**.

Moqui follows a hybrid approach:

- **Convention over Configuration** for many resources (such as Groovy scripts and screen files).
- **Explicit Configuration** for database entities and seed data.

This ensures the framework knows exactly which database definitions and data files to load.

---

# Example

Suppose your component contains:

```
component/
│
├── component.xml
│
├── entity/
│      PartyEntities.xml
│
└── data/
       PartyTypeData.xml
```

Your `component.xml` should include:

```xml
<entity-facade-xml location="entity/PartyEntities.xml"/>

<entity-facade-xml location="data/PartyTypeData.xml"/>
```

These entries instruct the Entity Facade to load both the entity definitions and the seed data.

---

# What If You Remove These Entries?

Consider this `component.xml`:

```xml
<component name="party">

    <!-- entity-facade-xml removed -->

</component>
```

Although the files still exist in the project, Moqui ignores them because they are no longer registered.

```
component.xml
        │
        ▼
No entity-facade-xml found
        │
        ▼
Entity XML ignored
        │
        ▼
Data XML ignored
```

---

# Consequence 1: Database Tables Are Not Created

Entity XML files define the database schema.

Example:

```
PartyEntities.xml
        │
        ▼
Defines Party Entity
        │
        ▼
Creates PARTY table
```

If the file is never loaded:

- The entity is never registered.
- The corresponding database table is not created (especially during a fresh setup).

---

# Consequence 2: EntityNotFoundException

Suppose your code contains:

```groovy
ec.entity.find("party.Party").list()
```

Normally:

```
EntityFacade
      │
      ▼
Finds Party Entity
      │
      ▼
Queries Database
```

If `PartyEntities.xml` was not loaded:

```
EntityFacade
      │
      ▼
Looks for Party Entity
      │
      ▼
Entity Not Registered
      │
      ▼
EntityNotFoundException
```

The framework does not know that the `party.Party` entity exists.

---

# Consequence 3: Seed Data Is Not Loaded

Data XML files usually contain **initial (seed) data** such as:

- Party Types
- Contact Mechanism Types
- Enumerations
- Status values
- Default records
- Configuration data

Example:

```xml
<entity-facade-xml location="data/PartyTypeData.xml"/>
```

If this entry is removed:

- Seed data is never imported.
- Required lookup values are missing.
- Dropdowns may appear empty.
- Default configuration records may not exist.

---

# Startup Comparison

## With `<entity-facade-xml>`

```
Application Starts
        │
        ▼
Reads component.xml
        │
        ▼
Loads Entity XML
        │
        ▼
Registers Entities
        │
        ▼
Creates Tables
        │
        ▼
Loads Seed Data
        │
        ▼
Application Works
```

---

## Without `<entity-facade-xml>`

```
Application Starts
        │
        ▼
Reads component.xml
        │
        ▼
Entity XML Ignored
        │
        ▼
Entities Not Registered
        │
        ▼
No Tables
        │
        ▼
No Seed Data
        │
        ▼
Runtime Errors
```

---

# Why Doesn't Moqui Auto-Scan Entity Files?

Many frameworks automatically scan directories for model or entity classes.

Moqui intentionally avoids this for Entity XML files because:

- It provides explicit control over what gets loaded.
- It avoids unnecessary scanning during startup.
- It allows developers to control loading order.
- It prevents accidental loading of unused or incomplete entity definitions.

---

# Summary of Effects

| If `<entity-facade-xml>` is Removed | Result |
|-------------------------------------|--------|
| Entity XML not loaded | Entities are not registered |
| Fresh setup | Database tables are not created |
| Entity access | `EntityNotFoundException` is thrown |
| Data XML not loaded | Seed data is missing |
| Lookup values | Dropdowns and enums may be empty |
| Application | Features depending on those entities fail |

---

# Key Takeaways

- `component.xml` is responsible for registering Entity XML and Data XML files.
- Moqui **does not automatically scan** Entity XML files.
- `<entity-facade-xml>` explicitly tells the Entity Facade what to load.
- Removing these entries causes:
  - Entities to remain unregistered.
  - Database tables to be absent on fresh installations.
  - `EntityNotFoundException` when accessing entities.
  - Seed data to be missing from the database.
- Therefore, **every Entity XML and Data XML file that should be used by the application must be listed in `component.xml`.**

---

# Quick Revision

| Element | Purpose |
|---------|---------|
| `<entity-facade-xml location="entity/..."/>` | Loads entity definitions (database schema) |
| `<entity-facade-xml location="data/..."/>` | Loads seed/initial data |
| Missing entry | File is ignored by Moqui |
| Result | No entities, no tables, no seed data, runtime errors |

---

## One-Line Summary

> **`<entity-facade-xml>` acts as an explicit registration mechanism for Entity XML and Data XML files. If these entries are omitted from `component.xml`, Moqui ignores the files entirely, resulting in missing entities, absent database tables, missing seed data, and runtime errors such as `EntityNotFoundException`.**
