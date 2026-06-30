# Architecture

## Overview

**Coco** is built around a simple architectural principle:

> **One engine. Many interfaces.**

Every feature is implemented once inside the core engine. The
command-line interface, graphical desktop application, APIs, cloud
services, and future integrations all invoke the same core
functionality.

This approach minimizes duplication, improves maintainability, and
guarantees consistent behavior across every interface.

------------------------------------------------------------------------

## High-Level Architecture

``` mermaid
graph TD
    User --> CLI
    User --> GUI
    User --> API

    CLI --> Core
    GUI --> Core
    API --> Core

    Core --> Commands
    Core --> Services
    Core --> Workflows
    Core --> Plugins

    Services --> OS
```

The core engine owns every piece of business logic.

Interfaces never communicate directly with the operating system.

------------------------------------------------------------------------

## Repository Structure

``` text
coco/
│
├── docs/
│
├── coco-cli/
│   └── Command-line application
│
├── coco-core/
│   └── Core engine and business logic
│
├── coco-plugin-api/
│   └── Public interfaces for extensions
│
├── coco-gui/
│   └── Desktop application
│
├── coco-plugins/
│   └── Official plugins
│
└── examples/
```

------------------------------------------------------------------------

## Module Responsibilities

### coco-core

The heart of the platform.

Responsible for:

-   Business logic
-   Services
-   Workflow execution
-   Plugin loading
-   Project detection
-   Diagnostics
-   Process execution
-   Configuration
-   Shared models

The core must never depend on the CLI or GUI.

------------------------------------------------------------------------

### coco-cli

Provides the command-line experience.

Responsibilities:

-   Parse command-line arguments
-   Validate user input
-   Invoke the core engine
-   Render results

The CLI contains no business logic.

------------------------------------------------------------------------

### coco-gui

Provides the graphical desktop application.

Responsibilities:

-   Display information
-   Collect user input
-   Invoke the core engine
-   Render results

The GUI must never implement features independently of the core.

------------------------------------------------------------------------

### coco-plugin-api

Defines the contracts used by third-party plugins.

Plugins should never depend directly on implementation classes inside
the core.

------------------------------------------------------------------------

### coco-plugins

Contains officially maintained plugins that extend the platform.

Examples include:

-   Git
-   Docker
-   Java
-   Node.js
-   Python
-   Databases

------------------------------------------------------------------------

## Request Lifecycle

``` mermaid
sequenceDiagram
    participant User
    participant CLI
    participant Core
    participant Service
    participant OS

    User->>CLI: coco doctor
    CLI->>Core: execute("doctor")
    Core->>Service: run diagnostics
    Service->>OS: inspect system
    OS-->>Service: results
    Service-->>Core: diagnostic model
    Core-->>CLI: response model
    CLI-->>User: formatted output
```

------------------------------------------------------------------------

## Layered Architecture

``` text
Presentation
│
├── CLI
├── GUI
└── API

↓

Core Engine

↓

Commands

↓

Actions

↓

Services

↓

Operating System
```

Each layer has a single responsibility.

No layer may bypass the one beneath it.

------------------------------------------------------------------------

## Dependency Rules

Allowed:

-   CLI → Core
-   GUI → Core
-   Plugins → Plugin API
-   Core → Plugin API

Forbidden:

-   Core → CLI
-   Core → GUI
-   GUI → CLI
-   CLI → GUI
-   Plugins → CLI
-   Plugins → GUI

Dependencies must always point inward toward the core.

------------------------------------------------------------------------

## Command Flow

Commands are intentionally lightweight.

A command should:

1.  Read input.
2.  Validate arguments.
3.  Call the appropriate service.
4.  Return the result.

Commands should never contain business logic.

------------------------------------------------------------------------

## Services

Services implement business behavior.

Examples:

-   DiagnosticsService
-   ProjectDetectionService
-   WorkflowService
-   ProcessService
-   ConfigurationService

Services may collaborate, but each should have a single responsibility.

------------------------------------------------------------------------

## Models

The core communicates using models instead of console output.

Example flow:

``` text
DiagnosticsService

↓

DiagnosticReport

↓

CLI Formatter

↓

Console
```

This allows multiple interfaces to reuse identical functionality.

------------------------------------------------------------------------

## Plugins

Plugins extend the platform without modifying the core.

Each plugin registers commands, workflows, services, or integrations
through the public plugin API.

The core engine discovers plugins during startup.

------------------------------------------------------------------------

## Future Architecture

The architecture is intentionally designed to support future
capabilities without major refactoring.

Planned additions include:

-   AI providers
-   Cloud synchronization
-   Marketplace
-   Remote execution
-   Team collaboration
-   Public SDK

Each addition should integrate through the existing extension points
rather than changing the overall architecture.

------------------------------------------------------------------------

## Architectural Goals

Every architectural decision should support the following objectives:

-   Simplicity
-   Maintainability
-   Extensibility
-   Cross-platform compatibility
-   Testability
-   Reusability
-   Predictability

If a proposed change increases complexity without strengthening these
goals, it should be reconsidered.
