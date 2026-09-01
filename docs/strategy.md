# Cyberwave Agent Skills Strategy

## Executive decision

Keep exactly one discoverable, cross-agent skill named `cyberwave`. Make its `SKILL.md` a concise orchestrator that establishes context, selects the safest execution plane, and routes to focused reference modules. CLI/edge operations and driver development are references and deterministic scripts of this skill, not separate skills. Multiple top-level skills would increase discovery noise, duplicate shared safety rules, and make installation inconsistent across clients.

The Cyberwave monorepo directory `cyberwave-clis/cyberwave-skills/` is the maintained source of truth. Its one skill is published through the existing `cyberwave-os/cyberwave-skill` repository; the Claude plugin repository is a generated compatibility distribution. Changes to MCP tools, SDK or CLI interfaces, user documentation, edge behavior, authentication, or robot-control semantics must include a skill impact review and, when relevant, a skill update before distribution sync runs.

## Outcomes

The skill system must let an agent reliably:

1. Register or sign in a user and establish workspace, project, environment, and twin context.
2. Create and edit environments with catalog assets, procedural geometry, transforms, areas, waypoints, and visual validation.
3. Create, edit, inspect, run, and monitor workflows.
4. Control robots in simulation and, only with explicit intent and appropriate safeguards, in live mode.
5. Register and configure edge deployments and diagnose their health.
6. Onboard new assets and develop drivers without conflating those two workflows.
7. Monitor robots through telemetry, schemas, frames or streams, workflow runs, alerts, and edge health.
8. Prefer the Cyberwave MCP server for typed, contextual operations while falling back cleanly to the SDK, CLI, dashboard, or official docs when a capability is unavailable.
9. Work from the same core package in Claude Code, Codex, ChatGPT/API skill runtimes, and other Agent Skills-compatible clients.

## Non-goals

- Do not embed a complete copy of Cyberwave API, SDK, CLI, or MCP documentation in the skill.
- Do not teach agents to bypass permissions, approvals, workspace ACLs, or hardware safety controls.
- Do not treat a missing tool as authorization to improvise an undocumented endpoint or MQTT topic.
- Do not make the public mirror an independently edited second source of truth after rollout.
- Do not bundle credentials, machine-specific identifiers, or live environment state.

## Architecture

```text
User request
    |
    v
cyberwave/SKILL.md                         one discoverable entrypoint
    |
    +-- establish auth and resource context
    +-- select MCP -> SDK/CLI -> dashboard/docs fallback
    +-- classify intent and mutation risk
    |
    +-- references/registration-and-auth.md
    +-- references/mcp-and-fallbacks.md
    +-- references/environment-management.md
    +-- references/workflow-authoring.md
    +-- references/robot-control.md
    +-- references/edge-configuration.md
    +-- references/cli-command-map.md
    +-- references/asset-and-driver-development.md
    +-- references/driver-development.md
    `-- references/robot-monitoring.md

    +-- scripts/scaffold_driver.py
    `-- assets/driver-template/
```

This follows the Agent Skills progressive-disclosure model: metadata is always visible, the short orchestrator loads only when relevant, and detailed domain guidance loads only for the current workflow. The explicit execution-plane matrix distinguishes MCP tasks from CLI/SDK/dashboard work. Provider-specific metadata may be added under `agents/`, but the operational guidance stays provider-neutral.

## Orchestrator contract

The entrypoint performs the following decisions in order:

1. **Preserve the user's objective.** Do not force a setup interview when the request or available context already identifies the target and desired outcome.
2. **Discover capabilities.** Inspect available Cyberwave MCP tools and current resource context. Never assume every client exposes the same tool set or that a tool name mentioned in a reference is callable.
3. **Establish identity and scope.** Resolve account access, workspace, project, environment, and twin only to the depth the task needs. Handle registration or authentication before protected operations.
4. **Classify the operation.** Distinguish read, reversible configuration, destructive mutation, simulated actuation, and live actuation.
5. **Load the narrow reference.** Usually one domain reference; add registration/auth or MCP fallback guidance only when necessary.
6. **Plan before mutation.** Use `execute=false`, preview, plan, or dry-run modes when a Cyberwave tool supports them. Show the resolved targets and material effects before high-impact actions.
7. **Execute within authorization.** A broad request to build or configure authorizes normal scoped mutations. Deletion, credential changes, external registration acceptance, and live physical motion retain their own approval or explicit-intent boundaries.
8. **Verify the outcome.** Re-read state, render the environment, inspect a workflow run, capture a frame, or check edge health as appropriate.
9. **Report concrete identifiers.** Return created or changed resource names, UUIDs/slugs when useful, mode (`simulation` or `live`), and verification status without exposing secrets.

## Execution-plane policy

| Plane | Use when | Rules |
| --- | --- | --- |
| Cyberwave MCP | Typed tools are available for the operation | Preferred. Reuse session context. Inspect tool schemas. Use preview/plan flags and structured errors. |
| Python SDK | The user is implementing an application or MCP lacks the needed operation | Verify current SDK surface from installed code or official docs. Use `CYBERWAVE_API_KEY`; never invent methods. |
| Cyberwave CLI | Edge/bootstrap or an explicitly CLI-oriented workflow is supported | Inspect `--help` or current docs before generating commands. Do not claim commands that are not present. |
| Dashboard | Registration, OAuth, billing, visual/manual setup, or another human-only step is required | Give the shortest handoff and resume from the resulting identifiers. |
| REST/MQTT/Zenoh | The user explicitly needs a lower-level integration or a maintained reference requires it | Verify current schemas and transport ownership. MQTT is the remote/cross-LAN path; Zenoh is for edge-colocated data bus traffic when declared. |
| Official docs search | Behavior or interface is uncertain or likely changed | Prefer Cyberwave MCP documentation search when present, then repository docs or official web docs. |

## Domain modules

### Registration and authentication

Cover account creation, API-key creation, secure local configuration, MCP connection, and resource-context discovery. Registration is an external account mutation: the agent may guide or use a dedicated registration flow, but must not accept legal terms, choose an organization, or expose credentials without user involvement.

### Environment creation and editing

Resolve or create the workspace/project/environment, plan the scene, prefer curated primitives before broad catalog search, add twins, edit transforms and properties, create areas/waypoints/procedural geometry, and verify with environment context plus rendered or analytical layout feedback. Destructive edits require an exact resolved object and explicit execution.

### Workflow authoring

Prefer prompt-based workflow creation/editing and template cloning when exposed by MCP. Inspect node schemas before proposing node-specific configuration. Keep `simulation` as the authoring default, preview mutations, validate the saved graph, trigger only when requested, and monitor the resulting run.

### Robot control

Resolve the twin and capabilities before selecting a control surface. Plan/resolve the route before dispatch. Simulation is the default. Live control requires explicit live intent in the current request, a single resolved physical target, compatible capability/driver state, and a bounded action. Never infer continuous live control, remove limits, or retry unsafe commands automatically. Prefer stop actions when state is uncertain.

### Edge configuration and CLI

Separate control-plane registration from host configuration and from driver deployment. MCP can prepare or verify cloud resources but cannot install packages, access local devices, or manage systemd/launchd and Docker on the target host. Verify installed CLI commands and the current edge runtime before giving commands. Keep secrets out of files and output, account for MQTT versus Zenoh transport declarations, validate driver manifests, and finish with health/connectivity checks.

### Asset onboarding and driver development

Treat an asset as the catalog/digital representation and a driver as the hardware bridge. Reuse an existing catalog asset and driver where possible. For new assets, validate files, metadata, universal-schema capabilities, and visualization. For drivers, load `references/driver-development.md` and use the embedded deterministic scaffold/template; test locally before registry or live deployment. After URDF upload, define capabilities immediately.

### Robot monitoring

Select the least expensive signal that answers the question: resource/schema state, joint/telemetry state, one frame, a short frame burst, stream, workflow runs, alerts, or edge logs/health. Bound polling and frame capture, avoid recording or storing video without explicit scope, and distinguish stale/offline data from healthy zero-valued readings.

## Safety and authorization model

| Class | Examples | Default behavior |
| --- | --- | --- |
| Read-only | List environments, inspect schemas, capture one requested frame | Execute and report. |
| Previewable mutation | Scene edits, workflow creation/update, control plan | Preview first when supported; execute when the user's request authorizes the change. |
| Destructive mutation | Delete asset/environment object/workflow/model | Resolve exact target, state impact, require explicit authorization, then verify. |
| Simulated actuation | Set joints, navigate, motion pose in simulation | Allowed when requested; use bounded actions and verify state. |
| Live actuation | Move a physical robot, dispatch live control | Require explicit live intent, resolved target, capability/health checks, bounded command, and a stop/recovery path. |
| Credential/legal/account | Register, create/revoke keys, accept terms | Keep user in the loop for identity, terms, organization choice, and secret handling. |

Skills never grant tool permissions. Client and platform approval systems remain authoritative.

## Cross-agent compatibility

### Portable core

- Conform to the open Agent Skills directory format with `SKILL.md`, `references/`, optional deterministic `scripts/`, and optional `agents/` metadata.
- Keep required frontmatter portable: `name` and a discriminating `description`.
- Avoid provider-only dynamic prompt injection and provider-only permission fields in the portable entrypoint.
- Resolve all relative resources from the skill directory.

### Claude Code

- Support the package installer linking/copying `skills/cyberwave` to `~/.claude/skills/cyberwave` or `.claude/skills/cyberwave`.
- Keep `/cyberwave` as the single direct invocation.
- Do not rely on Claude-only frontmatter for correctness.

### Codex

- Support the package installer linking/copying `skills/cyberwave` to `~/.codex/skills/cyberwave` or a project skill directory recognized by the client.
- Add `agents/openai.yaml` only for discoverability/UI metadata; keep behavior in `SKILL.md`.
- Validate with the Codex skill validator.

### ChatGPT and API runtimes

- Package only `skills/cyberwave` as a skill bundle when the runtime accepts uploaded skills.
- Treat remote MCP configuration and credentials as deployment configuration, not skill content.
- Version uploaded skill bundles immutably and promote a tested default version.

### Other clients

- Document the generic installation requirement: place the `cyberwave` directory beneath the client's Agent Skills search path.
- Degrade gracefully when the client cannot load references or use MCP: direct the agent to SDK/CLI/dashboard guidance rather than assuming tools.

## Monorepo ownership and drift prevention

### Source of truth

- Maintained source: `cyberwave-clis/cyberwave-skills/`
- Primary public distribution: `github.com/cyberwave-os/cyberwave-skill`
- Compatibility distribution: `github.com/cyberwave-os/cyberwave-plugin`
- Sync workflow: `.github/workflows/claude-plugin-sync.yml`

The production workflow uses `rsync --delete`, so public-mirror-only edits will eventually be erased. Development can use an isolated public-repository worktree for review, but accepted changes must land in the monorepo source and be validated before sync.

### Change-impact matrix

| Monorepo area | Required skill review |
| --- | --- |
| `cyberwave-clis/cyberwave-mcp-server/**` | Tool names, parameters, preview/execute semantics, error codes, context behavior, MCP setup. |
| `cyberwave-sdks/**` | Installation, auth environment variables, method names, simulation/live semantics, transports. |
| `cyberwave-clis/cyberwave-python-cli/**` | Bootstrap, login, edge registration/configuration, diagnostics, command examples. |
| `cyberwave-edge-core/**`, `cyberwave-edge-runtime/**`, `cyberwave-edge-nodes/**` | Driver manifests, MQTT/Zenoh routing, deployment and health checks. |
| `cyberwave-backend/src/app/api/**`, schemas, ACL/auth | Resource lifecycle, permissions, registration/auth, workflow and control behavior. |
| `docs-mintlify/**` | User-facing names, links, examples, compatibility claims. |
| Driver scaffold/template | Asset/driver handoff, `BaseDriver`, manifest and transport conventions, build/deployment checks. |

### CI contract

1. Assert that the package exposes exactly one `SKILL.md` entry and validate it with a spec-compatible validator.
2. Check local Markdown links and required files.
3. Maintain a machine-readable list of MCP tool names used by the skill and compare it with server registration, allowing explicitly documented optional tools.
4. Run scenario-level static checks for safety invariants: simulation default, explicit live intent, no embedded credentials, and no undocumented raw endpoint/topic recommendation.
5. Generate a sample driver, reject unresolved placeholders, and compile generated Python.
6. Expand the sync workflow path triggers so relevant MCP, SDK, CLI, edge, docs, and backend interface changes cause skill validation or a required impact-review result, rather than silently shipping drift.

## Validation matrix

The implementation is release-ready only after these realistic scenarios pass:

1. New user without an account asks to create a robot environment.
2. Authenticated user asks to add a common camera and position it.
3. User asks to build a pick-and-place workflow from natural language.
4. User asks to move an arm, without specifying simulation or live.
5. User explicitly asks for a bounded live motion on a named robot.
6. User asks to configure an edge host and install a driver.
7. User provides a new URDF asset and asks for onboarding.
8. User asks to monitor a robot's camera and joint state.
9. Cyberwave MCP is absent.
10. Cyberwave MCP exposes only a subset or renamed version of the documented tools.
11. A destructive request targets an ambiguous object.
12. A public compatibility mirror differs from the monorepo-owned source.
13. A user needs CLI installation or edge pairing and MCP is available but cannot perform the host mutation.
14. A user needs a new driver and the agent must use the same skill's scaffold rather than discover a second skill.

Tests should assert observable decisions and safety properties, not exact prose.

## Implementation sequence

1. Replace the 490-line monolithic entrypoint with the concise orchestrator.
2. Add focused references, including the MCP-versus-CLI/SDK/dashboard matrix, CLI command map, and driver development guide.
3. Move the driver scaffold and current `BaseDriver` template under the one skill's scripts/assets.
4. Add cross-agent metadata and a single-skill installer.
5. Add deterministic MCP, CLI, scaffold, and packaging validation.
6. Move the canonical package into an isolated monorepo worktree, update CI/change triggers, and rerun validation.
7. Review the diff, manually synchronize the initial large change to the existing public skill repository, and record both PRs.

## Rollout and versioning

- Land the monorepo source change first.
- Manually synchronize the initial structural change to `cyberwave-os/cyberwave-skill`, then use the production workflow for subsequent updates. Do not make the public repository the authoring source.
- Keep the Claude plugin compatibility distribution during migration; deprecate the plural and standalone driver repositories because their content now lives in the singular `cyberwave` skill.
- Use semantic metadata or release tags for skill bundles when the target client supports immutable versions.
- Roll back by reverting the monorepo source and allowing the sync workflow to restore the public mirror.
- Treat skill behavior changes that alter live-control authorization, credential handling, or destructive-action policy as high-risk changes requiring explicit review.
