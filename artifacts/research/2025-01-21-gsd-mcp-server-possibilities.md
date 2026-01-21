# GSD MCP Server: Superpowers Unlocked

**Created:** 2025-01-21
**Status:** Exploration
**Question:** What becomes possible with an MCP server that's impossible today?

---

## The Fundamental Shift: From Reactive to Interactive

### Current Architecture (Hooks + Commands)

```
┌─────────────────────────────────────────────────────────────────┐
│                        CURRENT: ONE-WAY                          │
│                                                                  │
│   User ──────▶ Claude ──────▶ Tools ──────▶ Hooks               │
│                   │                            │                 │
│                   │                            ▼                 │
│                   │                      [Side effects]          │
│                   │                      - Update index          │
│                   │                      - Queue entity          │
│                   │                                              │
│                   ▼                                              │
│              [Outputs]                                           │
│              - Code                                              │
│              - Explanations                                      │
│                                                                  │
│   CLAUDE CANNOT:                                                 │
│   - Request specific intel mid-thought                          │
│   - Check conventions before writing                            │
│   - Query dependencies dynamically                              │
│   - Validate against project patterns                           │
│   - Ask "what would break if..."                                │
└─────────────────────────────────────────────────────────────────┘
```

### MCP Architecture (Bidirectional)

```
┌─────────────────────────────────────────────────────────────────┐
│                       MCP: BIDIRECTIONAL                         │
│                                                                  │
│   User ◀────▶ Claude ◀────▶ MCP Server ◀────▶ Intel Layer       │
│                   │              │                               │
│                   │              ├── Tools (Claude can call)     │
│                   │              ├── Resources (Claude can read) │
│                   │              └── Prompts (Pre-built context) │
│                   │                                              │
│                   ▼                                              │
│              [Outputs]                                           │
│              - Code (validated against patterns)                 │
│              - Explanations (with full context)                  │
│                                                                  │
│   CLAUDE CAN NOW:                                                │
│   ✓ Ask "what files handle auth?" mid-conversation              │
│   ✓ Check "does this follow conventions?" before writing        │
│   ✓ Query "what depends on this file?"                          │
│   ✓ Request "show me similar implementations"                   │
│   ✓ Validate "would this change break anything?"                │
└─────────────────────────────────────────────────────────────────┘
```

---

## Superpowers Unlocked

### 1. On-Demand Intel Queries (Claude Asks Questions)

**Current limitation:** Claude must work with whatever context was injected at session start. If it needs to know about a file it didn't anticipate, it has to read the raw file.

**With MCP:**

```
Claude thinking: "User wants to add caching. I should understand the current
data flow before proposing changes."

Claude calls: gsd_query_intel({
  query: "files related to caching OR redis OR data fetching",
  include_purposes: true,
  max_results: 10
})

MCP returns: {
  files: [
    {
      path: "src/lib/cache.ts",
      purpose: "Redis cache wrapper with TTL support",
      exports: ["cacheGet", "cacheSet", "cacheInvalidate"],
      dependents: 8
    },
    {
      path: "src/services/user.service.ts",
      purpose: "User CRUD with optional caching",
      uses_cache: true,
      cache_patterns: ["getUserById cached for 5min"]
    }
  ]
}

Claude now knows exactly how caching works in this project before writing code.
```

**Why this matters:** Claude makes better decisions with relevant context. Currently it either gets too much (everything) or too little (just what was pre-injected).

---

### 2. Pre-Write Convention Validation

**Current limitation:** Claude writes code, then hooks detect patterns after the fact. If Claude violates conventions, the code is already written.

**With MCP:**

```
Claude thinking: "I need to create a new service for payments."

Claude calls: gsd_get_conventions({
  type: "service",
  directory: "src/services"
})

MCP returns: {
  file_pattern: "*.service.ts",
  naming: "PascalCase class, camelCase methods",
  structure: {
    imports: ["Injectable decorator from @nestjs/common"],
    class: "export class {Name}Service",
    constructor: "constructor injection for dependencies",
    methods: "async, return Promise<Result<T, Error>>"
  },
  example_file: "src/services/user.service.ts",
  example_snippet: "..."
}

Claude writes payment.service.ts following exact conventions.
```

**Why this matters:** Prevention > correction. Claude matches project style on first write.

---

### 3. Real-Time Blast Radius Analysis

**Current limitation:** Claude doesn't know what will break when it modifies a file. It might change a function signature without realizing 40 files import it.

**With MCP:**

```
Claude thinking: "I need to change the query() function signature in db.ts
to add a timeout parameter."

Claude calls: gsd_blast_radius({
  file: "src/lib/db.ts",
  export: "query",
  change_type: "signature_change"
})

MCP returns: {
  direct_dependents: 42,
  transitive_dependents: 67,
  high_risk_files: [
    "src/services/payment.service.ts (critical path)",
    "src/api/health.ts (monitoring)"
  ],
  suggested_approach: "Add optional parameter with default value to maintain backward compatibility",
  migration_scope: "If breaking change: 42 files need updates"
}

Claude decides to make timeout optional with default, avoiding breaking changes.
```

**Why this matters:** Claude can make informed decisions about backward compatibility and change scope.

---

### 4. Semantic Code Search (Beyond Grep)

**Current limitation:** Claude can grep for text patterns but can't ask "find code that does X" semantically.

**With MCP:**

```
Claude thinking: "User wants error handling like we do elsewhere. Let me find examples."

Claude calls: gsd_semantic_search({
  query: "error handling patterns for API endpoints",
  code_type: "pattern",
  limit: 3
})

MCP returns: {
  patterns: [
    {
      name: "Try-catch with Result type",
      files: ["src/api/users.ts", "src/api/payments.ts"],
      snippet: "try { ... } catch (e) { return Result.err(e) }",
      frequency: "89% of API handlers"
    },
    {
      name: "Express error middleware",
      file: "src/middleware/error.ts",
      description: "Central error handler, logs + formats response"
    }
  ]
}

Claude follows the established error handling pattern.
```

**Why this matters:** Semantic search finds intent, not just text. "How do we handle errors?" vs "grep for catch".

---

### 5. Live Phase State Management

**Current limitation:** Phase state is in markdown files. Claude has to read files to know phase status, and write files to update it.

**With MCP:**

```
Claude calls: gsd_phase_status()

MCP returns: {
  current_phase: 3,
  phase_name: "Authentication System",
  status: "in_progress",
  tasks: [
    { name: "JWT implementation", status: "completed" },
    { name: "OAuth integration", status: "in_progress" },
    { name: "Session management", status: "pending" }
  ],
  blockers: [],
  next_phase: {
    number: 4,
    name: "Payment Integration",
    dependencies: ["Phase 3 OAuth required for Stripe Connect"]
  }
}

Claude calls: gsd_update_task({
  phase: 3,
  task: "OAuth integration",
  status: "completed",
  notes: "Implemented Google and GitHub providers"
})
```

**Why this matters:** Structured phase management without file parsing. Claude can check/update status naturally.

---

### 6. Intelligent File Recommendations

**Current limitation:** When Claude needs to create a file, it guesses where it should go based on conventions it may or may not remember.

**With MCP:**

```
Claude thinking: "User wants a new endpoint for webhooks."

Claude calls: gsd_recommend_location({
  type: "api_endpoint",
  name: "webhooks",
  description: "Handle incoming Stripe webhooks"
})

MCP returns: {
  recommended_path: "src/api/webhooks.ts",
  reasoning: "API endpoints go in src/api/, following {name}.ts pattern",
  related_files: [
    "src/api/payments.ts (similar Stripe integration)",
    "src/services/payment.service.ts (business logic should go here)",
    "src/types/stripe.ts (type definitions)"
  ],
  suggested_structure: {
    api_file: "Route handlers only, delegate to service",
    service_file: "src/services/webhook.service.ts for processing logic",
    types_file: "Add webhook types to src/types/stripe.ts"
  }
}
```

**Why this matters:** Claude doesn't just know conventions—it gets specific recommendations for the task at hand.

---

### 7. Cross-File Refactoring Intelligence

**Current limitation:** When refactoring, Claude has to manually track all affected files. Easy to miss references.

**With MCP:**

```
User: "Rename UserService to AccountService throughout the codebase"

Claude calls: gsd_refactor_analysis({
  type: "rename",
  from: "UserService",
  to: "AccountService",
  scope: "class"
})

MCP returns: {
  occurrences: [
    { file: "src/services/user.service.ts", line: 15, type: "definition" },
    { file: "src/api/users.ts", line: 3, type: "import" },
    { file: "src/api/users.ts", line: 45, type: "usage" },
    { file: "src/api/auth.ts", line: 8, type: "import" },
    // ... 23 more
  ],
  file_renames: [
    { from: "src/services/user.service.ts", to: "src/services/account.service.ts" }
  ],
  test_files_affected: [
    "src/__tests__/user.service.test.ts"
  ],
  entity_updates_needed: [
    ".planning/intel/entities/src-services-user-service.md"
  ],
  total_changes: 27
}

Claude performs complete refactoring without missing any reference.
```

**Why this matters:** Refactoring with confidence. No forgotten imports or broken references.

---

### 8. Test Coverage Intelligence

**Current limitation:** Claude doesn't know what's tested and what isn't without reading all test files.

**With MCP:**

```
Claude calls: gsd_test_coverage({
  file: "src/services/payment.service.ts"
})

MCP returns: {
  test_file: "src/__tests__/payment.service.test.ts",
  coverage: {
    functions: {
      "createPayment": { tested: true, test_count: 3 },
      "refundPayment": { tested: true, test_count: 2 },
      "getPaymentHistory": { tested: false },  // ← Gap!
      "cancelSubscription": { tested: true, test_count: 1 }
    },
    line_coverage: "78%",
    branch_coverage: "65%"
  },
  suggestions: [
    "getPaymentHistory has no tests",
    "cancelSubscription only has happy path test"
  ]
}

Claude knows exactly what tests to write.
```

**Why this matters:** Claude can identify and fill test gaps intelligently.

---

### 9. Architecture Validation

**Current limitation:** Claude might create code that violates architectural boundaries (e.g., API layer directly accessing database).

**With MCP:**

```
Claude is about to write: import { query } from '../lib/db' in src/api/users.ts

Claude calls: gsd_validate_architecture({
  file: "src/api/users.ts",
  imports: ["../lib/db"]
})

MCP returns: {
  valid: false,
  violation: "API layer should not import directly from data layer",
  rule: "src/api/** cannot import from src/lib/db*",
  suggested_fix: "Import from service layer instead: src/services/user.service.ts",
  architecture: {
    "src/api": ["can import: src/services, src/middleware, src/types"],
    "src/services": ["can import: src/lib, src/models, src/types"],
    "src/lib": ["can import: src/config, src/types"]
  }
}

Claude fixes the import before writing.
```

**Why this matters:** Architectural integrity enforced proactively, not discovered in code review.

---

### 10. Smart Conflict Detection

**Current limitation:** Claude doesn't know if another part of the codebase already does what it's about to implement.

**With MCP:**

```
User: "Add a function to format dates as YYYY-MM-DD"

Claude calls: gsd_find_existing({
  description: "format date as YYYY-MM-DD",
  type: "function"
})

MCP returns: {
  existing_implementations: [
    {
      file: "src/utils/date.ts",
      function: "formatDate",
      signature: "formatDate(date: Date, format: string): string",
      supports_requested_format: true,
      usage_example: "formatDate(new Date(), 'YYYY-MM-DD')"
    }
  ],
  recommendation: "Use existing formatDate() from src/utils/date.ts instead of creating new function"
}

Claude: "This already exists! You can use formatDate() from src/utils/date.ts"
```

**Why this matters:** Prevents duplication. DRY enforcement before code is written.

---

### 11. Documentation Sync

**Current limitation:** When code changes, documentation gets stale. No automatic detection.

**With MCP:**

```
After Claude modifies payment.service.ts:

MCP automatically: gsd_check_doc_sync({
  modified_file: "src/services/payment.service.ts"
})

MCP detects: {
  stale_docs: [
    {
      doc: "docs/api/payments.md",
      reason: "References old function signature for createPayment()",
      last_doc_update: "2025-01-10",
      last_code_update: "2025-01-21"
    },
    {
      doc: ".planning/intel/entities/src-services-payment-service.md",
      reason: "Entity missing new export: validatePaymentMethod",
      action: "Queue for regeneration"
    }
  ]
}

Claude: "I've updated the code. Note: docs/api/payments.md may need updating
to reflect the new createPayment signature."
```

**Why this matters:** Documentation drift detected and surfaced automatically.

---

### 12. Learning From User Corrections

**Current limitation:** When user corrects Claude, the correction is lost after the session.

**With MCP:**

```
User: "No, we don't use console.log for logging. Use the logger from src/lib/logger.ts"

Claude calls: gsd_learn_correction({
  category: "logging",
  wrong_pattern: "console.log",
  correct_pattern: "import { logger } from 'src/lib/logger'; logger.info()",
  context: "All logging should use structured logger"
})

MCP stores in project memory.

Future session, Claude is about to write console.log:

Claude calls: gsd_check_patterns({ code: "console.log('user created')" })

MCP returns: {
  warnings: [
    {
      pattern: "console.log",
      correction: "Use logger from src/lib/logger.ts",
      learned_from: "User correction on 2025-01-21"
    }
  ]
}

Claude uses logger instead.
```

**Why this matters:** Claude learns project-specific preferences permanently.

---

## MCP Server Design

### Tools (Claude Can Call)

| Tool | Purpose | Returns |
|------|---------|---------|
| `gsd_query_intel` | Search files by keyword/purpose | Matching files with context |
| `gsd_blast_radius` | Analyze change impact | Dependent files, risk assessment |
| `gsd_get_conventions` | Get patterns for file type | Structure, naming, examples |
| `gsd_validate_architecture` | Check import is allowed | Valid/invalid with explanation |
| `gsd_find_existing` | Find similar implementations | Existing code that matches |
| `gsd_recommend_location` | Suggest file placement | Path + reasoning + related files |
| `gsd_refactor_analysis` | Analyze refactoring scope | All occurrences, file renames needed |
| `gsd_phase_status` | Get current phase state | Phase, tasks, blockers |
| `gsd_update_task` | Update task status | Confirmation |
| `gsd_test_coverage` | Get test coverage for file | Tested/untested functions |
| `gsd_semantic_search` | Find code by intent | Patterns and examples |
| `gsd_learn_correction` | Store user preference | Confirmation |
| `gsd_check_patterns` | Validate code against learned patterns | Warnings |

### Resources (Claude Can Read)

| Resource | URI Pattern | Content |
|----------|-------------|---------|
| Project summary | `gsd://intel/summary` | Full summary.md |
| Entity details | `gsd://intel/entity/{slug}` | Single entity content |
| Phase plan | `gsd://phase/{number}/plan` | Phase plan markdown |
| Conventions | `gsd://conventions/{type}` | Conventions for file type |
| Architecture rules | `gsd://architecture/rules` | Import/dependency rules |
| Session history | `gsd://sessions/recent` | Last 5 sessions summary |

### Prompts (Pre-Built Contexts)

| Prompt | Purpose |
|--------|---------|
| `plan-phase` | Full context for phase planning |
| `code-review` | Context for reviewing changes |
| `refactor` | Context for safe refactoring |
| `debug` | Context for debugging with dependencies |

---

## Implementation Approach

### Phase 1: Core Query Tools

```typescript
// gsd-mcp-server/src/tools/query.ts

import { z } from 'zod';

export const queryIntelTool = {
  name: 'gsd_query_intel',
  description: 'Search codebase intelligence for files matching query',
  inputSchema: z.object({
    query: z.string().describe('Keywords or natural language query'),
    include_purposes: z.boolean().default(true),
    include_exports: z.boolean().default(false),
    max_results: z.number().default(10),
    type_filter: z.enum(['service', 'api', 'util', 'component', 'model', 'all']).default('all')
  }),

  async execute({ query, include_purposes, max_results, type_filter }) {
    const graph = await loadGraph();
    const index = await loadIndex();

    // Keyword extraction
    const keywords = extractKeywords(query);

    // Search nodes by path and purpose
    const results = graph.searchNodes({
      keywords,
      typeFilter: type_filter === 'all' ? null : type_filter,
      limit: max_results
    });

    // Enrich with entity data if requested
    if (include_purposes) {
      for (const result of results) {
        const entity = await loadEntity(result.id);
        result.purpose = entity?.purpose;
        result.exports = include_exports ? entity?.exports : undefined;
      }
    }

    return { files: results };
  }
};
```

### Phase 2: Validation Tools

```typescript
// gsd-mcp-server/src/tools/validate.ts

export const validateArchitectureTool = {
  name: 'gsd_validate_architecture',
  description: 'Check if an import violates architecture rules',
  inputSchema: z.object({
    file: z.string().describe('File that would contain the import'),
    imports: z.array(z.string()).describe('Import paths to validate')
  }),

  async execute({ file, imports }) {
    const rules = await loadArchitectureRules();
    const violations = [];

    for (const importPath of imports) {
      const resolvedPath = resolveImport(file, importPath);
      const sourceLayer = getLayer(file);
      const targetLayer = getLayer(resolvedPath);

      if (!rules.canImport(sourceLayer, targetLayer)) {
        violations.push({
          import: importPath,
          violation: `${sourceLayer} cannot import from ${targetLayer}`,
          suggestion: rules.getSuggestion(sourceLayer, targetLayer)
        });
      }
    }

    return {
      valid: violations.length === 0,
      violations
    };
  }
};
```

### Phase 3: Learning System

```typescript
// gsd-mcp-server/src/tools/learn.ts

export const learnCorrectionTool = {
  name: 'gsd_learn_correction',
  description: 'Store a user correction for future reference',
  inputSchema: z.object({
    category: z.string().describe('Category: logging, error-handling, naming, etc'),
    wrong_pattern: z.string().describe('The pattern to avoid'),
    correct_pattern: z.string().describe('The preferred pattern'),
    context: z.string().optional().describe('Additional context')
  }),

  async execute({ category, wrong_pattern, correct_pattern, context }) {
    const memory = await loadProjectMemory();

    memory.corrections.push({
      category,
      wrong_pattern,
      correct_pattern,
      context,
      learned_at: new Date().toISOString()
    });

    await saveProjectMemory(memory);

    return {
      stored: true,
      message: `Learned: Use "${correct_pattern}" instead of "${wrong_pattern}"`
    };
  }
};
```

---

## What Becomes Possible: User Experience

### Before MCP (Current)

```
User: Add rate limiting to the auth endpoints

Claude: I'll add rate limiting to the auth endpoints.
[Reads src/api/auth.ts]
[Writes rate limiting code]
[May or may not follow project patterns]
[May create duplicate utility if one exists]
[May violate architecture rules]
[May miss related files that need updates]

User: Actually we already have a rate limiter in middleware...
User: Also, we don't import directly from lib in api files...
User: And you need to update the tests too...
```

### After MCP

```
User: Add rate limiting to the auth endpoints

Claude thinking: Let me check what exists and understand the architecture.

Claude calls: gsd_find_existing({ description: "rate limiting middleware" })
→ Found: src/middleware/rate-limit.ts with rateLimiter()

Claude calls: gsd_validate_architecture({
  file: "src/api/auth.ts",
  imports: ["../middleware/rate-limit"]
})
→ Valid: API layer can import from middleware

Claude calls: gsd_blast_radius({ file: "src/api/auth.ts" })
→ 3 test files affected

Claude: I found an existing rate limiter at src/middleware/rate-limit.ts.
I'll apply it to the auth routes. This will also require updating
3 test files. Here's my plan:

1. Import rateLimiter in src/api/auth.ts
2. Apply to /login and /register routes
3. Update tests in src/__tests__/auth.test.ts

[Writes correct code on first try]
```

---

## Comparison: Hooks vs Commands vs MCP

| Capability | Hooks | Commands | MCP |
|------------|-------|----------|-----|
| React to file changes | ✅ | ❌ | ✅ |
| Claude-initiated queries | ❌ | ❌ | ✅ |
| Pre-write validation | ❌ | ❌ | ✅ |
| Dynamic context loading | ❌ | ✅ (user must invoke) | ✅ |
| Learning from corrections | ❌ | ❌ | ✅ |
| Architectural validation | ❌ | ❌ | ✅ |
| Semantic search | ❌ | Limited | ✅ |
| Cross-session memory | Limited | ❌ | ✅ |
| Refactoring analysis | ❌ | ❌ | ✅ |
| Test coverage queries | ❌ | ❌ | ✅ |

---

## Summary: The Superpower Stack

| Superpower | Impact | Difficulty |
|------------|--------|------------|
| **On-demand intel queries** | Claude asks instead of guessing | Medium |
| **Pre-write validation** | Right code on first try | Medium |
| **Blast radius analysis** | Confident refactoring | Medium |
| **Semantic code search** | Find by intent, not text | Hard |
| **Architecture enforcement** | No rule violations | Medium |
| **Duplicate detection** | No redundant code | Medium |
| **Learning from corrections** | Permanent preferences | Easy |
| **Test coverage intelligence** | Smart test writing | Medium |
| **Cross-file refactoring** | Complete, safe renames | Medium |
| **Documentation sync** | No stale docs | Easy |

**The fundamental shift:** Claude moves from "write and hope" to "understand, validate, then write."

---

## Next Steps

1. **Prototype core query tools** - gsd_query_intel, gsd_blast_radius
2. **Add validation tools** - gsd_validate_architecture, gsd_check_patterns
3. **Implement learning system** - gsd_learn_correction, persistent memory
4. **Build semantic search** - Beyond keyword matching
5. **Integrate with existing hooks** - MCP server reads same intel data

The MCP server doesn't replace hooks—it complements them:
- **Hooks:** Capture data automatically (reactive)
- **MCP:** Query data on demand (interactive)

---

## PART 2: REVISED PERSPECTIVE (After Deep System Analysis)

**Updated:** 2025-01-21
**Context:** After comprehensive analysis of the full GSD system (commands, agents, workflows, hooks, templates)

---

## Critical Realization: MCP Serves the Agent System

The initial MCP proposal focused on ad-hoc queries during coding. But GSD is **agent-orchestrated** — the real consumers of MCP tools are:

1. **gsd-planner** - Creates executable plans
2. **gsd-executor** - Executes tasks with fresh context
3. **gsd-verifier** - Validates phase goals achieved
4. **gsd-phase-researcher** - Researches implementation approaches

These agents would gain **massive** capabilities from MCP tools.

---

## How GSD Currently Works (The Full Picture)

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         GSD ORCHESTRATION SYSTEM                         │
│                                                                          │
│  ┌─────────────┐   ┌─────────────┐   ┌─────────────┐   ┌─────────────┐ │
│  │  Commands   │──▶│  Workflows  │──▶│   Agents    │──▶│   Hooks     │ │
│  │             │   │             │   │             │   │             │ │
│  │ /gsd:plan   │   │ plan-phase  │   │ gsd-planner │   │ intel-index │ │
│  │ /gsd:exec   │   │ exec-phase  │   │ gsd-executor│   │ intel-session│ │
│  │ /gsd:verify │   │ verify-phase│   │ gsd-verifier│   │ intel-prune │ │
│  └─────────────┘   └─────────────┘   └─────────────┘   └─────────────┘ │
│         │                 │                 │                 │         │
│         ▼                 ▼                 ▼                 ▼         │
│  ┌──────────────────────────────────────────────────────────────────┐  │
│  │                        .planning/ BRAIN                           │  │
│  │                                                                   │  │
│  │  PROJECT.md │ REQUIREMENTS.md │ ROADMAP.md │ STATE.md │ intel/   │  │
│  │  (vision)   │ (what to build) │ (phases)   │ (now)    │ (learn)  │  │
│  └──────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────┘
```

### What Each Agent Currently Receives

| Agent | Context Loaded | What's Missing |
|-------|----------------|----------------|
| **gsd-planner** | PROJECT.md, ROADMAP.md, phase CONTEXT.md, intel summary | Can't query "how did we handle X before?" |
| **gsd-executor** | PROJECT.md, ROADMAP.md, STATE.md, PLAN.md, intel summary | Can't check blast radius before editing |
| **gsd-verifier** | PLAN.md must_haves, source files | Can't query "is behavior X implemented?" |
| **gsd-researcher** | Web search, file reads | Can't query "what patterns exist in our codebase?" |

### The Gap: Static Loading vs Dynamic Queries

**Current:** Agents load fixed context at spawn time. If they need different info mid-task, they must read raw files.

**With MCP:** Agents query exactly what they need, when they need it. Fresh, relevant, structured.

---

## MCP Tools Designed for GSD Agents

### For gsd-planner: "What Should I Plan?"

```typescript
// Before creating a plan, planner queries:

gsd_phase_relevant_files({
  phase_goal: "Add rate limiting to authentication",
  include_purposes: true,
  include_blast_radius: true
})

// Returns:
{
  relevant_files: [
    {
      path: "src/services/auth.service.ts",
      purpose: "JWT auth, session management",
      blast_radius: 28,
      recommendation: "Core auth logic - add rate limit here"
    },
    {
      path: "src/middleware/rate-limit.ts",
      purpose: "Express rate limiting (currently unused)",
      blast_radius: 0,
      recommendation: "Existing middleware - wire into auth routes"
    }
  ],
  suggested_approach: "Use existing rate-limit.ts middleware, apply to auth routes",
  patterns_from_codebase: "Middleware applied in route files, not services"
}

// Planner creates better plans with this intelligence
```

### For gsd-executor: "What Will Break?"

```typescript
// Before editing a file, executor queries:

gsd_pre_edit_check({
  file: "src/lib/db.ts",
  planned_change: "Add timeout parameter to query()",
  change_type: "signature_change"
})

// Returns:
{
  safe: false,
  blast_radius: 42,
  breaking_files: [
    "src/services/user.service.ts",
    "src/services/auth.service.ts",
    // ... 40 more
  ],
  recommendation: "Make timeout optional with default value",
  example: "query(sql, params, timeout = 30000)"
}

// Executor adjusts approach before writing code
```

### For gsd-verifier: "Is This Actually Done?"

```typescript
// Instead of manual file reads, verifier queries:

gsd_verify_behavior({
  must_have: "Users can log in with email/password",
  check_type: "behavior"
})

// Returns:
{
  verified: true,
  evidence: [
    { file: "src/api/auth.ts", line: 45, code: "router.post('/login', ...)" },
    { file: "src/services/auth.service.ts", line: 78, code: "validateCredentials()" },
    { test: "src/__tests__/auth.test.ts", description: "should login with valid credentials" }
  ],
  confidence: "high"
}

gsd_verify_artifact({
  must_exist: "Login API endpoint at /api/auth/login",
  check_type: "artifact"
})

// Returns:
{
  exists: true,
  location: "src/api/auth.ts:45",
  method: "POST",
  middleware: ["validateBody", "rateLimiter"]
}
```

### For gsd-researcher: "How Does Our Codebase Do X?"

```typescript
// Before researching external solutions, check internal patterns:

gsd_codebase_patterns({
  query: "error handling in API routes",
  include_examples: true
})

// Returns:
{
  patterns: [
    {
      name: "Try-catch with Result type",
      frequency: "89% of API handlers",
      example: {
        file: "src/api/users.ts",
        code: "try { ... } catch (e) { return Result.err(e) }"
      }
    }
  ],
  recommendation: "Follow existing Result type pattern"
}
```

---

## MCP Tools for Phase Workflow

### Phase State Management

```typescript
gsd_phase_status()
// Returns structured phase state (not requiring STATE.md parsing)
{
  current_phase: 3,
  phase_name: "Authentication",
  plans: [
    { id: "03-01", status: "completed", tasks: 3 },
    { id: "03-02", status: "in_progress", tasks: 2, current_task: 1 },
    { id: "03-03", status: "pending", tasks: 2 }
  ],
  verification_status: "pending",
  blockers: []
}

gsd_update_phase_state({
  plan: "03-02",
  task: 2,
  status: "completed",
  commit: "abc123"
})
// Updates STATE.md atomically
```

### Requirement Traceability

```typescript
gsd_requirement_status({
  requirement_id: "AUTH-03"
})

// Returns:
{
  requirement: "Users can reset password via email",
  status: "in_progress",
  traced_to: [
    { phase: 3, plan: "03-02", task: 2 }
  ],
  verification: {
    status: "pending",
    must_haves: [
      "Reset email sends within 5 seconds",
      "Token expires after 1 hour"
    ]
  }
}

gsd_untraceable_requirements()
// Returns requirements not yet mapped to phases
```

### Cross-Phase Learning

```typescript
gsd_similar_past_work({
  current_goal: "Add Stripe payment integration",
  include_summaries: true
})

// Returns:
{
  similar_phases: [
    {
      phase: 2,
      goal: "Add SendGrid email integration",
      approach: "Service layer wrapper, webhook handler, config in env",
      lessons: "Start with webhook handler - API calls are easy, webhooks are tricky",
      files_created: ["src/services/email.service.ts", "src/api/webhooks/sendgrid.ts"]
    }
  ],
  recommendation: "Follow email integration pattern: service wrapper + webhook handler"
}
```

---

## MCP Resources for Context Loading

Instead of loading entire files, agents request specific resources:

```typescript
// Resource: gsd://project/vision
// Returns: Just the vision/core-value section of PROJECT.md

// Resource: gsd://requirements/pending
// Returns: Only incomplete requirements

// Resource: gsd://roadmap/current-phase
// Returns: Just current phase details

// Resource: gsd://intel/hotspots
// Returns: Top 10 most-depended files with purposes

// Resource: gsd://phase/{n}/context
// Returns: User's preferences for phase N

// Resource: gsd://phase/{n}/research
// Returns: Research findings for phase N
```

**Benefit:** Agents load exactly what they need. Fresh 200k context used efficiently.

---

## MCP Prompts for Agent Spawning

Pre-built prompts that combine resources intelligently:

```typescript
// Prompt: gsd://spawn/planner
// Combines: vision + current phase goal + relevant intel + conventions
// Optimized for: Creating executable plans

// Prompt: gsd://spawn/executor
// Combines: plan + relevant source files + blast radius info + conventions
// Optimized for: Executing tasks without context waste

// Prompt: gsd://spawn/verifier
// Combines: must_haves + actual codebase state + test coverage
// Optimized for: Goal-backward verification
```

---

## Revised Architecture with MCP

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         GSD WITH MCP SERVER                              │
│                                                                          │
│  ┌─────────────┐                                                        │
│  │  Commands   │                                                        │
│  │             │                                                        │
│  │ /gsd:plan   │──┐                                                     │
│  │ /gsd:exec   │  │                                                     │
│  │ /gsd:verify │  │                                                     │
│  └─────────────┘  │                                                     │
│                   │                                                     │
│                   ▼                                                     │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                         MCP SERVER                                │   │
│  │                                                                   │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │   │
│  │  │   Tools     │  │  Resources  │  │   Prompts   │              │   │
│  │  │             │  │             │  │             │              │   │
│  │  │ query_intel │  │ gsd://proj  │  │ spawn/plan  │              │   │
│  │  │ blast_radius│  │ gsd://intel │  │ spawn/exec  │              │   │
│  │  │ verify_goal │  │ gsd://phase │  │ spawn/verify│              │   │
│  │  │ phase_state │  │ gsd://req   │  │             │              │   │
│  │  │ learn_pattern│ │             │  │             │              │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘              │   │
│  │         │                │                │                       │   │
│  │         └────────────────┼────────────────┘                       │   │
│  │                          │                                        │   │
│  │                          ▼                                        │   │
│  │  ┌───────────────────────────────────────────────────────────┐   │   │
│  │  │                    .planning/ BRAIN                        │   │   │
│  │  │                                                            │   │   │
│  │  │  PROJECT │ REQUIREMENTS │ ROADMAP │ STATE │ intel/ │ phases│   │   │
│  │  └───────────────────────────────────────────────────────────┘   │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                   │                                                     │
│                   ▼                                                     │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                         AGENTS                                    │   │
│  │                                                                   │   │
│  │  gsd-planner ◀──── MCP Tools ────▶ gsd-executor                  │   │
│  │       │                                   │                       │   │
│  │       │         gsd-verifier ◀────────────┘                       │   │
│  │       │              │                                            │   │
│  │       └──────────────┼────────────────────────────────────────────│   │
│  │                      ▼                                            │   │
│  │              [Better decisions, less wasted context]              │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                          │
│  ┌─────────────┐                                                        │
│  │   Hooks     │◀─── Still capture data automatically                   │
│  │             │                                                        │
│  │ intel-index │──── MCP server reads this data                         │
│  │ intel-prune │                                                        │
│  └─────────────┘                                                        │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## What Changes with This Understanding

### Original Proposal Focus:
- Ad-hoc queries during coding
- Duplicate detection
- Convention checking
- Learning from corrections

### Revised Proposal Focus:
- **Agent-level intelligence** - Planner, executor, verifier all use MCP
- **Phase workflow integration** - State, requirements, traceability
- **Cross-phase learning** - "How did we do X before?"
- **Smart context loading** - Resources replace static file loads
- **Verification acceleration** - Structured queries instead of file reads

### New Priority Order:

| Priority | Tool | Primary Consumer | Impact |
|----------|------|------------------|--------|
| P0 | `gsd_phase_relevant_files` | gsd-planner | Better plans from day 1 |
| P0 | `gsd_pre_edit_check` | gsd-executor | Prevent breaking changes |
| P0 | `gsd_verify_behavior` | gsd-verifier | Faster, more accurate verification |
| P1 | `gsd_phase_status` | All agents | Structured state access |
| P1 | `gsd_requirement_status` | gsd-verifier | Traceability queries |
| P1 | `gsd_codebase_patterns` | gsd-planner | Learn from existing code |
| P2 | `gsd_similar_past_work` | gsd-planner | Cross-phase learning |
| P2 | `gsd_learn_correction` | All | Permanent preferences |
| P3 | Resources (gsd://) | All agents | Efficient context loading |
| P3 | Prompts | Orchestrators | Optimized agent spawning |

---

## The Deepest Insight

**GSD's power comes from orchestration.** The phase loop (discuss → plan → execute → verify) is the core value.

**MCP amplifies orchestration.** Each agent becomes smarter:
- Planner creates plans knowing what code exists
- Executor changes code knowing what will break
- Verifier confirms goals knowing what to check

**The intel system is the data layer.** MCP is the query layer. Hooks capture; MCP serves.

**Result:** Not just "Claude can ask questions" but "GSD agents make better decisions at every step of the workflow."

---

## Implementation Implications

### MCP Server Must Understand GSD Structure

The server isn't generic — it's GSD-native:

```typescript
class GsdMcpServer {
  private planningDir: string;  // .planning/
  private intelDir: string;     // .planning/intel/
  private phasesDir: string;    // .planning/phases/

  // Reads structured GSD files
  private readProjectMd(): ProjectData;
  private readRequirementsMd(): Requirement[];
  private readRoadmapMd(): Phase[];
  private readStateMd(): CurrentState;

  // Queries intel layer
  private queryGraph(query: GraphQuery): GraphResult;
  private getHotspots(limit: number): Hotspot[];
  private getRelevantFiles(keywords: string[]): RelevantFile[];

  // Phase-aware operations
  private getCurrentPhase(): PhaseDetails;
  private getPhaseContext(phaseNum: number): Context;
  private getPlanMustHaves(planId: string): MustHave[];
}
```

### Hooks Feed, MCP Serves

```
Write/Edit ──▶ gsd-intel-index.js ──▶ index.json, graph.db
                                            │
                                            ▼
                                      MCP Server reads
                                            │
                                            ▼
                              gsd_query_intel() returns structured data
```

Hooks remain reactive data capture. MCP becomes the intelligent query interface.

### Agent Integration Points

Each agent's prompt would include MCP tool availability:

```markdown
# gsd-planner agent

You have access to these MCP tools:
- `gsd_phase_relevant_files` - Find files relevant to phase goal
- `gsd_codebase_patterns` - Discover existing patterns
- `gsd_similar_past_work` - Learn from previous phases

Use these tools BEFORE creating plans to ensure accuracy.
```

---

## Conclusion: MCP is GSD's Brain Upgrade

| Without MCP | With MCP |
|-------------|----------|
| Agents load static files | Agents query dynamically |
| Plans based on generic intel | Plans based on relevant intel |
| Executor doesn't know blast radius | Executor checks before editing |
| Verifier reads files manually | Verifier queries structured data |
| Each phase starts fresh | Phases learn from each other |
| Context wasted on irrelevant data | Context used for relevant data |

**GSD + MCP = Intelligent orchestration where every agent has full codebase awareness.**
