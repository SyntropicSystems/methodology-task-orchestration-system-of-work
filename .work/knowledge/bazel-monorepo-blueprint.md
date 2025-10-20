# Bounded Bazel Monorepo Architecture Brief

**Version**: 1.0  
**Status**: Normative  
**Purpose**: Reusable pattern for creating atomic, hermetic, scalable monorepos with AI-friendly boundaries  
**Scope**: General methodology applicable to any multi-component project  

---

## Executive Summary

This brief defines a **Bounded Bazel Monorepo** pattern that provides:
- ✅ Atomic commits across all components (monorepo benefit)
- ✅ Hermetic isolation preventing pollution (submodule benefit)
- ✅ Bounded context for AI agents (scalability)
- ✅ Enforced dependencies via build system (reliability)
- ✅ Clear component ownership (maintainability)

The pattern leverages Bazel's native features (visibility, sandboxing, aspects) to achieve bounded execution without custom tooling overhead.

---

## 1. Core Principles

### 1.1 Hermetic Boundaries
- Each component declares explicit boundaries via Bazel BUILD files
- Dependencies are explicit and enforced at build time
- No implicit coupling through filesystem access

### 1.2 Component Type System
- Components are classified by **type** (platform, product, library, data, tooling)
- Types define allowed dependency relationships
- Type violations fail the build

### 1.3 Visibility as Enforcement
- Bazel's `visibility` attribute is the primary boundary mechanism
- Default visibility is `private` (deny-all)
- Explicit allowlists grant access

### 1.4 AI-Bounded Context
- Each component defines AI context limits
- Context builders respect component boundaries
- AI agents cannot accidentally pollute across boundaries

### 1.5 Build-Time Validation
- All invariants checked during build
- No runtime surprises
- Fail fast on violations

---

## 2. Component Type Taxonomy

### Standard Component Types

| Type | Purpose | Can Depend On | Can Be Depended On By | Visibility |
|------|---------|---------------|----------------------|------------|
| **platform** | Core infrastructure, SDKs | External packages only | library, product, tooling | Usually public |
| **library** | Shared utilities, common code | platform, library | library, product, tooling | Usually public |
| **product** | End-user applications | platform, library | Nothing (leaf nodes) | Private |
| **tooling** | Build tools, scripts | platform, library | Nothing (leaf nodes) | Private |
| **data** | Schemas, configs, data files | Nothing | All (as data files, not code) | Controlled |
| **test** | Test fixtures, test utilities | Anything | Nothing | Private |

### Dependency Rules Matrix

```
         platform  library  product  tooling  data  test
platform    ❌       ❌       ❌       ❌      ✅    ✅
library     ✅       ✅       ❌       ❌      ✅    ✅
product     ✅       ✅       ❌       ❌      ✅    ✅
tooling     ✅       ✅       ❌       ❌      ✅    ✅
data        ❌       ❌       ❌       ❌      ❌    ❌
test        ✅       ✅       ✅       ✅      ✅    ❌
```

Legend:
- ✅ = Allowed to depend on
- ❌ = Forbidden to depend on
- Data can be **read as files** but not **imported as code**

---

## 3. Workspace Structure Template

```
{workspace-name}/
├── WORKSPACE                      # Bazel workspace definition
├── WORKSPACE.bazel               # Alternative extension
├── BUILD.bazel                   # Root build file
├── .bazelrc                      # Bazel configuration
├── .bazelversion                 # Pin Bazel version (e.g., "7.0.0")
│
├── .workspace/                   # Workspace metadata
│   ├── README.md                # Workspace overview
│   ├── ARCHITECTURE.md          # Architecture decisions
│   ├── component_types.bzl      # Type definitions (Starlark)
│   ├── boundary_rules.bzl       # Boundary enforcement logic
│   └── templates/               # Component templates
│       ├── platform.BUILD.template
│       ├── product.BUILD.template
│       └── library.BUILD.template
│
├── platform/                     # Platform components (type: platform)
│   ├── BUILD.bazel
│   ├── {component-a}/
│   │   ├── BUILD.bazel
│   │   ├── COMPONENT.yaml       # Component metadata
│   │   ├── LLM-CONTEXT.md      # AI context definition
│   │   └── src/
│   │       └── BUILD.bazel
│   └── {component-b}/
│
├── libraries/                    # Shared libraries (type: library)
│   ├── BUILD.bazel
│   └── {library-name}/
│       ├── BUILD.bazel
│       ├── COMPONENT.yaml
│       └── LLM-CONTEXT.md
│
├── products/                     # End-user products (type: product)
│   ├── BUILD.bazel
│   └── {product-name}/
│       ├── BUILD.bazel
│       ├── COMPONENT.yaml
│       └── LLM-CONTEXT.md
│
├── data/                         # Data, schemas, configs (type: data)
│   ├── BUILD.bazel
│   └── {dataset-name}/
│       └── BUILD.bazel          # Exports filegroups, not code
│
├── tooling/                      # Build tools, scripts (type: tooling)
│   ├── BUILD.bazel
│   ├── boundary_checker/
│   │   └── BUILD.bazel
│   └── ai_context/
│       └── BUILD.bazel
│
└── tests/                        # Workspace-level tests (type: test)
    ├── BUILD.bazel
    ├── integration/
    └── boundary_validation/
```

---

## 4. Component Manifest (COMPONENT.yaml)

Every component MUST include a `COMPONENT.yaml` manifest:

```yaml
# Standard component manifest schema
component:
  name: "{component-name}"
  type: "platform|library|product|tooling|data|test"
  visibility: "public|internal|private"
  version: "1.0.0"
  
  owners:
    team: "{team-name}"
    primary: "{github-handle}"
    reviewers:
      - "{github-handle-1}"
      - "{github-handle-2}"
  
  boundaries:
    # Types this component can depend on
    can_depend_on:
      - "platform"
      - "library"
    
    # Types that can depend on this component
    can_be_depended_on_by:
      - "product"
      - "library"
    
    # Specific path restrictions (optional)
    forbidden_dependencies:
      - "products/**"
      - "platform/*/internal/**"
  
  ai_context:
    max_files: 50                 # Limit files in AI context
    priority_files:
      - "README.md"
      - "LLM-CONTEXT.md"
      - "src/api/**"
    exclude_patterns:
      - "**/*_test.*"
      - "**/node_modules/**"
      - "**/generated/**"
  
  metadata:
    description: "Brief component description"
    documentation: "https://docs.example.com/{component}"
    repository: "https://github.com/org/repo"
```

---

## 5. LLM-CONTEXT.md Template

Every component MUST include an `LLM-CONTEXT.md` file for AI agents:

```markdown
# LLM Context: {Component Name}

## Component Identity
- **Name**: {component-name}
- **Type**: {platform|library|product|tooling|data}
- **Version**: {semver}
- **Visibility**: {public|internal|private}

## Purpose
{2-3 sentence description of what this component does}

## Boundaries

### This component CAN:
- ✅ Depend on: {list allowed component types}
- ✅ Export: {what APIs/interfaces it provides}
- ✅ Access: {what resources it can access}

### This component CANNOT:
- ❌ Depend on: {forbidden component types}
- ❌ Import from: {specific forbidden paths}
- ❌ Modify: {protected resources}

## For AI Agents

### Context Limits
- **Max files in context**: {number}
- **Priority order**: {list priority files}
- **Exclude patterns**: {patterns to skip}

### Key Concepts
{Bullet points of key concepts AI needs to understand}

### Common Tasks
1. {Task 1}: {brief how-to}
2. {Task 2}: {brief how-to}
3. {Task 3}: {brief how-to}

### Anti-Patterns
- ❌ {Anti-pattern 1}
- ❌ {Anti-pattern 2}
- ❌ {Anti-pattern 3}

## Architecture

### Dependencies
{Visualize or list key dependencies}

### Public API
{Document public API surface}

### Internal Structure
{Brief overview of internal organization}

## Build & Test

### Build Target
```bash
bazel build //{component-path}:target-name
```

### Test Target
```bash
bazel test //{component-path}:test-name
```

### Common Commands
```bash
# {Command purpose}
bazel {command}
```

## Related Components
- **Depends on**: {list dependencies}
- **Provides APIs to**: {list dependents}
- **Related documentation**: {links}
```

---

## 6. BUILD.bazel Templates

### 6.1 Platform Component Template

```python
# platform/{component-name}/BUILD.bazel
load("@rules_python//python:defs.bzl", "py_library")

package(default_visibility = ["//visibility:private"])

# Main library (public API)
py_library(
    name = "{component-name}",
    srcs = glob(
        ["**/*.py"],
        exclude = [
            "**/*_test.py",
            "internal/**",
        ],
    ),
    visibility = [
        "//libraries:__subpackages__",
        "//products:__subpackages__",
        "//tooling:__subpackages__",
    ],
    deps = [
        # External dependencies only
        # Platform cannot depend on other internal components
    ],
)

# Internal APIs (private to this platform component)
py_library(
    name = "internal",
    srcs = glob(["internal/**/*.py"]),
    visibility = ["//platform/{component-name}:__subpackages__"],
    deps = [":{component-name}"],
)

# Tests
py_test(
    name = "test",
    srcs = glob(["**/*_test.py"]),
    deps = [
        ":{component-name}",
        ":internal",
    ],
)

# Documentation
filegroup(
    name = "docs",
    srcs = glob([
        "*.md",
        "docs/**/*.md",
    ]),
    visibility = ["//visibility:public"],
)
```

### 6.2 Library Component Template

```python
# libraries/{library-name}/BUILD.bazel
load("@rules_python//python:defs.bzl", "py_library")

package(default_visibility = ["//visibility:private"])

py_library(
    name = "{library-name}",
    srcs = glob(["**/*.py"], exclude=["**/*_test.py"]),
    visibility = [
        "//libraries:__subpackages__",  # Other libraries can use
        "//products:__subpackages__",   # Products can use
        "//tooling:__subpackages__",    # Tooling can use
    ],
    deps = [
        "//platform/{platform-name}",   # Can depend on platforms
        "//libraries/{other-lib}",      # Can depend on other libraries
    ],
)

py_test(
    name = "test",
    srcs = glob(["**/*_test.py"]),
    deps = [":{library-name}"],
)
```

### 6.3 Product Component Template

```python
# products/{product-name}/BUILD.bazel
load("@rules_python//python:defs.bzl", "py_binary", "py_library")

package(default_visibility = ["//visibility:private"])

py_library(
    name = "{product-name}_lib",
    srcs = glob(["**/*.py"], exclude=["main.py", "**/*_test.py"]),
    deps = [
        "//platform/{platform-name}",
        "//libraries/{library-name}",
        # Products CANNOT depend on other products
    ],
    # Products are never dependencies - no external visibility
)

py_binary(
    name = "{product-name}",
    srcs = ["main.py"],
    main = "main.py",
    deps = [":{product-name}_lib"],
    data = [
        "//data/{dataset}:schemas",  # Can read data files
    ],
)

py_test(
    name = "test",
    srcs = glob(["**/*_test.py"]),
    deps = [":{product-name}_lib"],
)
```

### 6.4 Data Component Template

```python
# data/{dataset-name}/BUILD.bazel
package(default_visibility = ["//visibility:private"])

# Data components export filegroups, NOT code libraries
filegroup(
    name = "schemas",
    srcs = glob(["schemas/**/*.yml"]),
    visibility = [
        "//products/{product-name}:__pkg__",
        "//libraries/{library-name}:__pkg__",
    ],
)

filegroup(
    name = "configs",
    srcs = glob(["configs/**/*.yaml"]),
    visibility = ["//products:__subpackages__"],
)

# Data components MUST NOT define py_library, ts_library, etc.
# Data is consumed as files, not imported as code
```

---

## 7. Workspace Configuration Files

### 7.1 WORKSPACE File Template

```python
# WORKSPACE
workspace(name = "{workspace_name}")

load("@bazel_tools//tools/build_defs/repo:http.bzl", "http_archive")

# Python rules
http_archive(
    name = "rules_python",
    sha256 = "{sha256}",
    strip_prefix = "rules_python-{version}",
    url = "https://github.com/bazelbuild/rules_python/releases/download/{version}/rules_python-{version}.tar.gz",
)

load("@rules_python//python:repositories.bzl", "py_repositories")
py_repositories()

# Load workspace-level definitions
load("//.workspace:component_types.bzl", "COMPONENT_TYPES")
load("//.workspace:boundary_rules.bzl", "validate_workspace")

# Validate workspace structure on load
validate_workspace()
```

### 7.2 .bazelrc Template

```bash
# .bazelrc - Bazel configuration

# === Build Settings ===
build --incompatible_strict_action_env
build --sandbox_default_allow_network=false
build --jobs=auto

# === Cache Settings ===
build --disk_cache=~/.cache/bazel/{workspace-name}

# Remote cache (optional - configure with your infrastructure)
# build --remote_cache=grpc://cache.example.com:9092
# build --remote_executor=grpc://exec.example.com:9092

# === Logging & Observability ===
build --build_event_json_file=bazel-build-events.json
build --experimental_build_event_expand_filesets

# === Test Settings ===
test --test_output=errors
test --test_summary=detailed
test --test_verbose_timeout_warnings

# === Custom Configurations ===

# Boundary validation
build:check-boundaries --aspects //.workspace:boundary_rules.bzl%boundary_check
build:check-boundaries --output_groups=+validation

# AI context generation
build:ai-context --aspects //.workspace:ai_context.bzl%ai_context_aspect
build:ai-context --output_groups=+ai_context

# Strict mode (all validations)
build:strict --config=check-boundaries
build:strict --@io_bazel_rules_go//go/config:race

# === Aliases ===
common --color=yes
common --curses=yes
```

### 7.3 .bazelversion File

```
7.0.0
```

Pin the Bazel version to ensure reproducibility across machines.

---

## 8. Enforcement: boundary_rules.bzl

```python
# .workspace/boundary_rules.bzl
"""Boundary validation rules for the workspace."""

load(":component_types.bzl", "COMPONENT_TYPES", "get_component_type")

def _boundary_check_impl(target, ctx):
    """Validate component boundaries."""
    
    if not hasattr(ctx.rule.attr, "deps"):
        return []
    
    source_package = target.label.package
    source_type = get_component_type(source_package)
    
    if source_type not in COMPONENT_TYPES:
        return []
    
    source_rules = COMPONENT_TYPES[source_type]
    allowed_types = source_rules.get("can_depend_on", [])
    
    violations = []
    
    for dep in ctx.rule.attr.deps:
        dep_package = dep.label.package
        dep_type = get_component_type(dep_package)
        
        if dep_type == "unknown":
            continue  # External dependency
        
        # Check type compatibility
        if dep_type not in allowed_types:
            violations.append(
                "BOUNDARY VIOLATION: %s (%s) cannot depend on %s (%s). " %
                (target.label, source_type, dep.label, dep_type) +
                "Allowed types: %s" % (allowed_types,)
            )
        
        # Check for circular dependencies (products/products)
        if source_type == "product" and dep_type == "product":
            violations.append(
                "BOUNDARY VIOLATION: %s (product) cannot depend on %s (product). " %
                (target.label, dep.label) +
                "Products must be independent."
            )
        
        # Check data is not imported as code
        if dep_type == "data" and ctx.rule.kind in ["py_library", "ts_library", "go_library"]:
            violations.append(
                "BOUNDARY VIOLATION: %s cannot import %s as code. " %
                (target.label, dep.label) +
                "Data must be accessed as files (use data= attribute)."
            )
    
    if violations:
        fail("\n\n".join(violations))
    
    return []

boundary_check = aspect(
    implementation = _boundary_check_impl,
    attr_aspects = ["deps"],
)

def validate_workspace():
    """Validate workspace structure on load."""
    # Can perform workspace-level checks here
    # E.g., ensure required directories exist
    pass
```

---

## 9. Component Types Definition

```python
# .workspace/component_types.bzl
"""Component type definitions and rules."""

COMPONENT_TYPES = {
    "platform": {
        "can_depend_on": [],
        "can_be_depended_on_by": ["library", "product", "tooling", "test"],
        "description": "Core infrastructure and SDKs",
    },
    "library": {
        "can_depend_on": ["platform", "library"],
        "can_be_depended_on_by": ["library", "product", "tooling", "test"],
        "description": "Shared utilities and common code",
    },
    "product": {
        "can_depend_on": ["platform", "library"],
        "can_be_depended_on_by": ["test"],
        "description": "End-user applications (leaf nodes)",
    },
    "tooling": {
        "can_depend_on": ["platform", "library"],
        "can_be_depended_on_by": ["test"],
        "description": "Build tools and scripts",
    },
    "data": {
        "can_depend_on": [],
        "can_be_depended_on_by": [],  # Special: can be read as files
        "description": "Schemas, configs, and data files",
    },
    "test": {
        "can_depend_on": ["platform", "library", "product", "tooling"],
        "can_be_depended_on_by": [],
        "description": "Test fixtures and utilities",
    },
}

def get_component_type(package_path):
    """Determine component type from package path."""
    if package_path.startswith("platform/"):
        return "platform"
    elif package_path.startswith("libraries/"):
        return "library"
    elif package_path.startswith("products/"):
        return "product"
    elif package_path.startswith("tooling/"):
        return "tooling"
    elif package_path.startswith("data/"):
        return "data"
    elif package_path.startswith("tests/"):
        return "test"
    else:
        return "unknown"
```

---

## 10. Makefile Helpers

```makefile
# Makefile - Workspace-level commands
.PHONY: help
help:
	@echo "Bounded Bazel Monorepo Commands"
	@echo "================================"
	@echo "  make build              - Build all targets"
	@echo "  make test               - Run all tests"
	@echo "  make check-boundaries   - Validate boundaries"
	@echo "  make ai-context         - Generate AI contexts"
	@echo "  make graph              - Visualize dependencies"
	@echo "  make clean              - Clean build artifacts"

.PHONY: build
build:
	bazel build //...

.PHONY: test
test:
	bazel test //...

.PHONY: check-boundaries
check-boundaries:
	@echo "🔍 Validating workspace boundaries..."
	bazel build --config=check-boundaries //...

.PHONY: ai-context
ai-context:
	@echo "🤖 Generating AI context files..."
	bazel build --config=ai-context //...
	@echo "Context files generated in bazel-bin/"

.PHONY: graph
graph:
	@echo "📊 Generating dependency graph..."
	bazel query --output=graph 'deps(//...)' | dot -Tpng > workspace-deps.png
	@echo "Graph saved to workspace-deps.png"

.PHONY: clean
clean:
	bazel clean

.PHONY: new-component
new-component:
	@echo "Creating new component: $(name) (type: $(type))"
	@.workspace/scripts/new-component.sh $(type) $(name)
```

---

## 11. Component Scaffolding Script

```bash
#!/bin/bash
# .workspace/scripts/new-component.sh
# Create a new component from template

set -e

TYPE=$1
NAME=$2

if [ -z "$TYPE" ] || [ -z "$NAME" ]; then
    echo "Usage: new-component.sh <type> <name>"
    echo "Types: platform, library, product, tooling, data"
    exit 1
fi

# Determine directory based on type
case $TYPE in
    platform)
        DIR="platform/$NAME"
        ;;
    library)
        DIR="libraries/$NAME"
        ;;
    product)
        DIR="products/$NAME"
        ;;
    tooling)
        DIR="tooling/$NAME"
        ;;
    data)
        DIR="data/$NAME"
        ;;
    *)
        echo "Invalid type: $TYPE"
        exit 1
        ;;
esac

# Create directory
mkdir -p "$DIR"

# Copy template BUILD file
cp ".workspace/templates/${TYPE}.BUILD.template" "$DIR/BUILD.bazel"

# Create COMPONENT.yaml from template
cat > "$DIR/COMPONENT.yaml" <<EOF
component:
  name: "$NAME"
  type: "$TYPE"
  visibility: "internal"
  version: "0.1.0"
  
  owners:
    team: "your-team"
    primary: "your-handle"
  
  boundaries:
    can_depend_on: []
    can_be_depended_on_by: []
  
  ai_context:
    max_files: 50
    priority_files:
      - "README.md"
      - "LLM-CONTEXT.md"
  
  metadata:
    description: "TODO: Describe $NAME"
EOF

# Create LLM-CONTEXT.md from template
cp ".workspace/templates/LLM-CONTEXT.template.md" "$DIR/LLM-CONTEXT.md"
sed -i "" "s/{{NAME}}/$NAME/g" "$DIR/LLM-CONTEXT.md"
sed -i "" "s/{{TYPE}}/$TYPE/g" "$DIR/LLM-CONTEXT.md"

# Create README
cat > "$DIR/README.md" <<EOF
# $NAME

TODO: Describe this $TYPE component

## Build

\`\`\`bash
bazel build //$DIR:$NAME
\`\`\`

## Test

\`\`\`bash
bazel test //$DIR:test
\`\`\`
EOF

echo "✅ Created new $TYPE component: $DIR"
echo "Next steps:"
echo "  1. Edit $DIR/COMPONENT.yaml"
echo "  2. Edit $DIR/LLM-CONTEXT.md"
echo "  3. Implement your component"
echo "  4. Run: make check-boundaries"
```

---

## 12. Implementation Checklist

### Phase 1: Foundation
- [ ] Create workspace root structure
- [ ] Set up WORKSPACE and BUILD.bazel files
- [ ] Configure .bazelrc with hermetic settings
- [ ] Pin Bazel version in .bazelversion
- [ ] Create component_types.bzl with type definitions
- [ ] Create boundary_rules.bzl with validation logic

### Phase 2: Templates
- [ ] Create BUILD.bazel templates for each component type
- [ ] Create COMPONENT.yaml template
- [ ] Create LLM-CONTEXT.md template
- [ ] Create new-component.sh scaffolding script
- [ ] Test component creation workflow

### Phase 3: Enforcement
- [ ] Test boundary_check aspect with violations
- [ ] Configure pre-commit hooks (optional)
- [ ] Set up CI pipeline for boundary validation
- [ ] Document violation error messages
- [ ] Create boundary debugging guide

### Phase 4: AI Integration
- [ ] Create AI context builder aspect
- [ ] Test context generation for different component types
- [ ] Integrate with AI tools (Cline, Cursor, etc.)
- [ ] Create AI-specific documentation
- [ ] Test context size limits

### Phase 5: Documentation
- [ ] Write ARCHITECTURE.md for workspace
- [ ] Create component authoring guide
- [ ] Document boundary rules and rationale
- [ ] Create troubleshooting guide
- [ ] Add examples for common patterns

### Phase 6: Tooling
- [ ] Add Makefile helpers
- [ ] Create dependency visualization scripts
- [ ] Add component linting tools
- [ ] Create migration guides
- [ ] Build component discovery tools

---

## 13. Success Metrics

Track these metrics to validate the pattern:

1. **Boundary Violations**: Should trend to zero after initial migration
2. **Build Times**: Should decrease due to caching
3. **CI Times**: Should improve with incremental builds
4. **Refactoring Velocity**: Time to make cross-component changes
5. **Onboarding Time**: Time for new developers to be productive
6. **AI Agent Success Rate**: % of AI-proposed changes that build correctly
7. **Component Coupling**: Number of dependencies per component
8. **Test Isolation**: % of tests that can run independently

---

## 14. Anti-Patterns to Avoid

### ❌ Visibility Backdoors
```python
# DON'T make everything public
visibility = ["//visibility:public"]

# DO be explicit about consumers
visibility = ["//products/specific:__pkg__"]
```

### ❌ Type Confusion
```python
# DON'T put products in platform/
platform/my-app/  # Wrong - apps are products

# DO respect the taxonomy
products/my-app/  # Correct
```

### ❌ Circular Dependencies
```python
# DON'T create cycles
deps = ["//libraries/a"],  # where a depends on current

# DO use dependency injection
deps = ["//platform/interfaces"],
```

### ❌ Data as Code
```python
# DON'T import data as code
deps = ["//data/schemas:lib"],

# DO read data as files
data = ["//data/schemas:yaml_files"],
```

---

## 15. Next Steps

To apply this pattern to a new project:

1. **Copy workspace template** from reference implementation
2. **Customize component_types.bzl** for your domain
3. **Create first platform component** (your core SDK)
4. **Validate boundaries** with test violations
5. **Document decisions** in ARCHITECTURE.md
6. **Train team** on the pattern and tools
7. **Iterate** as you discover project-specific needs

---

## Appendix A: Reference Implementations

- Minimal example: `github.com/your-org/bazel-monorepo-template`
- Full example: `github.com/your-org/production-monorepo`
- Migration guide: `docs/migrating-to-bazel-monorepo.md`

## Appendix B: Bazel Resources

- Official docs: https://bazel.build
- Rules catalog: https://registry.bazel.build
- Community: https://slack.bazel.build

---

*End of Brief*"