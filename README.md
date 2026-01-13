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
| **[Seq: Upload (Legacy)](https://a-scolan.github.io/c4-hands-on-demo/#/project/legacy-vault-system/view/seq_upload_flow/)** | Legacy | Blocking upload | Synchronous processing (10-30s user wait) |
| **[Seq: Retrieval (Legacy)](https://a-scolan.github.io/c4-hands-on-demo/#/project/legacy-vault-system/view/seq_retrieval_flow/)** | Legacy | Document download | Direct NFS reads |
| **[Deployment (Legacy)](https://a-scolan.github.io/c4-hands-on-demo/#/project/legacy-vault-system/view/overview/)** | Legacy | Infrastructure | 2 monolith VMs, PostgreSQL, NFS, HAProxy |
| | | | |
| **[C1 Context](https://a-scolan.github.io/c4-hands-on-demo/#/project/refactored-vault-system/view/c1_context/)** | Refactored | System boundaries | Vault system, MinIO storage, VirusTotal API |
| **[C2 Container](https://a-scolan.github.io/c4-hands-on-demo/#/project/refactored-vault-system/view/c2_container/)** | Refactored | Microservices | Kong, Upload, Retrieval (merged), Worker, Queue, MongoDB |
| **[C3 Upload Service](https://a-scolan.github.io/c4-hands-on-demo/#/project/refactored-vault-system/view/c3_upload_service/)** | Refactored | Upload internals | Fail-fast validation before queuing |
| **[C3 Retrieval Service](https://a-scolan.github.io/c4-hands-on-demo/#/project/refactored-vault-system/view/c3_retrieval_service/)** | Refactored | Retrieval internals | Unified metadata + files with Redis cache |
| **[C3 Worker](https://a-scolan.github.io/c4-hands-on-demo/#/project/refactored-vault-system/view/c3_processing_worker/)** | Refactored | Worker internals | Async orchestrator: Scan → Encrypt → Store |
| **[Seq: Upload](https://a-scolan.github.io/c4-hands-on-demo/#/project/refactored-vault-system/view/seq_upload_flow/)** | Refactored | Async upload | Validate → Queue → Async processing (< 500ms response) |
| **[Seq: Retrieval](https://a-scolan.github.io/c4-hands-on-demo/#/project/refactored-vault-system/view/seq_retrieval_flow/)** | Refactored | Cached retrieval | Check cache → Fetch MinIO → Decrypt |
| **[Seq: HA Replication](https://a-scolan.github.io/c4-hands-on-demo/#/project/refactored-vault-system/view/seq_backup_flow/)** | Refactored | Disaster recovery | MinIO 3-node distributed replication |
| **[Deployment Overview](https://a-scolan.github.io/c4-hands-on-demo/#/project/refactored-vault-system/view/overview/)** | Refactored | Infrastructure | 7 zones, 11 VMs, Kong, MongoDB, MinIO, CI/CD |
| **[App Tier](https://a-scolan.github.io/c4-hands-on-demo/#/project/refactored-vault-system/view/app_tier/)** | Refactored | App deployment | Frontend, Upload, Retrieval services |
| **[CI/CD Pipeline](https://a-scolan.github.io/c4-hands-on-demo/#/project/refactored-vault-system/view/cicd/)** | Refactored | DevOps automation | GitLab, Harbor, schema migrations |

💡 **Tip:** Compare legacy vs. refactored: C1 Context → C2 Container → Seq Upload Flow to see the evolution

## 🚀 How to Use

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
- Explore sequence diagrams showing upload and retrieval flows

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

### VS Code Multi-Folder Workspace

The workspace file `c4-demo.code-workspace` configures VS Code to treat each project as a separate folder:

```bash
code c4-demo.code-workspace
```

This provides:
- IntelliSense for each project's LikeC4 files
- Separate workspace configurations
- Side-by-side editing of legacy and refactored architectures

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
│       ├── sequences.c4  # Dynamic sequence views
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
| **sequences.c4** | Dynamic interaction flows | [Sequences](https://likec4.dev/dsl/dynamic) |

**Key Principle:** Model = Business logic (deployment-agnostic) | Deployment = Physical infrastructure

### Naming Conventions

| Type | Pattern | Examples |
|------|---------|----------|
| Systems/Containers | PascalCase | `SecureVault`, `APIGateway` |
| Deployment nodes | snake_case | `api_gateway`, `app_tier` |
| FQN references | dots | `vault.api`, `prod.appTier.frontend_vm` |
| Tags | kebab-case | `#external`, `#legacy` |

[LikeC4 naming docs](https://likec4.dev/dsl/model#element-definitions)

### Relationship Types

| In model | In deployment | Examples |
|----------|---------------|----------|
| `uses`, `calls` | `http`, `https` | `vault.api -> vault.jobs` |
| `async`, `reads`, `writes` | `tcp`, `nfs`, `amqp` | `monitoring -[tcp]-> db_vm 'Scrape'` |

[LikeC4 relationships](https://likec4.dev/dsl/relationships)

### Best Practices Shown

✅ **Separation of concerns** - Logical and physical architectures separate  
✅ **Proper hierarchies** - Systems → containers → components; VMs → apps  
✅ **Shared specs** - Common definitions prevent duplication  
✅ **Clear relationships** - All connections have labels, kinds, and ports  
✅ **Consistent styling** - Colors, tags, and technologies documented  

### Quick Tips

- **Models are deployment-agnostic** - Same app can run on different infrastructure
- **Apps reference containers via `instanceOf`** - Links logical to physical
- **Shared specs prevent duplication** - Both projects inherit common definitions
- **Scoped wildcards in views** - Use `-> zone.*` and `zone.*->` for related elements only

For detailed syntax and examples, visit [LikeC4 Documentation](https://likec4.dev/dsl)
