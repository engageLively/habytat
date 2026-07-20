# Habytat

**Habytat is a personal computing environment for the cloud.**

It brings the autonomy, continuity, and convenience of a personal workstation into cloud infrastructure without reducing the user to a transient session or forcing every tool into one monolithic image.

Habytat combines persistent storage, interchangeable applications, specialized computational runtimes, automation, and AI collaborators into one coherent environment.

## Why Habytat

Traditional cloud workspaces often make an uncomfortable trade:

* Personal workstations provide continuity, privacy, familiar tools, and user control.
* Cloud platforms provide elasticity, shared infrastructure, accessibility, and operational scale.

Habytat is designed to provide both.

The name is inspired by Habitat 67, which combined the privacy and individuality of a home with the density and shared infrastructure of an urban building.

Habytat applies the same idea to computing:

> A personal workstation, reconstructed as a modular cloud environment.

## Core Idea

A user workspace should not be one enormous container containing every editor, runtime, package, service, and AI tool.

Instead, Habytat composes a workspace from independent systems:

```text
Habytat
├── workspace shell
├── persistent user storage
├── computational runtimes
├── editors and applications
├── automation
└── AI collaborators
```

Each component can evolve independently while remaining part of one persistent user environment.

## Architecture

```text
                    ┌──────────────────────────┐
                    │ Identity and tenancy     │
                    │ JupyterHub               │
                    └────────────┬─────────────┘
                                 │
                    ┌────────────▼─────────────┐
                    │ Habytat workspace shell  │
                    │ files, launchers, state  │
                    └────────┬─────────┬───────┘
                             │         │
                    ┌────────▼───┐   ┌─▼──────────────┐
                    │ Runtime    │   │ Application    │
                    │ fabric     │   │ fabric         │
                    │ JEG        │   │ Goblin King    │
                    └─────┬──────┘   └──────┬─────────┘
                          │                 │
                    kernel pods       service pods
                          └──────────┬──────┘
                                     │
                             persistent workspace
```

### Identity and tenancy

JupyterHub provides:

* authentication
* user identity
* multi-user tenancy
* root workspace lifecycle
* routing to each user environment

Habytat does not replace JupyterHub.

### Workspace shell

The root workspace is deliberately thin.

It provides:

* the primary user interface
* file access
* application launchers
* runtime selection
* workspace state
* notifications
* automation entry points
* access to persistent storage

JupyterLab may initially serve as one shell or application surface, but Habytat is not defined by JupyterLab.

The shell and the editor are separate concepts.

### Runtime fabric

Jupyter Enterprise Gateway manages computational runtimes.

Instead of exposing only language names, Habytat presents capability-oriented environments such as:

* Data Science
* SciPy
* TensorFlow
* PyTorch
* Geospatial
* R Analytics
* Julia Scientific

Each environment maps to a curated kernel image with its own:

* dependencies
* resource profile
* CPU and memory limits
* optional GPU requirements
* security context
* persistent workspace mount

Users choose the environment that matches the work, not merely the language used to implement it.

### Application fabric

Goblin King manages external applications and long-running services.

Examples include:

* VS Code
* RStudio
* Galyleo
* Markdown editors
* CSV editors
* database tools
* dashboards
* model servers
* Claude Code
* Codex
* user-defined services
* persistent agents

Goblin King owns:

* workload creation
* readiness
* lifecycle
* routing
* service ownership
* restart and cleanup
* project and user policy

Habytat should reuse Goblin King’s existing JupyterHub identity integration and Goblin Directory interfaces wherever possible.

### Persistent storage

Every component in a user’s Habytat should see the same persistent workspace at the same canonical path.

For example:

```text
/home/jovyan/work
```

That workspace may be mounted by:

* the root shell
* kernel pods
* VS Code
* RStudio
* Galyleo
* AI agents
* other user services

On GKE, the expected implementation is an RWX-capable Filestore-backed PVC.

Persistent storage belongs to the user’s Habytat, not to any individual pod.

Pods may stop, restart, or be replaced without destroying the workspace.

## AI Collaborators

Habytat supports persistent AI collaborators as a first-class part of the environment.

The public architecture is generic and may support:

* Claude Code
* Codex
* API-backed agents
* local models
* organization-hosted models
* named persistent collaborators
* user-provided subscriptions and credentials

A collaborator may have:

* persistent identity
* private and shared memory
* provenance
* workspace access
* tool permissions
* model portability
* reflection and experimentation
* continuity across sessions and runtimes

AI collaborators are not treated as interchangeable command-line utilities unless that is how the user chooses to configure them.

## Automation

A general cloud workstation needs more than launchers and mouse clicks.

Habytat should provide an automation layer capable of expressing workflows such as:

```text
open this project
start the Data Science runtime
launch Galyleo
mount a dataset
run an analysis
open the result in the CSV editor
ask an AI collaborator to review the work
```

Automation should be:

* scriptable
* reproducible
* inspectable
* composable
* callable by users and agents
* independent of any single editor

The scripting and workflow design is still open.

## Design Principles

### Invent as little as possible

Prefer:

1. configuration
2. existing extension points
3. small upstream contributions
4. custom integration code
5. forks only as a last resort

Habytat should compose mature systems rather than replace them.

### The workspace is logical, not physical

A Habytat is not one container or one pod.

It is a persistent user environment composed of multiple cooperating workloads.

```text
Habytat
├── one root workspace
├── zero or more active kernels
├── zero or more active applications
├── persistent storage
└── persistent identity
```

### Capability over implementation

Users should see:

```text
Data Science
TensorFlow
Geospatial
VS Code
Galyleo
RStudio
```

They should not need to reason about:

```text
container images
kernel process proxies
ZeroMQ channels
pod templates
service routes
PVC names
```

Those are implementation details.

### Identity must be trusted

User identity must flow from the authentication layer into:

* kernel ownership
* service ownership
* workspace selection
* route authorization
* resource policy

A browser or notebook client must never be allowed to select another user’s PVC or impersonate another workspace owner.

### Components should remain replaceable

Habytat should not become permanently coupled to one editor, one runtime manager, one model provider, or one user interface.

JupyterLab is an application.

VS Code is an application.

Jupyter Enterprise Gateway is a runtime fabric.

Goblin King is an application fabric.

None of them is the whole Habytat.

## Initial Implementation

The first implementation should use:

* Zero to JupyterHub
* a thin root workspace image
* Jupyter Enterprise Gateway
* Goblin King
* Goblin Directory
* GKE Autopilot
* Filestore-backed RWX storage
* curated kernel images
* declarative service definitions
* strong two-user isolation tests

## Initial Proof of Concept

The first milestone should demonstrate:

1. Two authenticated users receive separate Habytats.
2. Each user has a persistent workspace.
3. Each user can launch a Data Science kernel through JEG.
4. Each kernel sees the correct user workspace.
5. Each user can launch a VS Code service through Goblin King.
6. Each VS Code instance sees the correct user workspace.
7. One user cannot access another user’s service route.
8. One user cannot mount another user’s PVC.
9. The root image remains small.
10. JupyterHub remains unmodified.

## Repository Structure

A likely initial structure is:

```text
habytat/
├── README.md
├── images/
│   ├── root/
│   │   └── Dockerfile
│   └── kernels/
│       ├── datascience/
│       ├── scipy/
│       ├── tensorflow/
│       ├── geospatial/
│       ├── r-analytics/
│       └── julia-scientific/
├── deploy/
│   ├── z2jh-values.yaml
│   ├── enterprise-gateway-values.yaml
│   ├── goblin-king-values.yaml
│   ├── storage.yaml
│   └── network-policies.yaml
├── kernelspecs/
├── services/
└── tests/
```

## Open Questions

The first questions to resolve are:

* How should JEG derive the authenticated user’s PVC?
* Can that mapping be expressed entirely through existing templates?
* How should Goblin King attach the existing user workspace to service workloads?
* Does Goblin King already enforce per-user service ownership on every proxy request?
* Can Goblin Directory metadata describe launcher icons, categories, open modes, and singleton behavior?
* What should Habytat use as its long-term shell?
* What should the automation language look like?
* Which services should be multi-tenant, and which should be per-user workloads?
* What lifecycle and idle-culling policies provide the best balance of responsiveness and cost?

## Status

Habytat is currently an architectural prototype.

The immediate goal is not to build a large new platform.

The immediate goal is to prove that existing systems can be composed into something larger:

> A persistent, extensible personal workstation in the cloud.
