# Copilot Workspace Instructions

Use skills from `.github/skills/` for all LikeC4 tasks. Skills are automatically applied based on user requests.

**MCP servers:**
- **LikeC4 MCP** (REQUIRED before changes): Use `read-project-summary` to load context, `search-element` to find elements, `find-relationships` to check connections, `open-view` to preview diagrams
- **Context7 MCP** (as needed): Query `/likec4/likec4` library docs when uncertain about syntax or features

**Workflow:** Always start with `understand-project-structure` skill when beginning work on a project.

## Phase 1 Skills (Granular Splits)

**Deployment Infrastructure (6 skills):**
- `structure-deployment-tiers` - Standard tiers (DMZ, AppTier, ProcTier, DataTier, optional zones)
- `configure-deployment-firewall` - Firewall rules and scaling strategies (horizontal vs vertical)
- `name-deployment-nodes` - VM/zone naming formulas and FQN conventions
- `write-deployment-specs` - Rich VLAN/network specs using markdown tables
- `style-view-elements` - Colors/shapes/icons from shared spec, style predicates
- `navigate-views` - Navigation (navigateTo drill-down), external links, view metadata

## Phase 2 Skills (Granular Splits)

**Deployment Modeling (2 skills):**
- `model-deployment-hierarchy` - Infrastructure hierarchy (environments, zones, VMs, apps) with proper nesting
- `model-deployment-relationships` - Inter-tier relationships with concrete protocols and ports

**View Design (2 skills):**
- `design-view-hierarchy` - C1/C2/C3 hierarchy with parent context and view organization
- `design-view-includes-neighbors` - Include neighboring elements (callers, dependencies, related elements)

**Rich Descriptions (2 skills):**
- `describe-system-elements` - System model descriptions using metadata blocks for structured data
- `describe-deployment-elements` - Deployment descriptions using markdown tables + metadata

## Phase 3 Skills (Granular Splits)

**C4 Modeling Process (3 skills):**
- `model-system-context` - C1 Context design (system boundary, actors, external systems)
- `model-system-containers` - C2 Containers design (major deployable units and relationships)
- `model-system-components` - C3 Components design (internal modules for complex containers)