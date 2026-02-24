# Claude.md — Salesforce Development Agent Instructions

## Workflow Orchestration

### 1. Plan Node Default
- Enter plan mode for ANY non-trivial task (3+ steps or architectural decisions)
- If something goes sideways, STOP and re-plan immediately — don't keep pushing
- Use plan mode for verification steps, not just building
- Write detailed specs upfront: include object model, data flow, and governor limit impact
- Always consider **sandbox → scratch org → production** deployment path before writing code

### 2. Subagent Strategy
- Use subagents liberally to keep main context window clean
- Offload metadata discovery, dependency analysis, and org queries to subagents
- For complex problems (e.g. large data migrations, multi-object triggers), throw more compute via subagents
- One concern per subagent: e.g. separate agents for Apex logic, LWC, and deployment validation

### 3. Self-Improvement Loop
- After ANY correction from the user: update `tasks/lessons.md` with the pattern
- Write rules that prevent repeating the same Salesforce-specific mistake (e.g. SOQL in loops, missing null checks)
- Ruthlessly iterate on lessons until mistake rate drops
- Review lessons at session start for the relevant Salesforce project

### 4. Verification Before Done
- Never mark a task complete without proving it works in org or via tests
- Always run: `sf apex run test` or check test results in the org before closing a task
- Ask yourself: "Would a Salesforce Architect approve this design?"
- Validate against: governor limits, FLS/CRUD, sharing model, and bulk safety

### 5. Demand Elegance (Balanced)
- For non-trivial changes: pause and ask "is there a more elegant, platform-native way?"
- Prefer declarative (Flow, Validation Rules, Formula Fields) over Apex when equally powerful
- If code feels hacky, refactor: "Knowing everything I know now, implement the clean solution"
- Skip over-engineering for simple config changes — don't write Apex for what a Flow can do
- Challenge your own work before presenting it

### 6. Autonomous Bug Fixing
- When given a bug report: just fix it. Don't ask for hand-holding
- Check debug logs, exception emails, test failures — then resolve
- Zero context switching required from the user
- Proactively fix failing tests, deployment errors, and PMD violations without being told how

---

## Salesforce Development Standards

### Apex Best Practices
- **Bulkify everything**: all triggers and classes must handle 200+ records
- **No SOQL/DML inside loops** — ever. Query before, process in memory, DML after
- **One trigger per object** — use a handler/dispatcher pattern (e.g. TriggerHandler framework)
- **Use `with sharing`** by default; explicitly justify `without sharing` or `inherited sharing`
- **Always handle null**: check for null before accessing related object fields
- **Async when needed**: use `@future`, Queueable, or Batch for long-running operations
- **Avoid hardcoded IDs**: use Custom Labels, Custom Metadata, or Custom Settings
- **Write test classes to 85%+ coverage** with meaningful assertions, not just coverage padding
- Use `Test.startTest()` / `Test.stopTest()` to reset governor limits in tests
- Mock external callouts with `HttpCalloutMock` — never make real callouts in tests

### SOQL & Data Access
- Always use selective queries — include indexed fields in WHERE clauses
- Use `LIMIT` and pagination (`OFFSET` or keyset) for large datasets
- Prefer `FOR UPDATE` only when necessary to avoid lock contention
- Use `WITH SECURITY_ENFORCED` or `stripInaccessible()` for FLS compliance
- Never use `SELECT *` — always specify fields explicitly

### Lightning Web Components (LWC)
- Follow the **container/presentational component** pattern
- Use `@wire` for read-only Salesforce data; use imperative Apex for conditional or mutating calls
- Handle loading and error states explicitly in every component
- Use `lightning-record-form` or `lightning-record-edit-form` for standard CRUD when possible
- Keep business logic in Apex — LWC handles UI only
- Always define `@api` properties with proper types and validation

### Flows & Declarative Automation
- Prefer Flow over Apex for straight-line automation without complex branching
- Use **Record-Triggered Flows** over Workflow Rules (deprecated path)
- Avoid too many flows on the same object/trigger point — consolidate when possible
- Test flows with Flow Test or manually in sandbox before deploying
- Document flow description and each decision node's purpose

### Metadata & Deployment
- Use **Salesforce CLI (sf)** for all deployments: `sf project deploy start`
- Always deploy to a **sandbox or scratch org first** — never directly to production
- Use **scratch orgs** for isolated feature development with `project-scratch-def.json`
- Track all metadata in version control (Git) — no org-only changes
- Use `.forceignore` to exclude non-deployable metadata
- Validate destructive changes (`destructiveChangesPre.xml`) before executing
- Use **Change Sets** only as a last resort — prefer CLI or CI/CD pipelines

### Security & Compliance
- Enforce **FLS (Field Level Security)** and **CRUD** checks in all Apex
- Never expose sensitive fields via public APIs or LWC without proper access checks
- Use **Named Credentials** for all external service authentication — no hardcoded credentials
- Apply **IP restrictions** and **session policies** in connected apps
- Run **Salesforce Security Health Check** regularly and resolve critical findings

### Governor Limits Awareness
Always design with these limits in mind:
- SOQL queries: 100 per transaction
- DML statements: 150 per transaction
- Heap size: 6 MB (sync) / 12 MB (async)
- CPU time: 10,000 ms (sync) / 60,000 ms (async)
- Callouts: 100 per transaction, 10s timeout default
- If approaching limits, move work to Batch Apex or Queueable chains

---

## Task Management

1. **Plan First**: Write plan to `tasks/todo.md` — include affected objects, metadata types, and limit impact
2. **Verify Plan**: Check in before starting implementation; confirm sandbox availability
3. **Track Progress**: Mark items complete as you go; note deployment status per environment
4. **Explain Changes**: High-level summary at each step including *why* the Salesforce approach was chosen
5. **Document Results**: Add review section to `tasks/todo.md` with test coverage % and deployment outcome
6. **Capture Lessons**: Update `tasks/lessons.md` after corrections, especially for governor limit issues

---

## Core Principles

- **Simplicity First**: Use the most platform-native solution. Declarative before programmatic.
- **No Laziness**: Find root causes. No temp fixes. Think like a Salesforce Architect.
- **Minimal Impact**: Touch only what's necessary. Avoid side effects on unrelated objects or automations.
- **Bulk Safe Always**: Every solution must work on 1 record and 10,000 records equally.
- **Security by Default**: FLS, CRUD, sharing — enforce them without being asked.
- **Test Everything**: No untested code ships. Meaningful assertions only — no coverage padding.