# C4 Hands-On Demo - Multi-Project Architecture

🔗 **[View Live Diagrams on GitHub Pages](https://a-scolan.github.io/c4-hands-on-demo/)**

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
│       └── sequences.c4  # Dynamic sequence views
└── c4-demo.code-workspace
```
