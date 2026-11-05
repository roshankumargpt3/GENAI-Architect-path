# AIDLC Adoption Roadmap for the Existing Project

*Architecture and delivery plan | Project context: Polarion ALM, GitHub, Visual Studio with GitHub Copilot, Azure*

> **Planning placeholders:** `DATETIME-1` is the agreed program start/target date to be confirmed. `PERSON-1` is the named AIDLC coach or pilot facilitator for developer working sessions. `PERSON-2` is the project knowledge-transfer (KT) coordinator / nominated project SME. Replace these placeholders with names and dates during kickoff.
>
> **Decision principle:** AIDLC means an AI-enabled, evidence-driven SDLC—not removal of engineering accountability, required controls, or human approval. Copilot may accelerate work; people remain accountable for requirements, design decisions, acceptance, and release approval.

## 1. Executive summary

This roadmap proposes a staged transition from the current SDLC to an AI-Driven Development Lifecycle (AIDLC) for the existing project. It uses the approved project toolchain—Polarion for requirements and test management, GitHub for source and delivery, Visual Studio with GitHub Copilot for development, and Azure for cloud services—while adding repeatable AI workflows, project-specific context, verification, and governance.

The rollout is deliberately tool- and vendor-neutral beyond the existing stack. It does **not** depend on Superpowers or another unapproved prompt/skill library. Instead, the team will define and maintain its own lightweight AIDLC playbook, repository instructions, reusable templates, and quality gates. Polarion integration will start with a security and capability assessment of available MCP implementations. A third-party MCP server is not an official Polarion product unless independently confirmed; if no implementation meets the project’s security, support, and functional requirements, the project will build a small, controlled adapter or defer write operations until it is approved.

The program will begin with knowledge transfer and baseline measurement, then run a limited pilot, evaluate evidence, and scale only when security, quality, usability, and business outcomes meet agreed acceptance criteria. Expected productivity gains are hypotheses to validate, not guaranteed outcomes.

## 2. Objectives and success measures

### Objectives

- Reduce avoidable time spent locating project context, drafting routine changes, and maintaining traceability.
- Improve consistency of requirements-to-code-to-test evidence.
- Make AI-assisted development safe, reviewable, reproducible, and compatible with existing Polarion/GitHub/Azure controls.
- Free engineering capacity for modernization without reducing testing, review, security, or compliance rigor.
- Bring new team members up to speed through structured KT and discoverable project knowledge.

### Baseline and measures

During the first two weeks, record current baselines and agree targets with Engineering Management, Product, QA, Security, and the team. Compare pilot work with a representative pre-pilot sample; account for task complexity and avoid using raw code volume as a productivity proxy.

| Measure | Suggested definition | Owner / cadence |
|---|---|---|
| Lead time | Work-item start to merged PR, segmented by work type | Engineering lead / sprint |
| Cycle time by activity | Time in clarification, implementation, review, rework, and test | Team / sprint |
| First-pass quality | PRs accepted without a material rework cycle; pair with defect data | QA and engineering / sprint |
| Escaped defects | Production or downstream defects attributable to shipped work | QA / release |
| Test effectiveness | Relevant automated test coverage and failure detection, not coverage percentage alone | QA / sprint |
| Traceability completeness | Sampled work items with valid links to PRs, commits, and test evidence where required | Polarion owner / sprint |
| Developer experience | Short, voluntary survey on usefulness, friction, and cognitive load | `PERSON-1` / pilot checkpoint |
| AI operating cost | License/model, Azure, hosting, support, and integration cost per active user or work item | FinOps / month |
| Security and policy | Data exposure, policy exceptions, and security findings | Security / continuous |

Set numerical targets after the baseline. Do not claim a productivity improvement unless the data supports it and quality and security guardrails remain healthy.

## 3. Target operating model and architecture

### Tool responsibilities

- **Polarion ALM:** Authoritative source for approved requirements, work-item state, test definitions/results, and formal traceability.
- **GitHub:** Source control, issues/PRs as applicable, code review, branch protection, CI, and release evidence.
- **Visual Studio + GitHub Copilot:** Developer-facing assistance for code navigation, explanation, test drafting, and implementation within organization policy and repository instructions.
- **Azure:** Existing application hosting and, where approved, managed identity, Key Vault, monitoring, and AI/compute services. Select any model or Azure AI service only after architecture, privacy, procurement, and data-residency review.
- **AIDLC playbook:** Project-owned workflow guidance, templates, and acceptance criteria. It may include repository custom instructions and approved Copilot configuration, but it is not treated as a security boundary or as a substitute for review.
- **Polarion integration:** A narrowly scoped integration service/MCP server, if approved. MCP is a protocol, not a trust guarantee or an authorization system by itself.

### Proposed flow

1. A developer selects an approved Polarion work item and confirms its status, acceptance criteria, constraints, and out-of-scope items.
2. The developer asks Copilot to summarize relevant context and propose a plan. The developer verifies that summary against Polarion and project sources; a human approves the plan before material implementation begins.
3. The developer implements in a feature branch, using tests and existing project conventions. AI-generated code and tests are treated as suggestions and independently reviewed.
4. Local checks and GitHub CI execute deterministic tests, static analysis, dependency/security scanning, and required policy checks.
5. A human reviews the diff, test evidence, security implications, and traceability. The PR links to the Polarion item using an agreed, validated identifier format.
6. Polarion updates occur only through approved identities and workflows. Initially, the integration may be read-only; any write action (such as creating/updating test results) requires explicit authorization, auditability, validation, and human confirmation where required.
7. Existing release approval and deployment controls remain in force. AI does not merge or deploy to production autonomously.

### AIDLC working loop (project-owned, library-independent)

**Understand → Plan → Test → Implement → Verify → Review → Record.** The team will publish a short checklist and examples for each stage. The workflow should be followed proportionately to risk: small, low-risk changes need not incur heavyweight planning, while safety-, security-, privacy-, or compliance-sensitive changes require deeper review and explicit approvals. Instructions guide behavior but cannot guarantee that an AI model follows them.

## 4. Polarion MCP / integration strategy

Polarion has not been assumed to provide or officially support an MCP server. Before selecting or building one:

1. **Discover and validate options:** Identify available community/vendor implementations and their provenance, license, maintainers, release history, dependency health, supported Polarion versions/APIs, and known security issues. Confirm compatibility with the deployed Polarion environment and its API terms.
2. **Threat-model the integration:** Review data sent to the model, tool permissions, authentication, authorization, prompt-injection exposure from work-item text, input/output validation, logging, network paths, and failure modes. Treat retrieved Polarion content as untrusted input.
3. **Choose a least-privilege model:** Use a dedicated service identity only if needed. Start read-only and restrict accessible projects/work-item types. Separate read and write capabilities/identities; never expose broad administrator credentials or tokens to prompts, source code, or client logs.
4. **Select a deployment boundary:** Prefer an organization-controlled environment with approved network egress, TLS, managed secret storage (for example, Azure Key Vault where appropriate), patching, monitoring, and access controls. Validate the specific IDE/Copilot MCP support and organizational policies before deployment.
5. **Pilot safely:** Use synthetic or non-sensitive data first. Require security sign-off before production data. Log tool identity, operation, target item, outcome, and correlation ID without unnecessarily copying requirement content into logs.
6. **Build only if justified:** If no external implementation meets requirements, build a minimal adapter against supported Polarion APIs. Keep a narrow, versioned tool surface; validate inputs and authorization server-side; add timeouts, rate limits, idempotency where appropriate, audit logs, tests, and an owner/on-call plan. Avoid implementing a general-purpose proxy.
7. **Fallback:** If integration is not approved or unavailable, use a documented manual workflow in which developers open approved Polarion records and provide only permitted, necessary context. AIDLC pilot work can still proceed without MCP.

**Go/no-go gate:** No MCP server or integration may access production Polarion data until Architecture, Security, Polarion administration, and the data owner approve the implementation, permissions, hosting, retention, and operational support.

## 5. Security, privacy, reliability, and cost guardrails

### Security and privacy

- Follow enterprise Copilot, GitHub, Azure, and data-classification policies. Confirm what prompts, code, requirements, telemetry, and outputs are retained or processed by each enabled service before using confidential or regulated data.
- Classify project information and define allowed data categories for AI use. Minimize context; exclude credentials, secrets, personal data, and restricted content unless explicitly approved and protected by the relevant service configuration.
- Enforce least privilege, SSO/MFA, managed identities where supported, short-lived credentials where possible, secret rotation, environment separation, and auditable access. Never put tokens in repository files or prompts.
- Protect against prompt injection and unsafe tool execution: retrieved documents are data, not instructions; require allowlisted tools and operations, server-side authorization, validated parameters, bounded outputs, and human confirmation for consequential writes.
- Maintain branch protection, required reviews, CI checks, dependency scanning, secret scanning, and approved release controls. AI output receives the same or greater scrutiny as human-authored output.
- Establish an incident path for suspected data exposure, unsafe generated changes, compromised credentials, or incorrect ALM updates. Disable the affected integration quickly and preserve relevant audit evidence.

### Performance and reliability

- Measure end-to-end response latency, tool-call latency, timeout/error rates, rate limits, and CI duration during the pilot. Establish practical service objectives with users before scaling.
- Use bounded context and retrieve only relevant Polarion items/files. Avoid sending entire repositories or broad requirement sets to the model by default.
- Add timeouts, retries with backoff for transient failures, request limits, caching only where permitted and safe, and clear behavior when Copilot, Azure, GitHub, or Polarion is unavailable.
- Do not make a model or MCP call a hard dependency for builds, deployments, or core application runtime. Developers must be able to complete approved work using the documented manual fallback.
- Keep authoritative records and tests in systems of record; do not rely on model memory or unversioned chat history for project decisions.

### Cost management

- Establish current Copilot licensing and Azure consumption ownership; identify any additional model, hosting, networking, logging, and support charges before pilot expansion.
- Track active usage and cost by team/workflow where available, with privacy-aware aggregation. Set budgets/alerts and review consumption monthly.
- Reduce waste through focused prompts, context minimization, reusable approved templates, and limiting expensive model or tool use to tasks that benefit from it.
- Compare total cost of ownership—including engineering time for integration, patching, security reviews, and support—with measured time saved and quality outcomes. Stop or redesign features that add cost without evidence of value.

## 6. Rollout plan

The schedule below is relative to `DATETIME-1`; confirm durations and named owners at kickoff. Phases can overlap where governance permits, but gates must not be skipped.

### Phase 0 — Kickoff, discovery, and baseline (Weeks 1–2)

**Activities**
- Name the sponsor, product owner, engineering lead, QA lead, Security/Architecture contacts, Polarion administrator, GitHub administrator, FinOps contact, `PERSON-1` (AIDLC coach), and `PERSON-2` (KT coordinator/SME).
- Map current SDLC from approved requirement through implementation, test, PR, release, and Polarion evidence. Identify mandatory regulatory/customer controls and the existing definition of done.
- Capture productivity, quality, traceability, developer experience, latency, and cost baselines using the measures above.
- Complete the KT inventory in Section 7; identify knowledge gaps, critical SMEs, and single points of failure.
- Confirm Copilot entitlement, Visual Studio version/configuration, organization policy, Azure boundaries, data classification, and permitted AI usage.

**Exit criteria**
- Current-state workflow and system/data-flow diagrams reviewed by owners.
- Baseline and pilot scope approved; sensitive-data rules documented.
- KT sessions scheduled and project knowledge source locations identified.

### Phase 1 — Guardrails and workflow design (Weeks 2–4)

**Activities**
- Publish the project AIDLC playbook: intake prompt template, context checklist, lightweight plan format, test/verification checklist, PR template, traceability format, and escalation/fallback path.
- Configure repository instructions and approved Copilot settings. Keep instructions concise, version-controlled, and reviewed like code; avoid embedding secrets or relying on instructions for enforcement.
- Define risk tiers and human approval points. Preserve existing Polarion state transitions, approvals, GitHub branch protection, and deployment gates.
- Assess MCP candidates or decide on a manual/read-only first step. Complete threat model, architecture review, license review, and support ownership before any build or deployment.
- Add or tune deterministic CI checks where gaps exist (tests, linting, static analysis, dependency/secret scanning, traceability validation). Do not use AI output as the sole compliance control.

**Exit criteria**
- Security, architecture, product, QA, and ALM owners approve pilot controls.
- Playbook and repository configuration pass a small developer usability review.
- MCP decision is recorded: approved candidate, controlled build proposal, or manual fallback.

### Phase 2 — Controlled pilot and developer coaching (Weeks 4–8)

**Pilot scope**
- Start with a representative but bounded team and low-to-medium-risk work items. Exclude high-impact production changes and restricted data until separately approved.
- Each participating developer completes onboarding and at least one AIDLC working session with `PERSON-1` (or a delegated coach). Hold a recurring weekly clinic/office hour during the pilot; reserve time for individual developer pairing based on team capacity.
- `PERSON-2` coordinates existing-project KT sessions (Section 7) and makes recordings/notes available only in approved repositories with the right access controls.
- Use read-only Polarion retrieval only if integration has passed the security gate. Otherwise follow the manual context checklist.
- Require human plan acceptance, code review, test evidence, and valid Polarion traceability. Test any Polarion write operation only in a non-production/test environment with explicit approval.

**Pilot evaluation**
- Review measures each sprint, compare with baseline and a suitable non-pilot sample, and gather developer/QA feedback.
- Record false starts, inaccurate summaries, rejected suggestions, rework, model/tool latency, policy exceptions, costs, and operational issues.
- Adjust playbook and templates from evidence; do not tune for higher AI usage or code volume alone.

**Exit criteria**
- No unresolved critical security/compliance issue; required controls remain effective.
- Pilot demonstrates acceptable usability and quality, with outcome evidence assessed against agreed targets.
- Owners approve scale-up, extend pilot, redesign, or stop decision.

### Phase 3 — Expand and operationalize (Weeks 8–12 and ongoing)

**Activities**
- Expand incrementally by team and work type, with onboarding and coaching capacity planned for every developer.
- Harden approved Polarion integration for supportability: monitoring, alerting, patching, ownership, incident response, access recertification, API-version compatibility, and documented fallback.
- Add automated traceability validation in GitHub CI where reliable identifiers and Polarion availability permit. Prefer deterministic validation and explicit exceptions over an AI judgment gate.
- Review policy, costs, outcomes, and knowledge freshness monthly; reassess after major Copilot, Visual Studio, Polarion, or Azure changes.
- Use freed capacity for prioritized modernization work selected by product/engineering leadership, and measure that work independently.

**Scale criteria**
- Quality and security guardrails are not worse than baseline; any change is understood and accepted by accountable owners.
- Evidence supports a useful improvement in at least one agreed productivity outcome without unacceptable cost or developer burden.
- Training, support, KT maintenance, and integration ownership can sustain the expanded cohort.

## 7. Existing-project knowledge transfer (KT)

Because the architect is new to the project, KT is a prerequisite—not an optional add-on. `PERSON-2` coordinates sessions with the relevant owners and maintains a project onboarding index linking to authoritative, access-controlled sources. The sessions below should be recorded as concise notes, diagrams, decisions, links, owners, and open questions; avoid creating duplicate or conflicting specifications.

| KT session | Required topics | Suggested presenters | Output |
|---|---|---|---|
| Product and business context | Users, business outcomes, product boundaries, priorities, domain terminology, success measures | Product owner / business SME | Context map, glossary, key journeys |
| Polarion and requirements | Project structure, work-item types, states, approval rules, baselines, queries, test management, traceability conventions, API/version constraints | Polarion admin / BA / QA | Requirement-to-release workflow, access and data-classification notes |
| Architecture and integrations | Context/container diagrams, major components, interfaces, data flows, identity, external dependencies, architecture decisions, known constraints | Solution architect / technical leads | Current architecture diagrams, ADR links, dependency map |
| Codebase and developer workflow | Repository structure, branching, coding standards, build/run/debug, local environment, test suites, CI, release process | Senior developers / DevOps | Setup guide, build/test commands, contribution checklist |
| Quality, test, and operations | Test strategy, environments, test data, defect patterns, observability, SLOs, incident history, rollback and support | QA lead / SRE / operations | Test/operational runbooks, quality risks, escalation routes |
| Security, privacy, and compliance | Data classification, threat model, access controls, vulnerability process, audit evidence, retention, required approvals | Security / compliance / data owner | Approved AI-use boundaries and security requirements |
| Azure and delivery platform | Subscriptions/resource groups, network boundaries, identity, Key Vault, monitoring, cost allocation, deployment and disaster recovery | Azure platform owner | Environment map, deployment/DR and cost ownership notes |
| Roadmap and modernization | Current commitments, technical debt, modernization candidates, dependencies, constraints, decision process | Engineering manager / product owner | Prioritized opportunity list and dependency/risk log |

**KT cadence and completion**
- In Weeks 1–2, schedule the core sessions and identify which artifacts already exist; do not repeat sessions unnecessarily where authoritative materials are current.
- Run a weekly 30–60 minute question clinic during the pilot, coordinated by `PERSON-2`; use asynchronous notes for questions that do not require a meeting.
- Ask each new participant to complete a small, low-risk change using the documented setup and workflow, then capture gaps in the onboarding guide.
- Track completion by role and topic, not attendance alone. Confirm the developer can find, explain, and apply key information.
- Review KT material quarterly or after material architecture/process changes. Assign owners and review dates to diagrams and runbooks.

## 8. Roles and decision rights

| Role | Accountability |
|---|---|
| Engineering sponsor/manager | Business outcomes, capacity, risk acceptance within authority, scale/stop decisions |
| Product owner | Requirement quality and priority; confirms intended acceptance behavior |
| `PERSON-1` AIDLC coach | Developer enablement, working sessions, playbook feedback, adoption friction |
| `PERSON-2` KT coordinator | Existing-project KT plan, SME coordination, onboarding index and follow-up |
| Developers | Verify context, make design choices, review AI suggestions, test changes, maintain traceability |
| QA lead | Test strategy and evidence quality; approves test-process changes |
| Architecture/Security | Architecture and threat-model review, data/tool boundaries, integration approval |
| Polarion administrator/data owner | ALM permissions, workflow and API constraints, record correctness |
| GitHub/Azure platform owners | Platform policies, CI/CD, identity, monitoring, operational support |
| FinOps | Cost baseline, budgets, alerts, value/cost review |

## 9. Key risks and mitigations

| Risk | Mitigation |
|---|---|
| Unapproved or vulnerable community MCP implementation | Provenance, license, dependency, and security review; controlled deployment; read-only pilot; build a minimal adapter or use manual fallback |
| Sensitive requirements or source code exposed to an unapproved service | Data classification, enterprise policy validation, context minimization, access restrictions, approved service configuration, audit and incident process |
| AI produces incorrect code, tests, or requirement interpretation | Verify against authoritative sources; human plan/code review; deterministic tests and CI; risk-based review; no autonomous production release |
| Traceability automation writes incorrect ALM records | Start read-only; validate IDs and states server-side; use test environment; narrow write scope, audit, idempotency, and human confirmation |
| Prompt injection through work items, issues, or repository content | Treat retrieved content as untrusted data; do not follow embedded tool instructions; allowlist tools; constrain permissions and outputs |
| Productivity measured incorrectly or gains do not materialize | Establish baseline, compare representative work, include rework/quality/cost, and use stop/adjust gates |
| Tool outages, latency, or quotas interrupt development | Manual fallback, timeouts, rate limits, monitoring, and no dependency in build/runtime paths |
| Knowledge concentrated in a few SMEs | KT index, recorded decisions, cross-training, named backups, periodic freshness reviews |
| AI use adds review burden or erodes skills | Coach on verification, monitor developer feedback and review effort, retain fundamentals and ownership, adjust workflow |

## 10. Decisions required at kickoff

1. Confirm `DATETIME-1`, sponsor, pilot team, work-item scope, and delivery calendar.
2. Assign actual owners for `PERSON-1` and `PERSON-2`, and agree their capacity for coaching and KT coordination.
3. Approve the project-specific AI data classification and confirm Copilot/Azure service policies for the relevant information types.
4. Confirm whether an MCP integration is in scope for the first pilot; authorize assessment only until the security gate is met.
5. Agree baseline metrics, initial targets, review cadence, and who can approve scale/stop decisions.
6. Confirm existing release, Polarion workflow, electronic signature, and audit controls that must remain unchanged.
7. Prioritize modernization work that may use validated capacity released by the pilot.

## 11. Recommended initial deliverables

- Current-state AIDLC/SDLC workflow and system/data-flow diagrams.
- Project AI-use and data-classification guidance approved by Security and data owners.
- AIDLC playbook, prompt/context checklist, plan and PR templates, and human approval matrix.
- KT schedule, onboarding index, architecture/dependency map, and open-question log.
- Polarion MCP assessment and threat model, including a documented manual fallback.
- Baseline dashboard and pilot scorecard covering quality, productivity, developer experience, traceability, security, latency, and cost.
