# Cyberwave Agent Skill

A portable Agent Skill for building and operating Physical AI systems on [Cyberwave](https://cyberwave.com). One `cyberwave` entrypoint routes agents to focused guidance for:

- registration, authentication, and Cyberwave MCP connection
- environment creation and editing
- workflow authoring and run management
- safe robot control in simulation and live mode
- edge pairing, drivers, workers, and MQTT/Zenoh configuration
- asset onboarding and driver development
- robot telemetry, camera, workflow, and edge monitoring

The core follows the open [Agent Skills](https://agentskills.io) format. Cyberwave MCP is the preferred typed execution plane when available, but the skill degrades to the verified SDK, CLI, dashboard, or official documentation.

## Install

### Canonical package (Claude Code or Codex)

```bash
git clone https://github.com/cyberwave-os/cyberwave-skills ~/.cyberwave/agent-skills
python3 ~/.cyberwave/agent-skills/scripts/install_skills.py --client claude
# or: --client codex
```

Use `/cyberwave` in Claude Code, `$cyberwave` in Codex, or describe a matching Cyberwave task and allow automatic discovery.

### Singular compatibility mirror

```bash
git clone https://github.com/cyberwave-os/cyberwave-skill ~/.claude/skills/cyberwave
# or clone to ~/.codex/skills/cyberwave
```

The compatibility mirror contains the same `skills/cyberwave` content but not the package-level installer/plugin wrapper.

### Project-local or other Agent Skills clients

Clone/copy the repository as a directory named `cyberwave` beneath the client's project or user Agent Skills search path. The required entrypoint is `SKILL.md`; provider-specific metadata is additive.

For ChatGPT or API runtimes that accept uploaded skill bundles, upload the same directory/ZIP and promote an immutable tested version. Keep MCP credentials in runtime configuration, not the bundle.

## Cyberwave MCP

Hosted endpoint: `https://mcp.cyberwave.com/mcp` (Streamable HTTP). It uses a user-scoped Cyberwave API key in the `Authorization: Bearer ...` header. Configure the key with the client's secret mechanism; never commit it.

The skill discovers the tools actually exposed by the current client. MCP is optional for guidance and code authoring, but live platform execution and verification require an authenticated execution plane.

## Architecture

`SKILL.md` is a compact orchestrator. It loads only the relevant module from `references/` for the current task. This keeps the discovery and activation context small while retaining detailed domain procedures.

See [the strategy](docs/strategy.md) for the routing model, safety/authorization policy, cross-agent support, validation matrix, and rollout plan.
See [the implementation audit](docs/implementation-audit.md) for requirement-level release evidence and the remaining public-repository gate.

## Development and ownership

The maintained source of truth lives in the Cyberwave monorepo:

```text
cyberwave-clis/cyberwave-skills/skills/cyberwave/
```

The production workflow `.github/workflows/claude-plugin-sync.yml` mirrors the full package to `cyberwave-os/cyberwave-skills` and this directory to the singular compatibility repository with `rsync --delete`. Public-mirror-only changes will therefore be overwritten. Changes to MCP tools, SDK/CLI interfaces, edge/runtime behavior, backend auth/control/workflows, documentation, or driver interfaces must include a Cyberwave skill impact review.

Validate the package:

```bash
python3 scripts/validate_skill.py
```

From a monorepo checkout, also validate every referenced MCP tool:

```bash
python3 ../../scripts/validate_skills.py \
  --mcp-source ../../../cyberwave-mcp-server/cyberwave_mcp_server \
  --cli-source ../../../cyberwave-python-cli/cyberwave_cli \
  --sdk-source ../../../../cyberwave-sdks/cyberwave-python
```

Before distribution, compare the monorepo source with a standalone checkout:

```bash
python3 scripts/validate_skill.py --compare /path/to/other/cyberwave-skill
```

The evaluator scenarios in `evals/scenarios.json` define the expected routing and safety decisions for realistic user requests. Tests should assert those decisions and observable effects rather than exact prose.

## License

Apache-2.0.
