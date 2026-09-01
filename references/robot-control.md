# Robot control

Read this for navigation, motion poses, joint commands, stop commands, observation actions, controller-policy routing, and simulation/live handoff.

## Classify the request

Determine:

- target environment and twin,
- requested action and bounds,
- mode: `simulation` or `live`,
- whether the user requested a plan or execution,
- and what observation will prove success.

If mode is omitted, use `simulation`. Do not infer live mode from words like "robot", "edge", "camera", or a physical asset name.

## Resolve target and affordances

1. Use `cw_resolve_twin` or `cw_list_twins` until exactly one twin is selected.
2. Use `cw_list_control_surfaces` to inspect available movement, pose, joint, observation, controller-policy, and stop affordances.
3. Inspect twin/schema/joint state or edge/driver health when it affects safe execution.
4. If the target or affordance is ambiguous, ask the user to choose; do not guess.

## Plan

Use `cw_plan_control_action` as the primary entry for move, navigate, stop, observe, pose/motion, or joint intents. State the action precisely and pass explicit environment/twin/mode.

Use `cw_resolve_control_route` only as a lower-level pre-validation step when a specific route needs structured inputs. If it reports `needs_setup`, fill missing fields only from accepted target shapes and listed candidates, then plan again.

Inspect the plan for:

- selected twin and mode,
- exactly described action(s),
- limits/units/targets,
- warnings and missing requirements,
- and whether dispatch is required.

Planning never authorizes motion.

## Simulation execution

When the user requested execution, pass exactly one returned action to `cw_dispatch_control_action` with `mode="simulation"`. Verify its result and observed state before another action.

Atomic tools such as `cw_set_joint`, `cw_motion_pose`, `cw_navigation_goto`, and `cw_navigation_stop` remain useful for supported direct/diagnostic flows. Inspect their schemas and preview/execute behavior. Prefer the plan/dispatch path for general agent control.

## Live execution gate

Before dispatching with `mode="live"`, all of these must be true:

1. The current user request explicitly says live, real, or physical execution—not merely planning or code generation.
2. One physical target twin is resolved by stable identifier.
3. The action is bounded: one pose, one joint delta/target, one navigation destination, or one stop.
4. Compatible control capabilities and a ready control route are present.
5. Edge/driver health and relevant telemetry are sufficiently current for the action.
6. The user has not asked to bypass limits or safety systems.
7. A stop/recovery path is available.

Then show the resolved target, mode, and bounded action and use the client's confirmation/approval flow. Pass exactly one planned action to `cw_dispatch_control_action` with the confirmation field required by the live tool schema.

If any condition is false, do not dispatch. Offer simulation, planning, a stop, or the specific missing setup step.

## Stop and uncertain state

A stop request has priority over new motion. Resolve the target as quickly as safely possible and use the planned/direct stop route supported by the tool set. Do not wait for nonessential monitoring.

If dispatch times out or returns an uncertain result:

- do not retry the motion automatically,
- inspect current state/telemetry,
- use a stop if it is safe and supported,
- report the uncertainty and request operator direction.

## Continuous control

Do not translate a single request into an unbounded motion loop, autonomous live policy, or indefinite teleoperation session. Continuous live operation requires explicit scope, duration/termination conditions, operator controls, and a purpose-built controller/workflow.

## Completion evidence

Report target twin name/UUID, mode, planned action, whether dispatch occurred, dispatch/request ID, returned/observed state, and stop/recovery status when relevant.
