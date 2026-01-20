# C4 Hands-On Demo

A **multi-project workspace** demonstrating independent C4 architecture models using [LikeC4](https://likec4.dev). Each project showcases different architectural approaches with complete system models, deployment infrastructure, and operational views.

## 🎯 Available Projects

| Project | Focus | Key Concepts |
|---------|-------|--------------|
| **legacy** | Monolithic architecture | Load balancing, shared storage (NFS), synchronous processing |
| **refactored** | Microservices architecture | API gateway, async queues, object storage (MinIO) |
| **video-streaming** | CDN & streaming platform | Edge caching, HLS streaming, bandwidth optimization |
| **spec-showcase** | LikeC4 reference examples | C1/C2/C3 diagrams, patterns, best practices |

Each project is **self-contained** with its own specifications, models, and views. Start with any project—they share common LikeC4 patterns.

**To understand design decisions for each project:** See [Project Architectures](#-project-architectures) section below.

## 🚀 Quick Start

### Installation

1. **Clone this repository:**
   ```bash
   git clone https://github.com/a-scolan/c4_hands-on-demo.git
   cd c4_hands-on-demo
   ```

2. **Install Node.js dependencies** (if needed):
   ```bash
   npm install  # or use yarn/pnpm
   ```

3. **Install LikeC4 CLI** (global):
   ```bash
   npm install -g likec4
   ```

### Run the Development Server

Start the LikeC4 development server to preview all projects:

```bash
npx likec4 start
```

This will:
- Detect all 4 projects
- Load shared specifications from `projects/shared/`
- Start a local server at `http://localhost:3000`
- Watch files and hot-reload on changes

### View Project Architecture

Once the server is running:
- **Browse all project views** in the web interface
- **Compare different projects** side-by-side
- **Navigate between C1/C2/C3** diagrams
- **Explore sequences** showing use case flows
- **Review deployment** infrastructure topology

### Alternative: Work on Individual Projects

You can focus on a single project:

```bash
cd projects/legacy       # or: projects/refactored, projects/video-streaming
npx likec4 start         # Starts server for only this project
```

## 📁 Workspace Structure

```
c4_hands-on-demo/
├── projects/
│   ├── shared/              # Shared specifications for all projects
│   │   ├── spec-global.c4   # Global element kinds, colors, tags
│   │   ├── spec-context.c4  # Actor and System definitions
│   │   ├── spec-containers.c4 # Container kinds and colors
│   │   ├── spec-components.c4 # Component kinds
│   │   ├── spec-code.c4     # Code-level element kinds
│   │   ├── spec-deployment.c4 # Deployment infrastructure kinds
│   │   ├── SPEC_CHEATSHEET.md # Quick reference for all kinds
│   │   └── images/          # SVG icons for elements
│   │
│   ├── legacy/              # Vault: Monolithic architecture
│   │   ├── likec4.config.json
│   │   ├── system-model.c4  # Logical architecture
│   │   ├── system-views.c4  # C1/C2/C3 diagrams
│   │   ├── deployment.c4    # Physical infrastructure (VMs, networks)
│   │   ├── deployment-views.c4 # Infrastructure topology views
│   │   ├── operations.c4    # Monitoring & backup infrastructure
│   │   ├── operations-views.c4 # Operational views
│   │   ├── system-sequences.c4 # Use case flows
│   │   └── ARCHITECTURE_DECISIONS.md # Design rationale
│   │
│   ├── refactored/          # Vault: Microservices architecture
│   │   ├── likec4.config.json
│   │   ├── system-model.c4  # Logical architecture
│   │   ├── system-views.c4  # C1/C2/C3 diagrams
│   │   ├── deployment.c4    # Physical infrastructure
│   │   ├── deployment-views.c4 # Infrastructure topology views
│   │   ├── operations.c4    # Monitoring & backup infrastructure
│   │   ├── operations-views.c4 # Operational views
│   │   ├── system-sequences.c4 # Use case flows
│   │   └── ARCHITECTURE_DECISIONS.md # Design rationale
│   │
│   ├── video-streaming/     # Video Streaming Platform
│   │   ├── likec4.config.json
│   │   ├── system-model.c4
│   │   ├── system-views.c4
│   │   ├── deployment.c4
│   │   ├── deployment-views.c4
│   │   ├── ADR/             # Architecture decision records
│   │   └── system-sequences.c4
│   │
│   └── spec-showcase/       # LikeC4 Reference Examples
│       ├── likec4.config.json
│       ├── components.c4
│       ├── containers.c4
│       ├── actors-systems.c4
│       ├── context-views.c4
│       ├── container-views.c4
│       └── component-views.c4
│
├── .github/
│   ├── copilot-instructions.md # AI assistant instructions
│   └── skills/              # 14 reusable Copilot skills
│
├── ARCHITECTURE_DECISIONS.md # Workspace-level architectural decisions
├── README.md               # (You are here)
└── c4_hands-on-demo.code-workspace
```

### Multi-Project Principles

- **Independent Scopes:** Each project has its own namespace—no naming conflicts
- **Shared Specifications:** All projects inherit element kinds, tags, and styling from `projects/shared/`
- **Separate likec4.config.json:** Each project has unique configuration (name, includes, paths)
- **No Cross-Project References:** Models don't depend on elements from other projects
- **Parallel Development:** Work on multiple projects independently; they can be developed, versioned, and deployed separately

### File Organization Pattern

Every LikeC4 project follows this structure:

| File | Purpose | Contains |
|------|---------|----------|
| **spec.c4** files | Shared specifications | Element kinds, relationship types, colors, tags, icons |
| **system-model.c4** | Logical architecture | Systems, containers, components (deployment-agnostic) |
| **system-views.c4** | C1/C2/C3 visualizations | Context, Container, and Component level views |
| **deployment.c4** | Physical infrastructure | Environments, zones, VMs, deployed apps |
| **deployment-views.c4** | Infrastructure topology | Deployment views showing actual infrastructure |
| **operations.c4** | Monitoring & backup | Observability, logging, backup infrastructure |
| **operations-views.c4** | Operational views | Monitoring topology and operational infrastructure |
| **system-sequences.c4** | Dynamic flows | Use case diagrams and interaction sequences |

**Key Principle:** Keep business logic (model files) separate from physical infrastructure (deployment files). This enables the same system to be deployed in different ways without changing the logical model.

## 📚 LikeC4 Essentials

### Naming Conventions

Follow **PascalCase** for all element types:

| Type | Pattern | Examples |
|------|---------|----------|
| Elements | PascalCase | `SecureVault`, `ApiGateway`, `Customer` |
| Categorized | Category_Subtype | `Actor_User`, `System_External`, `Container_Database` |
| Deployment nodes | Category_Subtype | `Zone_AppTier`, `Node_Vm`, `Tier_DMZ` |
| FQN references | Dots | `vault.api`, `Prod.AppTier.ApiGwVm` |
| Tags | PascalCase | `#External`, `#Production`, `#Infrastructure` |
| Identifiers (FQN strings) | kebab-case | `"prod-api-gw-vm"`, `"prod-monolith-vm-01"` |

**Why this pattern?**
- Clear visual distinction between categories and subtypes
- Consistent with most programming language naming
- Self-documenting element hierarchies
- Element identifiers use kebab-case for consistency with common configuration patterns

### Relationship Types

Relationships in LikeC4 use typed kinds to specify communication patterns:

| Model Relationships | Deployment Relationships |
|-------------------|------------------------|
| `calls`, `uses` | `http`, `https` |
| `async` | `amqp`, `tcp`, `udp` |
| `reads`, `writes` | `nfs`, `smb`, `s3` |

**Key Rules:**
- Always specify relationship kind (e.g., `vault -[calls]-> api`)
- One-directional only—no return/response relationships (they're implicit)
- Use descriptive labels (e.g., `"Route upload requests"`, not just `"calls"`)
- Model relationships are deployment-agnostic; deployment relationships show actual protocols

### Element Structure

Every element should have:
1. **Name** (identifier)
2. **Description** (what it does)
3. **Technology** (tech stack)
4. **Optional metadata** (markdown tables for deployment infrastructure)

Example system element:
```likec4
Customer = Actor_User "Customer" {
  description """
  External user accessing the vault system.
  Interacts via web browser.
  """
  technology "Web browser (HTTPS)"
}
```

Example deployment element:
```likec4
ProdMonolithVm01 = Node_Vm "prod-monolith-vm-01" {
  technology "Ubuntu 20.04 LTS, Spring Boot 2.7"
  description """
  Primary application server for monolith.
  
  | Network | eth0 |
  | IP Address | 10.0.1.10 |
  | VLAN | 100 (App Tier) |
  | CPU | 8 cores |
  | RAM | 16 GB |
  | Storage | 100 GB SSD |
  """
}
```

## 🤖 GitHub Copilot Skills

This workspace includes **14 reusable Copilot skills** that teach the AI assistant domain-specific procedures for LikeC4 work:

| Skill | Purpose |
|-------|---------|
| [understand-project-structure](./.github/skills/understand-project-structure/SKILL.md) | Load project context before making changes |
| [create-element](./.github/skills/create-element/SKILL.md) | Define elements with proper naming, hierarchy, metadata |
| [create-relationship](./.github/skills/create-relationship/SKILL.md) | Add typed relationships with FQN, kinds, labels |
| [design-view](./.github/skills/design-view/SKILL.md) | Create C1/C2/C3 views with filtering and layout |
| [create-sequence-view](./.github/skills/create-sequence-view/SKILL.md) | Show temporal flows and use case interactions |
| [model-deployment](./.github/skills/model-deployment/SKILL.md) | Define deployment infrastructure (zones, VMs, apps) |
| [write-rich-descriptions](./.github/skills/write-rich-descriptions/SKILL.md) | Create self-documenting metadata with tables |
| [customize-view](./.github/skills/customize-view/SKILL.md) | Advanced styling, colors, layout hints, navigation |
| [c4-modeling-process](./.github/skills/c4-modeling-process/SKILL.md) | C4 methodology: design hierarchy top-to-bottom |
| [test-model](./.github/skills/test-model/SKILL.md) | Validate model correctness with MCP tools |
| [troubleshoot-errors](./.github/skills/troubleshoot-errors/SKILL.md) | Diagnose common LikeC4 errors and fix issues |
| [lookup-element-kinds](./.github/skills/lookup-element-kinds/SKILL.md) | Quick reference for element kinds and relationship types |
| [document-decision](./.github/skills/document-decision/SKILL.md) | Create Architecture Decision Records (ADRs) |
| [implement-pattern](./.github/skills/implement-pattern/SKILL.md) | Apply tested architecture patterns |

**How to use:** Ask Copilot to help with any architecture task. Copilot will automatically apply relevant skills with full context.

Browse all skills: [`.github/skills/`](.github/skills/)

## 📐 Best Practices

### Relationship Typing Rules

**✅ ALWAYS use typed kinds** in static relationships:
- Model: `vault -[calls]-> api`, `worker -[async]-> queue`, `service -[reads]-> cache`
- Deployment: `app1 -[https]-> app2`, `vm -[nfs]-> storage`

**✅ Use descriptive labels:**
- Good: `"Route upload requests"`, `"Scrape metrics"`, `"Save encrypted file"`
- Avoid: `"calls"`, `"uses"`, `"communicates"`

**✅ One-directional only:**
- Model: A → B (implies response from B → A)
- Never: Model both A → B and B → A

**⚠️ Dynamic views (sequences) are the exception:**
- Use untyped arrows for temporal flow clarity
- Focus on use case step-by-step interactions

### Deployment Relationships

- **VM-to-VM links:** Infrastructure-level connections between hosts
- **App-to-App links:** Deployed instances communicating across VMs
- **Port documentation:** Use `any -> targetPort` for most flows; `sourcePort -> targetPort` only when source port is architecturally significant

### View Design Patterns

- **Scoped includes:** `include Prod.AppTier.**` (descendants) or `include -> Prod.AppTier.*` (incoming)
- **Rank guidance:** Use `rank source` (users), `rank same` (horizontal), `rank sink` (databases)
- **Avoid wildcards:** Use scoped filters instead of `include ** -> **`

### Tagging Strategy

Use semantic tags for filtering and styling:

- **Purpose:** `#External`, `#Internal`, `#Legacy`, `#Saas`
- **Environment:** `#Production`, `#Staging`, `#Development`
- **Infrastructure:** `#Networking`, `#Security`, `#Monitoring`, `#Storage`
- **Service:** `#API`, `#Cache`, `#Queue`, `#Database`

### Architecture Organization

**Separation of Concerns:**
- **Models** = Deployment-agnostic business logic
- **Deployment** = Physical infrastructure
- **Views** = C1/C2/C3 visualizations
- **Sequences** = Dynamic use case flows

**Key Principle:** Same logical model can be deployed to different physical infrastructures.

## 🏗️ Project Architectures

Each project documents its architecture decisions in an `ARCHITECTURE_DECISIONS.md` file:

### Legacy (Monolithic Vault)
**File:** [projects/legacy/ARCHITECTURE_DECISIONS.md](projects/legacy/ARCHITECTURE_DECISIONS.md)

Key decisions:
- Monolithic Spring Boot application
- HAProxy load balancer
- Shared NFS storage
- PostgreSQL database
- Synchronous request/response pipeline

### Refactored (Microservices Vault)
**File:** [projects/refactored/ARCHITECTURE_DECISIONS.md](projects/refactored/ARCHITECTURE_DECISIONS.md)

Key decisions:
- Microservices architecture
- Kong API gateway
- Async job queue (RabbitMQ)
- MinIO object storage
- Asynchronous processing pipeline

### Video Streaming Platform
**File:** [projects/video-streaming/ADR/](projects/video-streaming/ADR/)

Key decisions:
- CDN edge caching
- HLS streaming protocol
- Bandwidth optimization
- Multi-region deployment

### Spec Showcase
Educational reference examples for LikeC4 patterns and best practices.

## 📚 LikeC4 Documentation

- **Official Docs:** [likec4.dev](https://likec4.dev)
- **DSL Reference:** [LikeC4 DSL Guide](https://likec4.dev/dsl/)
- **CLI Reference:** [likec4 start](https://likec4.dev/tooling/cli#start)
- **Live Editor:** [likec4.dev/playground](https://likec4.dev/playground)

## 🔧 VS Code Setup

### Recommended Extensions

- **LikeC4 Extension** — Syntax highlighting, IntelliSense, live preview
  - Search for "LikeC4" in Extensions marketplace
  - Install by [LikeC4](https://marketplace.visualstudio.com/items?itemName=likec4.likec4-vscode)

- **GitHub Copilot** — AI-assisted coding
  - Search for "GitHub Copilot" in Extensions marketplace
  - Sign in with your GitHub account

### Useful MCP Servers

For enhanced AI workflows:

- **LikeC4 MCP** — Query and navigate models programmatically
  - [LikeC4 MCP Docs](https://likec4.dev/tooling/mcp/)
  - [GitHub](https://github.com/likec4/likec4)

- **Context7 MCP** — Documentation search for LikeC4 and other libraries
  - [context7.com](https://context7.com/)

- **GitHub MCP** — Manage issues, pull requests, and repositories
  - [GitHub MCP Server](https://github.com/github/github-mcp-server)

## 🔄 Template Sync

This repository uses the [c4-template](https://github.com/a-scolan/c4-template) as its foundation. To stay current with improvements to skills, specifications, and examples:

```bash
# 1. Add template remote (first time only)
git remote add c4-template https://github.com/a-scolan/c4-template.git

# 2. Fetch latest
git fetch c4-template main

# 3. Update the three essentials
git checkout c4-template/main -- .github/copilot-instructions.md .github/skills/
git checkout c4-template/main -- projects/shared/
git checkout c4-template/main -- projects/spec-showcase/

# 4. Review and commit
git add .github/ projects/shared/ projects/spec-showcase/
git commit -m "sync: update template files (copilot instructions, skills, specs, examples)"
git push
```

**What to sync together:**
- `.github/` — Copilot instructions and skill files
- `projects/shared/` — Shared specifications and icons
- `projects/spec-showcase/` — Example C4 diagrams

## 📖 Contributing

When adding new projects or features:
1. Follow the [c4-modeling-process](./.github/skills/c4-modeling-process/SKILL.md)
2. Use naming conventions consistently across all files
3. Document architecture decisions in `ARCHITECTURE_DECISIONS.md`
4. Create views that support stakeholder communication
5. Test with `npx likec4 start` before committing

## 📄 License

This project is provided as an educational example. Refer to LICENSE file for details.

---

**Questions?** See the skills in `.github/skills/` or visit [likec4.dev](https://likec4.dev)
