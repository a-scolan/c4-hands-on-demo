# C4 Hands-On Demo - Multi-Project Architecture

🔗 **[View Live Diagrams on GitHub Pages](https://a-scolan.github.io/c4-hands-on-demo/)**  
📋 **[Architecture Decisions (ADR)](./ARCHITECTURE_DECISIONS.md)** - Why Kong? Why MinIO? Why remove S3?

This workspace contains two **independent** LikeC4 projects demonstrating architecture evolution:

## 🎯 Project Structure

### 🏛️ Legacy Architecture (`projects/legacy/`)
Initial monolithic architecture of the Vault system
- **Config file**: `likec4.config.json`
- **Specifications**: `spec.c4`
- **Model**: `model.c4`
- **Views**: `views.c4`

### 🚀 Refactored Architecture (`projects/refactored/`)
Refactored microservices architecture with async processing
- **Config file**: `likec4.config.json`
- **Specifications**: `spec.c4` (extended with queues, microservices, etc.)
- **Model**: `model.c4`
- **Views**: `views.c4` (multiple views: landscape, processing, security)

### 📊 Key Architecture Views

Compare the evolution from monolith to microservices:

| View | Project | Description | What You'll See |
|------|---------|-------------|-----------------|
| **[C1 Context](https://a-scolan.github.io/c4-hands-on-demo/#/project/legacy-vault-system/view/c1_context/)** | Legacy | System boundaries | Monolithic vault, NFS storage, VirusTotal |
| **[C2 Container](https://a-scolan.github.io/c4-hands-on-demo/#/project/legacy-vault-system/view/c2_container/)** | Legacy | Monolith internals | Spring Boot monolith with PostgreSQL + NFS |
| **[C3 Monolith](https://a-scolan.github.io/c4-hands-on-demo/#/project/legacy-vault-system/view/c3_monolith_components/)** | Legacy | Component details | Synchronous layers: Upload → Validate → Scan → Encrypt |
| **[Use Cases: Upload (Legacy)](https://a-scolan.github.io/c4-hands-on-demo/#/project/legacy-vault-system/view/usecases_upload_flow/)** | Legacy | Blocking upload | Browser loads SPA → API calls → Synchronous processing (10-30s) |
| **[Use Cases: Retrieval (Legacy)](https://a-scolan.github.io/c4-hands-on-demo/#/project/legacy-vault-system/view/usecases_retrieval_flow/)** | Legacy | Document download | SPA → API → Direct NFS reads |
| **[Deployment (Legacy)](https://a-scolan.github.io/c4-hands-on-demo/#/project/legacy-vault-system/view/overview/)** | Legacy | Infrastructure | 2 monolith VMs, PostgreSQL, NFS, HAProxy |
| | | | |
| **[C1 Context](https://a-scolan.github.io/c4-hands-on-demo/#/project/refactored-vault-system/view/c1_context/)** | Refactored | System boundaries | Vault system, MinIO storage, VirusTotal API |
| **[C2 Container](https://a-scolan.github.io/c4-hands-on-demo/#/project/refactored-vault-system/view/c2_container/)** | Refactored | Microservices | Kong, Upload, Retrieval (merged), Worker, Queue, MongoDB |
| **[C3 Upload Service](https://a-scolan.github.io/c4-hands-on-demo/#/project/refactored-vault-system/view/c3_upload_service/)** | Refactored | Upload internals | Fail-fast validation before queuing |
| **[C3 Retrieval Service](https://a-scolan.github.io/c4-hands-on-demo/#/project/refactored-vault-system/view/c3_retrieval_service/)** | Refactored | Retrieval internals | Unified metadata + files with Redis cache |
| **[C3 Worker](https://a-scolan.github.io/c4-hands-on-demo/#/project/refactored-vault-system/view/c3_processing_worker/)** | Refactored | Worker internals | Async orchestrator: Scan → Encrypt → Store |
| **[Use Cases: Upload](https://a-scolan.github.io/c4-hands-on-demo/#/project/refactored-vault-system/view/usecases_upload_flow/)** | Refactored | Async upload | Browser loads SPA → API calls → Validate → Queue → Async processing (< 500ms) |
| **[Use Cases: Retrieval](https://a-scolan.github.io/c4-hands-on-demo/#/project/refactored-vault-system/view/usecases_retrieval_flow/)** | Refactored | Cached retrieval | SPA → API → Check cache → Fetch MinIO → Decrypt |
| **[Use Cases: HA Replication](https://a-scolan.github.io/c4-hands-on-demo/#/project/refactored-vault-system/view/usecases_backup_flow/)** | Refactored | Disaster recovery | MinIO 3-node distributed replication |
| **[Deployment Overview](https://a-scolan.github.io/c4-hands-on-demo/#/project/refactored-vault-system/view/overview/)** | Refactored | Infrastructure | 7 zones, 10 VMs, Nginx, Kong, MongoDB, MinIO, CI/CD |
| **[App Tier](https://a-scolan.github.io/c4-hands-on-demo/#/project/refactored-vault-system/view/app_tier/)** | Refactored | App deployment | Nginx (SPA), Upload, Retrieval services |
| **[CI/CD Pipeline](https://a-scolan.github.io/c4-hands-on-demo/#/project/refactored-vault-system/view/cicd/)** | Refactored | DevOps automation | GitLab, Harbor, schema migrations |

💡 **Tip:** Compare legacy vs. refactored: C1 Context → C2 Container → Use Cases Upload Flow to see the evolution

## � Installation & Setup

### Prerequisites

- **Node.js** (v18 or later)
- **VS Code** (recommended for best LikeC4 experience)
- **Git** (for cloning the repository)

### Quick Start

```bash
# Clone the repository
git clone https://github.com/your-username/c4_hands-on-demo.git
cd c4_hands-on-demo

# Install LikeC4 (if not already installed globally)
npm install -g likec4

# Start the development server
npx likec4 start
```

### Recommended VS Code Extensions

#### 1. **LikeC4 Extension** (Required)
The official LikeC4 VS Code extension provides syntax highlighting, IntelliSense, and live preview.

**Installation:**
- Open VS Code Extensions (Ctrl+Shift+X / Cmd+Shift+X)
- Search for "LikeC4"
- Install the extension by LikeC4

**Features:**
- Syntax highlighting for `.c4` files
- IntelliSense for element kinds, relationship types, and icons
- Live diagram preview
- Error detection and validation

📚 **Documentation:** [LikeC4 VS Code Extension](https://marketplace.visualstudio.com/items?itemName=likec4.likec4-vscode)

### Useful MCP Servers

For enhanced AI assistant workflows, these Model Context Protocol (MCP) servers provide powerful integrations:

#### **Context7** - Documentation Search
Access up-to-date documentation for LikeC4 and other libraries.

📚 **Documentation:** [Context7 MCP Server](https://context7.com/docs/resources/all-clients)  
🔗 **Website:** [context7.com](https://context7.com/)

#### **LikeC4 MCP Server** - Architecture Analysis
Query and navigate LikeC4 models programmatically.

📚 **Documentation:** [LikeC4 MCP](https://likec4.dev/tooling/mcp/)  
🔧 **GitHub:** [likec4/likec4](https://github.com/likec4/likec4)

#### **GitHub MCP Server** - Repository Management
Manage issues, pull requests, and repository operations.

📚 **Documentation:** [MCP GitHub Server](https://github.com/github/github-mcp-server)

#### **Language Server Protocol (LSP)**
VS Code's built-in LSP provides IntelliSense for LikeC4 files (307+ AWS icons, 2000+ Bootstrap icons).

📚 **Documentation:** [VS Code LSP Guide](https://marketplace.visualstudio.com/items?itemName=sehejjain.lsp-mcp-bridge)

### Verification

After setup, verify everything works:

```bash
# 1. Check LikeC4 installation
npx likec4 --version

# 2. Start development server
npx likec4 start

# 3. Open browser to http://localhost:3000
# You should see both legacy and refactored projects
```

## �🚀 How to Use

This workspace is configured as a **multi-project setup** where both legacy and refactored architectures coexist independently.

### Running the Development Server

Start the LikeC4 development server from the workspace root:

```bash
# From the workspace root directory
npx likec4 start
```

This will:
- Automatically detect both projects (`legacy-vault-system` and `refactored-vault-system`)
- Load their shared specifications from `projects/shared/`
- Start a local server to preview all views from both projects
- Watch for changes and hot-reload

### View Your Architecture

Once the server is running, you can:
- Browse all views from both projects in the web interface
- Compare legacy vs. refactored architectures side-by-side
- Navigate between C1 (Context), C2 (Container), and C3 (Component) views
- Explore use case diagrams showing upload and retrieval flows

### Alternative: Open Individual Projects

You can also work on projects separately:

```bash
# Navigate to a specific project
cd projects/legacy
npx likec4 start

# Or the refactored project
cd projects/refactored
npx likec4 start
```

## 📝 Important Notes

- Each project has a unique `name` in its `likec4.config.json`
- Elements (like `customer`, `vault`, etc.) are **scoped to their project**
- No naming conflicts between projects as they exist in separate namespaces
- Both projects share common specifications from `projects/shared/spec.c4`

## 🏗️ Project Architecture

```
c4_hands-on-demo/
├── projects/
│   ├── shared/           # Common specifications
│   │   └── spec.c4       # Shared elements, tags, relationships
│   ├── legacy/           # Monolithic architecture
│   │   ├── likec4.config.json
│   │   ├── spec.c4
│   │   ├── model.c4
│   │   └── views.c4
│   └── refactored/       # Microservices architecture
│       ├── likec4.config.json
│       ├── spec.c4
│       ├── model.c4
│       ├── views.c4
│       ├── sequences.c4  # Dynamic use case views
│       ├── deployment.c4 # Infrastructure deployment model
│       └── deployment-views.c4 # Deployment views
├── shared/               # Shared specifications
│   ├── spec.c4           # Base element kinds, relationships, tags
│   └── deployment-spec.c4 # Deployment-specific element kinds
└── c4-demo.code-workspace
```

## 📚 LikeC4 Code Structure & Organization
### File Organization

Each project has specialized files following [LikeC4 DSL structure](https://likec4.dev/dsl/):

| File | Purpose | Learn More |
|------|---------|---|
| **spec.c4** | Element & relationship kinds, colors, tags | [DSL Specs](https://likec4.dev/dsl/) |
| **model.c4** | Logical architecture - systems, containers, components | [Model](https://likec4.dev/dsl/model) |
| **views.c4** | C1/C2/C3 visualizations of the model | [Views](https://likec4.dev/dsl/views) |
| **deployment.c4** | Physical infrastructure - environments, zones, VMs, apps | [Deployment](https://likec4.dev/dsl/deployment) |
| **deployment-views.c4** | Infrastructure topology views | [Deployment Views](https://likec4.dev/dsl/deployment/views) |
| **sequences.c4** | Dynamic interaction flows (use cases) | [Sequences](https://likec4.dev/dsl/dynamic) |

**Key Principle:** Model = Business logic (deployment-agnostic) | Deployment = Physical infrastructure

### Naming Conventions

**PascalCase with Category_Subtype pattern:**

| Type | Pattern | Examples |
|------|---------|----------|
| Simple elements | PascalCase | `SecureVault`, `ApiGateway`, `Customer` |
| Categorized elements | Category_Subtype | `Actor_Person`, `System_Legacy`, `Container_Api` |
| Deployment nodes | Category_Subtype | `Zone_Internet`, `Node_Vm`, `Infra_Router` |
| Components | PascalCase | `FileValidator`, `CacheManager` |
| FQN references | dots | `vault.api`, `Prod.AppTier.FrontendVm` |
| Tags | PascalCase | `#External`, `#Legacy`, `#Production` |

**Naming Rules:**
- Use PascalCase for all parts of element names
- Use underscore `_` to explicitly separate category from subtype
- Category represents the element family (Actor, System, Container, Zone, Node, Infra)
- Subtype represents the specific variant (Person, Legacy, Api, Internet, Vm, Router)
- Simple elements without categorization use plain PascalCase

**Why this pattern?**
- Clear visual separation of category and subtype
- Explicit hierarchy in element definitions
- Easy to scan and understand element taxonomies
- Consistent with type naming conventions in many languages

[LikeC4 naming docs](https://likec4.dev/dsl/model#element-definitions)

### Relationship Types

| In model | In deployment | Examples |
|----------|---------------|----------|
| `uses`, `calls` | `http`, `https` | `vault.api -> vault.jobs` |
| `async`, `reads`, `writes` | `tcp`, `nfs`, `amqp` | `monitoring -[tcp]-> db_vm 'Scrape'` |

[LikeC4 relationships](https://likec4.dev/dsl/relationships)

## 📐 Best Practices & Standards

This repository demonstrates production-ready LikeC4 modeling patterns. All best practices are consistently applied across both legacy and refactored projects.

### 🎯 Relationship Typing Rules

**RULE 1: All static relationships MUST have a typed kind**

✅ **DO** - Always specify relationship kinds in model and deployment files:
```likec4
// Model files - business logic relationships
customer -[calls]-> vault.api 'Access API'
vault.worker -[async]-> vault.jobs 'Queue job'
vault.service -[reads]-> vault.db 'Fetch metadata'

// Deployment files - infrastructure protocols
AppTier -[https]-> DataTier 'API traffic'
Worker -[tcp]-> Database 'Persist metadata'
Upload -[amqp]-> Queue 'Publish jobs'
```

❌ **DON'T** - Use untyped arrows in model or deployment:
```likec4
// WRONG - Missing relationship kind
customer -> vault.api 'Access API'
AppTier -> DataTier 'API traffic'
```

✅ **EXCEPTION** - Dynamic views (use case flows) may use untyped arrows for flow clarity:
```likec4
// Acceptable in dynamic views - focus is on temporal flow
dynamic view usecases_upload {
  customer -> vault.frontend 'Upload file'
  vault.frontend -> vault.api 'POST /upload'
}
```

✅ **NO RETURN RELATIONSHIPS** - Return/response flows are implicit and should not be modeled:
```likec4
// ❌ BAD - Explicit return
browser -[calls]-> webServer 'Load SPA'
webServer -[calls]-> browser 'Returns HTML/JS'  // Don't add this

// ✅ GOOD - Return is implicit
browser -[calls]-> webServer 'Load SPA (HTML/JS/CSS)'
```

**Why?** Typed relationships enable:
- Clear communication protocols and patterns
- Better diagram filtering and queries
- Consistent styling and visual hierarchy
- Self-documenting architecture

### 📡 Deployment Relationships Best Practice

Deployment models show physical infrastructure and actual network flows. Follow these rules:

#### **Linking Strategy**
```likec4
// ✅ VM-to-VM for infrastructure level flows
UserBrowser -[https]-> Prod.Dmz.ProdApigwVm 'Load API'

// ✅ App-to-App when linking deployed instances (Node_App)
UserBrowser.browserApp -> Prod.Dmz.ProdApigwVm.apiApp 'API requests'

// ❌ DON'T link to non-existent properties
UserBrowser.browserApp.frontend -> VM.someProperty  // frontend doesn't exist!

// ❌ DON'T use functional descriptions
description "Web UI calls database"

// ✅ USE network-only descriptions
description "any -> 27017"  // protocol and port only
```

#### **Description Format: Port Semantics**

Port descriptions show network flow direction. Use:

- **`any -> targetPort`**: Connections from any source port to target
  - Browser to server: `any -> 443`
  - Monitoring scrapers: `any -> 8080`
  - Service to database: `any -> 27017`
  - Backup processes: `any -> 5432`

- **`sourcePort -> targetPort`**: When source port is known and significant
  - API Gateway to microservice: `443 -> 3001` (Kong listening on 443, calls service on 3001)
  - Load balancer to app: `443 -> 8080`

**Rule**: Use `any` for most connections (source port is implementation detail). Use actual source port only when it's architecturally significant.

#### **Valid Node_App References**
Only reference Node_App instances that actually exist in the deployment:
```likec4
// ✅ These exist and work
ProdApigwVm.apiApp         // defined in VM
ProdUploadVm.uploadApp     // defined in VM
UserBrowser.browserApp     // defined in UserBrowser

// ❌ These don't exist
ProdApigwVm.api            // api doesn't exist, only apiApp
UserBrowser.frontend       // frontend doesn't exist, only browserApp
```

### 🏗️ Architecture Organization

✅ **Separation of concerns** - Logical and physical architectures separate  
✅ **Proper hierarchies** - Systems → containers → components; VMs → apps  
✅ **Shared specs** - Common definitions prevent duplication  
✅ **File-per-purpose** - One concern per file (model, deployment, views, sequences)

**File Structure Pattern:**
```
project/
├── spec.c4              # Element kinds, colors, tags
├── model.c4             # Business logic (deployment-agnostic)
├── views.c4             # C1/C2/C3 visualization
├── deployment.c4        # Physical infrastructure
├── deployment-views.c4  # Infrastructure topology views
└── sequences.c4         # Dynamic use case flows
```

### 🎨 Naming & Documentation Standards

**RULE 2: Use PascalCase for all element names**

✅ **Consistent naming across all element types:**
```likec4
// Elements
element ApiGateway
element UploadService
element ProdApigwVm

// Tags
#Production
#Infrastructure
#Deployment

// Zones and environments
Prod.AppTier.ProdFrontendVm
Cicd.CicdZone.GitServerVm
```

**RULE 3: Always include descriptions and technologies**

✅ **Rich metadata for every element:**
```likec4
container uploadService 'Upload Service' {
  technology 'Node.js / Express'
  description 'Handles file uploads and validation'
  
  uploadHandler = component 'Upload Handler' {
    description 'Receives and validates files'
    technology 'Multer / Express'
  }
}
```

✅ **Detailed deployment infrastructure:**
```likec4
ProdDatabaseVm = vm 'ProdDatabaseVm' {
  technology "MongoDB 6.0"
  description """
    Document metadata and encryption keys
    
    | Property | Value |
    |:---------|:------|
    | eth0 | 10.3.0.17/24 |
    | CPU | 16 vCPU |
    | RAM | 64 GB |
    | Port | 27017 |
  """
}
```

**Why?** Rich metadata provides:
- Self-documenting diagrams
- Better stakeholder communication
- Operational clarity (ports, IPs, specs)
- Onboarding efficiency

### 🔗 Relationship Best Practices

**RULE 4: All relationships must have descriptive labels**

✅ **Clear, actionable labels:**
```likec4
vault.api -[calls]-> uploadService 'Route upload requests'
monitoring -[tcp]-> database 'Scrape metrics'
worker -[writes]-> storage 'Save encrypted file (primary)'
```

**RULE 5: Include technical details in deployment relationships**

✅ **Document ports, protocols, and routing:**
```likec4
ProdApigwVm -[https]-> ProdUploadVm 'Route uploads' {
  #Service #Routing
  description "443 -> 3001"
}

AppTier -[tcp]-> DataTier 'Service reads/writes' {
  description "any -> 27017 (MongoDB), 9000-9001 (MinIO)"
}
```

### 📊 View Design Patterns

**RULE 6: Use scoped wildcards for focused views**

✅ **Filter by context, not dump everything:**
```likec4
deployment view app_tier {
  include Prod.AppTier.**
  include -> Prod.AppTier.*    // Only incoming to this tier
  include Prod.AppTier.* ->     // Only outgoing from this tier
}
```

❌ **DON'T** - Include unfiltered wildcards:
```likec4
// WRONG - Shows too much
include **
include ** -> **
```

**RULE 7: Use rank to guide layout**

✅ **Control diagram flow with rank:**
```likec4
view c2_container {
  include customer, vault.*
  
  rank source {
    customer
  }
  
  rank sink {
    minio,
    scanner
  }
}
```

### 🏷️ Tagging Strategy

**RULE 8: Use semantic tags for filtering and styling**

✅ **Consistent tag taxonomy:**
```likec4
// Purpose tags
#External, #Internal, #Legacy, #Saas

// Environment tags
#Production, #Cicd, #Development

// Infrastructure tags
#Infrastructure, #Networking, #Security, #Monitoring

// Service tags
#Deployment, #Service, #Routing, #Persistence
```

### 🔄 Model-Deployment Linking

**RULE 9: Link logical to physical with instanceOf**

✅ **Connect model containers to deployment apps:**
```likec4
// In model.c4
container uploadService 'Upload Service' {
  technology 'Node.js / Express'
}

// In deployment.c4
ProdUploadVm = vm 'ProdUploadVm' {
  uploadApp = app 'Upload Service App' {
    instanceOf vault.uploadService  // Links to model
  }
}
```

**Why?** This enables:
- Same model deployed to multiple environments
- Clear traceability from business to infrastructure
- Deployment-agnostic architecture design

### 📝 Documentation Excellence

**RULE 10: Use multi-line descriptions for complex elements**

✅ **Structured tables in descriptions:**
```likec4
description """
  Primary storage cluster with high availability
  
  | Property | Value |
  |:---------|:------|
  | Storage | 500 TB |
  | Replication | 3-node cluster |
  | Availability | 99.9% |
"""
```

### Quick Tips

- **Models are deployment-agnostic** - Same app can run on different infrastructure
- **Apps reference containers via `instanceOf`** - Links logical to physical
- **Shared specs prevent duplication** - Both projects inherit common definitions
- **Scoped wildcards in views** - Use `-> zone.*` and `zone.*->` for related elements only
- **Technology matters** - Always specify tech stack for containers and components
- **Use icons** - `icon tech:spring-icon`, `icon tech:react` for visual recognition

### 📚 Learn More

For detailed syntax and examples, visit [LikeC4 Documentation](https://likec4.dev/dsl)

---

## 🔄 Keeping Template Files in Sync

This repository uses the [c4-template](https://github.com/a-scolan/c4-template) as its foundation. To stay current with improvements to Copilot skills, shared specifications, and example projects, periodically sync from the template.

### Three Directories to Always Update Together

| Directory | Contains | Why Update |
|-----------|----------|------------|
| `.github/` | Copilot instructions & 14 skill files | Better AI assistance workflows |
| `projects/shared/` | 6 spec files + 28+ SVG icons | Latest element kinds, tags, relationships |
| `projects/spec-showcase/` | Example C4 diagrams | Reference implementations |

**Important:** Always update all three directories together to maintain consistency with template standards.

### Recommended: Direct Checkout Method (Simpler)

```bash
# 1. Add template remote (first time only)
git remote add c4-template https://github.com/a-scolan/c4-template.git

# 2. Fetch latest from template
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

This method is simpler because it only pulls the specific files you need without full subtree history tracking.

### Alternative: Git Subtree Method (Full History)

If you want to preserve complete history of template changes:

```bash
# Fetch latest
git fetch c4-template main

# Pull updates for each subtree with squashed history
git subtree pull --prefix=.github c4-template main --squash
git subtree pull --prefix=projects/shared c4-template main --squash
git subtree pull --prefix=projects/spec-showcase c4-template main --squash

# Review and push
git status
git push
```

**Note:** The `--squash` flag consolidates all c4-template changes into one commit per sync. Updates are manual—they do not happen automatically.
