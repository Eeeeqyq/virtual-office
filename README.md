# Virtual Office

A private website for submitting requests from a phone or computer and coordinating Claude Code, Muse, and OpenAI/Codex agents around shared project context. The intended workflow is to discuss approaches, divide work, execute tasks, review each other's results, and display progress and deliverables in one place.

## Project status and scope

This repository currently contains project planning documentation only. The architecture and integrations below are proposals, not implemented or verified capabilities. This initial step does not deploy services, purchase products, or enable unattended agent execution.

## Project goals

- Provide one private, mobile-friendly place to submit and track requests.
- Maintain shared project background, requirements, decisions, and artifacts across agents.
- Support explicit planning, task assignment, execution, and cross-agent review.
- Show task status, discussion history, review findings, and final deliverables.
- Accept requests even when the local computer is offline; preserve them until an eligible worker reconnects.

## MVP features

1. **Private access:** owner authentication and access control for the website and stored artifacts.
2. **Request inbox:** submit a request with project context, attachments, and acceptance criteria from desktop or mobile.
3. **Shared project context:** versioned project briefs, decisions, task history, and links to relevant files.
4. **Planning and discussion:** record proposed approaches, agent responses, task assignments, and dependencies. Start with manual participation where an integration is unavailable.
5. **Durable task queue:** persist requests and track queued, running, awaiting input, in review, completed, failed, and cancelled states.
6. **Local worker:** after permissions are verified, connect outbound to the backend, claim eligible tasks, run approved Claude Code workflows, and upload progress and results.
7. **Peer review:** let an available second agent review outputs against acceptance criteria, recording issues and revisions. Do not report a review as completed unless it actually ran.
8. **Progress and results:** display an activity timeline, worker availability, execution logs, review outcomes, and downloadable artifacts.
9. **Human control:** support clarification requests, approval checkpoints for consequential actions, and cancellation.

## Preliminary architecture

```text
Phone / desktop browser
          | HTTPS
          v
Cloud web frontend + backend API
          |
          +-- Database: projects, requests, context, discussions, task state
          +-- File storage: attachments, logs, and deliverables
          +-- Persistent task queue: pending work, leases, and retries
          ^
          | Outbound authenticated connection initiated by local worker
          | Task claims, heartbeats, progress, and result uploads
Local computer
          +-- Agent worker
                  +-- Claude Code adapter (permissions and execution method TBD)

Muse and OpenAI/Codex adapters: integration methods and permissions TBD
Future cloud workers: optional, subject to permissions and cost review
```

### Execution flow

1. The backend saves a request and its context, then durably queues the work.
2. Planning produces explicit tasks, dependencies, and acceptance criteria. Agent participation depends on verified integrations; manual handoffs are an initial fallback.
3. The local worker initiates an authenticated outbound connection and claims compatible tasks. No inbound connection to the local computer is required by this design.
4. The worker runs only authorized workflows and sends heartbeats, logs, and artifacts to the backend.
5. Completed work moves to review; findings and revisions are recorded before final completion.
6. The website presents the outcome and links to deliverables.

### Offline and recovery behavior

- The cloud website remains able to accept and store requests while the local computer is offline.
- Tasks requiring the local worker stay queued until it reconnects; the website should clearly show that they are waiting for a worker.
- Worker leases and heartbeats detect interrupted runs. Retries must avoid duplicate external actions and preserve execution history.
- Use idempotent task handling, bounded retries, and explicit reconciliation for partially completed work.
- Store secrets outside project context and logs; scope worker access to the projects and actions it needs.

### Technology choices

Hosting, frontend/backend frameworks, database, file storage, queue provider, and agent SDKs have not been selected. Prefer a small maintainable stack with durable storage and clear operating costs. Cloud workers are a later option, not an MVP dependency.

## Implementation phases

### Phase 0 - Validate access and define boundaries

- Verify the Northeastern University-provided Claude Code entitlement, API access, MCP availability, and unattended execution permissions separately.
- Identify the exact Muse product and its supported integration methods.
- Confirm the intended OpenAI/Codex interface, authentication, permissions, and budget.
- Define allowed actions, human approval requirements, data handling, and initial acceptance criteria.
- Record available capabilities and blockers before implementing automated adapters.

### Phase 1 - Private request portal

- Implement authentication, project context, request submission, database persistence, and file storage.
- Add a durable queue and task status UI; validate that requests survive an offline local computer.
- Allow manual task execution and result entry while integrations are being verified.

### Phase 2 - Local execution pilot

- Implement an outbound local worker with authentication, task claims, heartbeats, cancellation, and result uploads.
- Add Claude Code execution only through an approved, verified interface.
- Test disconnects, restarts, retries, partial failures, and human approval checkpoints.

### Phase 3 - Multi-agent collaboration

- Add supported Muse and OpenAI/Codex adapters after their access requirements are confirmed.
- Implement shared context snapshots, planning discussions, task dependencies, and cross-agent review.
- Make review findings, revisions, and final acceptance visible in the website.

### Phase 4 - Reliability and optional cloud workers

- Improve monitoring, audit history, backups, retention, and cost controls.
- Evaluate cloud workers only after permissions, security requirements, and operating costs are understood.

## Open questions and unverified assumptions

- **Claude Code / Northeastern University:** Claude Code is provided by Northeastern University. API access, MCP availability, and permission for unattended or automated execution are not yet verified. The existing entitlement must not be assumed to include separate API credentials, API billing, or authorization for server-side or unattended use. Confirm applicable university and provider requirements before integration.
- **Muse:** Which product and account are intended? Are API, SDK, MCP, CLI, webhook, or manual integration methods supported and permitted? Its integration method remains unconfirmed.
- **OpenAI/Codex:** Which interface will be used, what account permissions are available, and what authentication and billing model applies? No integration is configured yet.
- **Hosting and budget:** Which region and providers should host the site, database, files, and queue? What are the monthly spending and per-task limits?
- **Data and privacy:** What project data may be shared with each agent or provider? What retention, deletion, backup, and university data-handling requirements apply?
- **Local runtime:** Which directories and repositories may the worker access? How should startup, sleep, network loss, and concurrent tasks be handled?
- **Coordination:** Which agent plans, executes, and reviews each task? How are disagreements resolved and context changes versioned?
- **Approvals and completion:** Which actions require owner approval, and who accepts final results?
- **Cloud workers:** Are the relevant licenses and interfaces compatible with cloud execution, and would the reliability benefit justify the cost?

## Initial success criteria

The first working version should let the owner submit a request from a phone, see it persist while the computer is offline, reconnect an authorized local worker, follow execution progress, inspect an actual review or a clearly marked pending review, and retrieve the final artifacts. Integrations that are not available must remain explicitly marked as unverified or manual.
