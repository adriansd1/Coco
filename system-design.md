# System Design

## Purpose

This document defines the long-term technical blueprint for **Coco**.

Unlike the architecture document, which describes the high-level
structure of the platform, this document focuses on how the individual
systems interact with one another. It serves as the primary reference
when designing new features or modifying existing ones.

The goal is to ensure that every subsystem fits naturally within the
overall platform without introducing unnecessary complexity.

------------------------------------------------------------------------

## Design Philosophy

Every subsystem should satisfy the following goals:

-   Solve one problem well.
-   Be independently testable.
-   Hide implementation details behind clear interfaces.
-   Avoid direct dependencies on presentation layers.
-   Be reusable across every interface.
-   Be designed for long-term extensibility.

------------------------------------------------------------------------

## C4 Level 1 --- System Context

``` mermaid
graph TD
    User --> CLI
    User --> GUI
    User --> API

    CLI --> Core
    GUI --> Core
    API --> Core

    Core --> OperatingSystem
```

The user never communicates directly with the operating system.

Every request flows through the core engine.

------------------------------------------------------------------------

## C4 Level 2 --- Containers

``` mermaid
graph TD

CLI --> Core
GUI --> Core
API --> Core

Core --> WorkflowEngine
Core --> PluginManager
Core --> Diagnostics
Core --> ProjectDetection
Core --> ProcessRunner
Core --> Configuration

PluginManager --> Plugins
Diagnostics --> OperatingSystem
ProjectDetection --> FileSystem
ProcessRunner --> OperatingSystem
```

Each container owns a specific area of responsibility.

No container should become responsible for unrelated concerns.

------------------------------------------------------------------------

## Core Components

The core engine is composed of several independent subsystems.

### Command Dispatcher

Responsibilities:

-   Receive parsed commands.
-   Locate the correct command implementation.
-   Execute commands.
-   Return structured responses.

The dispatcher should never contain business logic.

------------------------------------------------------------------------

### Workflow Engine

Responsible for executing multi-step workflows.

Future responsibilities include:

-   Sequential execution
-   Parallel execution
-   Conditional branching
-   Retry policies
-   Progress reporting
-   Workflow templates

Every workflow should execute through this engine regardless of whether
it originates from the CLI, GUI, or future cloud services.

------------------------------------------------------------------------

### Plugin Manager

Responsibilities:

-   Discover plugins
-   Load plugins
-   Validate compatibility
-   Register extensions
-   Manage plugin lifecycle

Plugins should never modify the core engine directly.

------------------------------------------------------------------------

### Diagnostics Engine

Responsible for gathering information about the current system.

Examples include:

-   Java installations
-   Git configuration
-   Docker availability
-   Network connectivity
-   Disk usage
-   Memory usage
-   Environment variables
-   Running processes

Each diagnostic should exist as an independent component.

------------------------------------------------------------------------

### Project Detection

Responsible for identifying the user's current project.

Examples include:

-   Maven
-   Gradle
-   Node.js
-   Python
-   Rust
-   Go
-   Docker
-   Git repositories

Detection should rely on project metadata rather than user configuration
whenever possible.

------------------------------------------------------------------------

### Process Runner

The Process Runner is the only subsystem responsible for executing
external programs.

Examples:

-   git
-   docker
-   java
-   npm
-   gradle
-   python

No other subsystem should invoke operating system processes directly.

This centralization provides consistent logging, timeout handling, error
reporting, and platform abstraction.

------------------------------------------------------------------------

### Configuration Manager

Responsible for loading and managing configuration.

Future sources may include:

-   Local configuration
-   Project configuration
-   User configuration
-   Environment variables
-   Cloud configuration

Configuration precedence should always be deterministic and documented.

------------------------------------------------------------------------

## Package Layout

``` text
coco-core/

commands/
actions/
services/
diagnostics/
projects/
workflow/
plugins/
process/
configuration/
filesystem/
events/
models/
exceptions/
utils/
```

Each package represents a single architectural concern.

Packages should not evolve into miscellaneous collections of unrelated
classes.

------------------------------------------------------------------------

## Request Lifecycle

``` mermaid
flowchart TD

User --> CLI
CLI --> Dispatcher
Dispatcher --> Action
Action --> Service
Service --> ProcessRunner
ProcessRunner --> OperatingSystem
OperatingSystem --> Result
Result --> Formatter
Formatter --> CLI
```

The same lifecycle should exist for every interface.

Only the presentation layer changes.

------------------------------------------------------------------------

## Dependency Flow

``` text
Presentation

↓

Dispatcher

↓

Actions

↓

Services

↓

Infrastructure

↓

Operating System
```

Dependencies must always flow downward.

Lower layers should never depend on higher layers.

------------------------------------------------------------------------

## Plugin Lifecycle

``` mermaid
flowchart TD

Startup --> Discover
Discover --> Validate
Validate --> Register
Register --> Initialize
Initialize --> Ready
```

Future plugin updates should integrate into this lifecycle without
modifying the core architecture.

------------------------------------------------------------------------

## Future Systems

The following systems are intentionally excluded from the initial
implementation but should fit naturally into the architecture.

-   AI providers
-   Marketplace
-   Cloud synchronization
-   Remote execution
-   Team collaboration
-   Public SDK
-   Background services
-   Scheduled automation

These systems should extend existing components rather than introducing
parallel architectures.

------------------------------------------------------------------------

## Non-Functional Goals

The platform should prioritize:

-   Fast startup
-   Low memory usage
-   Predictable behavior
-   Cross-platform compatibility
-   Clear logging
-   Strong testability
-   High maintainability
-   Stable public APIs

------------------------------------------------------------------------

## Design Rule

When adding a new feature, always ask:

1.  Does this belong in an existing subsystem?
2.  Can it be reused by every interface?
3.  Does it introduce unnecessary coupling?
4.  Can it be tested independently?
5.  Will it still make sense two years from now?

If the answer to any of these questions is no, the design should be
revisited before implementation.
