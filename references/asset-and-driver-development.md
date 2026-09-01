# Asset and driver development

Read this when onboarding a new catalog asset, uploading URDF/visual files, defining capabilities, or implementing a hardware driver.

## Distinguish the deliverables

- **Asset:** catalog/digital representation, files, metadata, universal schema, capabilities, visualization, and documentation.
- **Twin:** environment instance of an asset.
- **Driver:** edge service that bridges a real device's native interface to the Cyberwave twin/topic model.

A new asset does not always need a new driver, and a new driver may target an existing asset. Ask only enough to determine which deliverables are required.

## Reuse first

1. Search catalog/primitives for the exact or compatible hardware.
2. Inspect capabilities and setup documentation.
3. Check existing Cyberwave edge drivers and the detailed [driver-development reference](driver-development.md).
4. Extend an existing asset/driver when the protocol and command surface are genuinely compatible.

Do not fork a driver merely for a different display name or environment placement.

## Asset onboarding

Collect:

- manufacturer/model/version and stable name,
- asset type and physical dimensions/frame convention,
- URDF/package or GLB/visual source,
- joints, limits, sensors, movements/poses, and commands,
- visibility/workspace ownership and license/provenance,
- existing driver compatibility and setup guide.

For URDF through MCP:

1. Validate the ZIP locally: main URDF path, referenced meshes/textures, case-sensitive paths, units, joints/limits, and no unsafe paths.
2. Preview/resolve workspace and visibility.
3. Call `cw_create_urdf_asset_from_zip` with the exact package and main file when needed.
4. Immediately call `cw_set_asset_capabilities(execute=false)` with capabilities grounded in the asset/driver, inspect the preview, then execute.
5. Fetch/search the asset, instantiate a test twin with `cw_add_twin_to_environment`, and render/inspect it in a safe environment.

Capabilities live under `/extensions/cyberwave/capabilities`. Do not infer capabilities from appearance alone. Verify field names and semantics from the current Digital Twins documentation or schema before writing them.

## Driver development handoff

For a driver deliverable, continue with [driver development](driver-development.md). The deterministic scaffold and template are resources of this same `cyberwave` skill; there is no separate driver skill to install or maintain.

## Completion evidence

For assets, report asset UUID/slug, files, capability status, test twin, and render/schema validation. For drivers, report repository/path, manifest summary, build/tests, development twin/edge binding, and what remains before live deployment.
