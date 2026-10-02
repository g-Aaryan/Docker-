# Docker — Complete Learning Notes

Docker is a platform used to build, package, ship, and run applications in a consistent and isolated environment.

> **Core idea: Build once, run consistently anywhere.**

---

# 1. Why Do We Need Docker?

An application does not run in isolation. It depends on several components of its environment, such as:

- Programming language/runtime version
- Libraries and dependencies
- Database versions
- Cache versions
- Operating-system libraries
- Environment variables
- Configuration files
- System-level dependencies

## The "It Works on My Machine" Problem

Suppose a team has three developers:

```text
Developer A
├── Node.js 20
├── MySQL 8
├── Redis 7
└── Windows

Developer B
├── Node.js 18
├── MySQL 5.7
├── Redis 6
└── Linux

Developer C
├── Node.js 20
├── MySQL 8
├── Redis 6
└── macOS
```

The same application may work perfectly on Developer A's machine but fail on Developer B's machine.

This happens because the environments are different.

The same problem can occur when moving an application between:

```text
Developer Machine
       ↓
Testing Environment
       ↓
Staging Environment
       ↓
Production Environment
```

Each environment may have different versions, configurations, libraries, or dependencies.

This leads to the famous problem:

> **"It works on my machine!"**

---

# 2. How Does Docker Solve This Problem?

Docker allows us to package an application together with its required user-space dependencies into a **Docker image**.

Instead of manually configuring every machine, we can define the application's environment using a Dockerfile and build an image from it.

Conceptually:

```text
                 Docker Image
                      │
        ┌─────────────┴─────────────┐
        │                           │
 Application Code            Dependencies
        │                           │
        ├── Node.js                 │
        ├── Libraries               │
        └── Configuration           │
                      │
                      ▼
                  Container
                      │
                      ▼
               Running Application
```

The same image can then be used across different environments:

```text
                  Docker Image
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
      Developer      Testing    Production
       Machine       Server       Server
```

This provides greater consistency between development, testing, and production environments.

---

# 3. Example — Node.js Application

Suppose we have a Node.js application that requires Node.js 20.

We can define the environment using a Dockerfile:

```dockerfile
FROM node:20

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

CMD ["npm", "start"]
```

Here:

- `FROM node:20` provides the Node.js 20 environment.
- `WORKDIR /app` sets the working directory.
- `COPY package*.json ./` copies package files.
- `RUN npm install` installs dependencies.
- `COPY . .` copies the application source code.
- `CMD ["npm", "start"]` starts the application.

The basic workflow becomes:

```text
Dockerfile
    ↓
Docker Image
    ↓
Docker Container
    ↓
Running Application
```

---

# 4. Where is Docker Used in the IT Industry?

Docker is used throughout the software development and deployment lifecycle.

## 4.1 Development

Docker helps developers create consistent development environments.

For example, instead of manually installing Node.js, MySQL, and Redis:

```text
Node.js
MySQL
Redis
```

we can run these services using containers:

```text
Node.js Application
       │
       ├── Node.js Container
       ├── MySQL Container
       └── Redis Container
```

This reduces environment-related inconsistencies between developers.

---

## 4.2 Testing

Docker can be used to create isolated environments for automated testing.

A CI pipeline can follow a flow such as:

```text
Source Code
    ↓
Build Docker Image
    ↓
Start Container
    ↓
Run Tests
    ↓
Destroy Container
```

This allows tests to run in a predictable environment.

---

## 4.3 Deployment

Docker images can be built and deployed to servers.

A common workflow is:

```text
Source Code
    ↓
Dockerfile
    ↓
Docker Build
    ↓
Docker Image
    ↓
Container Registry
    ↓
Production Server
    ↓
Docker Container
```

The same application image can be used across different environments.

---

## 4.4 Microservices

Docker is particularly useful in microservice architectures.

For example, a ride-booking application might contain:

```text
                    API Gateway
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
   Profile Service   Trip Service   Payment Service
          │              │              │
      Container       Container       Container
```

Each service can have its own runtime and dependencies.

---

# 5. Virtualization

Before understanding containerization, it is important to understand traditional virtualization.

Virtualization allows a physical machine to run multiple **Virtual Machines (VMs)**.

Each VM behaves like an individual computer and generally has its own guest operating system.

```text
                 Physical Machine
                        │
                    Hypervisor
                        │
        ┌───────────────┼───────────────┐
        ▼               ▼               ▼
       VM 1            VM 2            VM 3
        │               │               │
    Guest OS         Guest OS         Guest OS
        │               │               │
      App A           App B           App C
```

For example:

```text
VM 1 → Ubuntu + Application A
VM 2 → Ubuntu + Application B
VM 3 → CentOS + Application C
```

Each VM requires its own guest operating system.

Therefore, running multiple VMs generally requires more resources.

---

# 6. Containerization

Containerization takes a different approach.

Containers provide isolated environments for applications while sharing the **host operating system's kernel**.

```text
                  Host Machine
                       │
                   Host Kernel
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
   Container 1    Container 2    Container 3
        │              │              │
      App A           App B           App C
```

For example:

```text
Container 1 → Node.js Application
Container 2 → Redis
Container 3 → MySQL
```

Containers are isolated from each other while sharing the host kernel.

---

# 7. Why Are Containers Lightweight?

A virtual machine generally looks like:

```text
Virtual Machine
├── Application
├── Dependencies
├── Guest Operating System
└── Virtualized Hardware
```

A container is conceptually closer to:

```text
Container
├── Application
├── Dependencies
└── Isolated processes/filesystem/networking
          │
          ▼
      Host Kernel
```

Because containers do not each require a separate guest kernel, they generally have lower overhead than running multiple full virtual machines.

This can result in:

- Faster startup
- Lower resource overhead
- Higher application density
- Easier scaling

The exact resource usage depends on the workload and configuration.

---

# 8. Virtualization vs Containerization

| Feature | Virtual Machine | Container |
|---|---|---|
| Main abstraction | Hardware/system | OS-level environment |
| Guest OS | Yes | No separate guest kernel |
| Kernel | Each VM has its own guest kernel | Containers share host kernel |
| Resource overhead | Generally higher | Generally lower |
| Startup | Generally slower | Generally faster |
| Isolation | VM-level isolation | Process/resource-level isolation |
| Size | Generally larger | Generally smaller |
| Typical use | Full OS isolation | Application packaging and deployment |

---

# 9. Important Technical Clarification

A common simplified statement is:

> **"VMs virtualize hardware, while containers virtualize the operating system."**

This is useful for building intuition, but technically containers do not virtualize an entire operating system.

Containers primarily provide **isolated processes and resources using operating-system features of the host kernel**.

Also, containers still use the host kernel.

---

# 10. Core Docker Mental Model

The most important concept to remember at this stage is:

```text
                 Application
                      │
                      ▼
                 Dockerfile
                      │
                      ▼
                Docker Build
                      │
                      ▼
                 Docker Image
                      │
                      ▼
                Docker Container
                      │
                      ▼
              Running Application
```

### Dockerfile

A Dockerfile contains instructions that describe how to build an image.

### Docker Image

An image is a packaged, immutable template containing the application and everything required to run it.

### Docker Container

A container is a running instance of an image.

---

# 11. Key Takeaways

### Why Docker?

Docker helps solve the **environment consistency problem** by packaging an application and its required user-space dependencies into a reproducible image.

### Virtualization

Virtual machines generally virtualize a complete machine and run their own guest operating systems.

### Containerization

Containers provide isolated application environments while sharing the host kernel.

### Docker

Docker provides tools and a platform for building images, running containers, distributing images, and managing containerized applications.

### Core Flow

```text
Dockerfile
    ↓
Docker Image
    ↓
Docker Container
    ↓
Running Application
```

---

# 12. Topics To Be Added

As I continue learning Docker, this document will be updated with practical examples, commands, projects, and implementation references.

- [ ] Docker Architecture
- [ ] Docker Engine
- [ ] Docker CLI
- [ ] Docker Daemon
- [ ] Docker Desktop
- [ ] Docker Images
- [ ] Docker Containers
- [ ] Dockerfile in Depth
- [ ] Docker Hub
- [ ] Container Registry
- [ ] Docker Volumes
- [ ] Docker Networks
- [ ] Docker Compose
- [ ] Multi-Stage Docker Builds
- [ ] Distroless Images
- [ ] Docker Scout
- [ ] Docker Security
- [ ] Docker in Real Projects
- [ ] Docker Commands Cheat Sheet
