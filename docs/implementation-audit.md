# Cyberwave skill implementation audit

This audit maps the requested outcome to authoritative repository evidence. It is a release checklist, not a substitute for automated validation.

## Product requirements

| Requirement | Evidence | Status |
| --- | --- | --- |
| One agent entrypoint | `skills/cyberwave/SKILL.md`; package validator rejects any second skill directory | Implemented |
| Environment creation and editing | `references/environment-management.md`; current MCP tools checked against server source | Implemented |
| Workflow authoring and runs | `references/workflow-authoring.md`; schema/template/prompt/run routes | Implemented |
| Robot control in simulation and live | `references/robot-control.md`; simulation default, explicit live intent, plan/resolve/one-action dispatch | Implemented |
| Edge configuration and CLI installation | `references/edge-configuration.md` and `references/cli-command-map.md`; checked against current CLI source | Implemented |
| Asset onboarding | `references/asset-and-driver-development.md`; URDF validation, capability preview, test twin/render | Implemented |
| Driver creation | `references/driver-development.md`, `scripts/scaffold_driver.py`, and `assets/driver-template/` | Implemented |
| Robot/video monitoring | `references/robot-monitoring.md`; bounded frames/streams, telemetry, freshness, runs, alerts and edge state | Implemented |
| Registration and authentication | `references/registration-and-auth.md`; signup/profile, CLI token creation, hosted MCP authentication and human-owned steps | Implemented |
| MCP reuse and non-MCP boundaries | `references/mcp-and-fallbacks.md` capability ownership matrix | Implemented |
| Claude, Codex, ChatGPT/API and other clients | portable `SKILL.md`, `agents/openai.yaml`, Claude plugin metadata, package installer and upload guidance | Implemented |
| Detailed strategy before implementation | `docs/strategy.md` records architecture, safety, ownership, validation and rollout | Implemented |
| Monorepo as source of truth | `cyberwave-clis/cyberwave-skills/`, `AGENTS.md`, PR checklist and drift-trigger workflow | Implemented |
| Public distribution sync | `.github/workflows/claude-plugin-sync.yml` targets the existing singular skill repository plus the plugin compatibility mirror | Implemented |
| Local Claude installation | `~/.claude/skills/cyberwave` links to the canonical monorepo worktree skill | Verified locally |
| GitHub skill worktree | singular compatibility worktree on `codex/agentic-skills-system`; canonical monorepo worktree on `codex/cyberwave-agentic-skills-source` | Implemented |

## Automated gates

`scripts/validate_skills.py` proves:

1. exactly one discoverable `cyberwave` skill exists;
2. portable frontmatter, local links, provider metadata, eval scenarios and secret scans pass;
3. documented MCP tools exist in current MCP registration source;
4. documented CLI groups and edge commands exist in current CLI source;
5. the embedded driver scaffold resolves every placeholder and generates valid Python;
6. with Python 3.11+ and `--sdk-source`, the generated driver is concrete under the current `BaseDriver`, constructs through `create()`, and produces the expected manifest;
7. Claude plugin and hosted MCP metadata are valid.

The standalone `skills/cyberwave/scripts/validate_skill.py --compare` check proves that the singular compatibility worktree is byte-identical to the canonical skill directory.

## Public repository decision

On 2026-09-01, the public distribution was consolidated on the existing `cyberwave-os/cyberwave-skill` repository to avoid maintaining singular and plural duplicates. The initial structural update is synchronized manually for review; subsequent production syncs use the monorepo-owned subtree. The plural repository is intentionally not a workflow target.
