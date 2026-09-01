# Environment management

Read this for environment creation, scene building, catalog twins, primitives, areas, waypoints, transforms, property edits, layout validation, and object deletion.

## Establish context

1. Use `cw_get_environment_context` if an environment is already selected.
2. Otherwise resolve workspace/project/environment through the registration/context workflow.
3. Create a missing environment with `cw_create_environment(execute=false)` first, show the resolved workspace/project and proposed name, then execute when the user's request authorizes creation.
4. Refresh `cw_get_environment_context` and retain the created environment UUID.

Do not create a second environment merely because its name differs slightly. List and resolve first.

## Plan the scene

Translate the request into a short object inventory with stable semantic names/IDs, types, approximate sizes, and explicit positions. In Cyberwave's Z-up frame, positions are `[x, y, z]`; Euler rotations are `[roll, pitch, yaw]` in radians unless the tool schema says otherwise.

Use the right kind:

- catalog twin: robot, sensor, or reusable catalog asset
- procedural primitive: tangible floor, wall, platform, obstacle, box, stairs, enclosure, or support surface
- area: intangible safety/no-go, patrol, inspection, staging, target/search, or coverage zone
- waypoint: named navigation location on the navigable floor

Do not model physical walls/floors as areas. Do not put ground-robot route waypoints at table/rack height.

## Discover assets and primitives

1. For common curated catalog objects, call `cw_list_primitives` first.
2. Use `cw_search_catalog` for broader or multi-object lookup; batch queries when the schema supports it.
3. Treat results as candidates, not scene edits. Select the exact returned catalog/procedural identifier.
4. Prefer a catalog result with matching capabilities and a maintained setup guide. Do not substitute a vaguely similar robot without telling the user.

## Create and edit

Prefer `cw_edit_environment` for normal multi-object or intent-level scene edits. Copy current `scene_edit_operation_hint` values from catalog search results, give every non-ground object and waypoint an explicit position, and create support surfaces before operations that snap to them.

Use atomic tools when the task is narrow or the intent-level tool is unavailable:

- `cw_add_twin_to_environment`
- `cw_create_procedural_primitive`
- `cw_create_area`
- `cw_create_waypoint`
- `cw_transform_environment_object`
- `cw_update_environment_entity_properties`
- `cw_resize_area`
- `cw_set_area_image`

For catalog twins, use the `registry_id` returned by current MCP catalog search because that MCP contract explicitly accepts it. Give the twin an explicit position during creation when possible.

Use `snap`/`snap_target` against exact existing IDs for floors, tables, racks, or walls. Use overlap avoidance/clearance options where available instead of estimating repeated nudges.

## Validate spatially

After material edits:

1. Refresh `cw_get_environment_context` or `cw_list_environment_entities`.
2. Run `cw_analyze_environment_layout` for fast overlap/stacking checks.
3. Run `cw_render_environment_preview` for visual validation. Use a valid single view name from the tool schema; do not invent composite view names.
4. Correct unintended intersections, floating objects, blocked routes, or hidden important assets.

One top/isometric view can miss vertical problems. Use front/side or perspective when height and stacking matter.

## Deletion

Use `cw_delete_environment_object` only after listing/resolving the exact area, waypoint, twin, or procedural primitive. State the environment and target ID/name, obtain explicit deletion authorization, then use the host confirmation flow. Refresh context afterward.

Never use `all` or a broad name match unless the user explicitly requested that exact scope and the tool's preview shows the complete target set.

## Completion evidence

Report:

- environment name and UUID/slug,
- created/changed object names and stable identifiers,
- important transforms,
- analysis/render result,
- and any unresolved visual or capability warning.
