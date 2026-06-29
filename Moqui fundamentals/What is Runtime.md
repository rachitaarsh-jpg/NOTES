# Moqui Runtime Directory

## Overview

The **`runtime`** directory is a fundamental architectural concept in the **Moqui Framework** (which **Maarg** is built upon).

It provides a clear separation between:

- **Framework (Engine):** Core Moqui source code and libraries.
- **Runtime (Workspace):** Custom applications, configurations, logs, databases, and environment-specific resources.

This separation makes development, deployment, and framework upgrades much easier and safer.

---

# Why Do We Need the Runtime Directory?

## 1. Separation of Concerns

The `framework` directory contains the core Moqui engine and should remain mostly unchanged.

The `runtime` directory contains everything related to your application, including:

- Custom components
- Business logic
- Configuration files
- Logs
- Database
- Additional libraries

### Benefits

- Keeps the framework clean.
- Prevents accidental modification of core code.
- Makes customization easier.

---

## 2. Easy Framework Upgrades

Since all application-specific code lives inside `runtime`, upgrading Moqui becomes straightforward.

### Without Runtime Separation

Upgrading the framework could overwrite:

- Custom code
- Configuration
- Database settings

### With Runtime Separation

Only the framework is upgraded.

Your:

- Components
- Configurations
- Database
- Logs

remain untouched.

---

## 3. Environment Management

The same Moqui framework can be used with multiple runtime directories.

Example:

```
Framework
    │
    ├── Runtime (Development)
    ├── Runtime (Testing)
    ├── Runtime (Staging)
    └── Runtime (Production)
```

Each runtime has its own:

- Database configuration
- Logs
- Components
- Environment settings

This makes deployment much simpler.

---

# Runtime Directory Structure

Example path:

```
/home/rachitaarsh/moqui-projects/moqui-framework/runtime
```

Inside this directory are several important folders.

---

# 1. component/

## Purpose

The **most important folder for developers.**

Contains:

- Custom applications
- Plugins
- Business components
- Business logic

Example:

```
runtime/
    component/
        mantle-shopify-connector/
        my-custom-app/
```

Your custom development is usually done here.

---

# 2. base-component/

## Purpose

Contains foundational components provided by Moqui.

Examples include:

- Mantle UDM
- Mantle USL
- Webroot
- Other core business components

These components provide reusable functionality that custom components build upon.

Think of it as:

```
Framework
      ↓
Base Components
      ↓
Custom Components
```

---

# 3. conf/

## Purpose

Stores environment-specific configuration files.

Examples:

```
MoquiDevConf.xml
MoquiProductionConf.xml
```

Used for configuring:

- Database connections
- Cache settings
- Elasticsearch
- Transaction settings
- Server properties

Different environments can use different configuration files.

---

# 4. log/

## Purpose

Stores application log files.

Example:

```
runtime/log/moqui.log
```

Logs include:

- Errors
- Warnings
- Startup information
- SQL logs
- Debug information

Useful for troubleshooting and monitoring.

---

# 5. db/

## Purpose

Stores database files when using the embedded H2 database.

Example:

```
runtime/db/
```

If another database like MySQL or PostgreSQL is used, this folder may not contain application data.

---

# 6. lib/

## Purpose

Stores custom Java libraries (`.jar` files).

These libraries become available globally to runtime components.

Example:

```
runtime/lib/

    custom-library.jar

    payment-sdk.jar
```

Useful for:

- Third-party libraries
- Custom Java utilities
- External SDKs

---

# 7. tmp/

## Purpose

Temporary working directory.

Used by the framework for:

- Temporary files
- Cached data
- Processing intermediate resources

Contents can usually be safely regenerated.

---

# 8. txlog/

## Purpose

Stores transaction logs.

Used by **Atomikos Transaction Manager** for:

- Transaction recovery
- Distributed transaction management
- Crash recovery

Important for maintaining transaction consistency.

---

# Framework vs Runtime

| Framework | Runtime |
|-----------|----------|
| Core Moqui engine | Your application workspace |
| Java source code | Components |
| Internal libraries | Configurations |
| Rarely modified | Frequently modified |
| Can be upgraded | Preserved during upgrades |

---

# Architecture Overview

```
                Moqui Framework
                (Core Engine)
                       │
                       │
          ┌────────────┴────────────┐
          │                         │
     Core Libraries          Runtime Directory
                                   │
        ├──────────────┬───────────────┬──────────────┐
        │              │               │              │
    component/     conf/          log/          db/
        │
   Custom Apps
   Business Logic
   Plugins
```

---

# Key Takeaways

- `runtime` is the **workspace** for a Moqui application.
- It keeps **custom code separate** from the core framework.
- Makes **framework upgrades safe** without affecting application code.
- Supports **multiple environments** (development, staging, production).
- Stores:
  - Components
  - Configuration
  - Logs
  - Database
  - Libraries
  - Temporary files
  - Transaction logs

---

# Quick Revision

| Folder | Purpose |
|---------|----------|
| `component/` | Custom applications and business logic |
| `base-component/` | Core business components provided by Moqui |
| `conf/` | Environment-specific configuration files |
| `log/` | Application logs |
| `db/` | Embedded H2 database files |
| `lib/` | Custom Java `.jar` libraries |
| `tmp/` | Temporary files |
| `txlog/` | Transaction logs (Atomikos) |

---

## One-Line Summary

> **The `runtime` directory is the application workspace of Moqui, containing all custom components, configurations, logs, databases, and environment-specific resources while keeping them separate from the core framework for easier maintenance and upgrades.**
