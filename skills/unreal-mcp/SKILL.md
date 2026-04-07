---
name: unreal-mcp
description: "Control Unreal Editor via MCP: create actors, edit Blueprints, manipulate widgets, materials, Niagara, Control Rigs, and more"
---

# Unreal MCP

Unreal MCP exposes hundreds of tools from the Unreal Editor through an MCP server. Tools are registered via **EDA's ToolsetRegistry** and bridged to MCP automatically. When the editor is running, all registered toolsets are available via the `unreal-mcp` MCP server.

All tools are auto-discovered via MCP. You do not need to memorize tool names. They appear in Claude Code's tool list when the editor is connected.

## Prerequisites

- Unreal Editor running with the ModelContextProtocol and ClaudeCode plugins enabled
- MCP server active (check Output Log for startup messages, or open Tools > Claude Code)
- Default endpoint: `http://localhost:8000/mcp`

## Safety

- **Save before bulk operations.** MCP tools modify editor state directly. Save before and after large changes.
- **Wait for compilation.** Don't issue tool calls while C++ or shader compilation is in progress.
- **PIE context.** Some editor-only tools (e.g., asset creation) may behave differently during Play-in-Editor.
- **Sequential execution.** Tool calls execute on the game thread. Don't issue them in parallel.
- **Check results.** Operations like Blueprint compilation can fail. Always check the result before proceeding.

## Workflows

### Inspect a Level

1. `SceneTools.find_actors` to list actors in the level
2. `ObjectTools.list_properties` then `ObjectTools.get_properties` to inspect a specific actor
3. `EditorAppToolset.CaptureAssetImage` to screenshot the viewport (pass the current level path as `assetPath`)

### Create and Place Objects

1. `SceneTools.add_to_scene_from_asset` or `add_to_scene_from_class` to spawn an actor
2. `ActorTools.set_actor_transform` to position it
3. `ObjectTools.set_properties` to configure properties
4. `EditorAppToolset.FocusOnActors` to center the viewport on it

### Blueprint Creation

1. `BlueprintTools.create` with a parent class
2. `ActorTools.add_component` to add components
3. `ObjectTools.set_properties` to configure component properties
4. `BlueprintTools.add_variable` to add member variables
5. `BlueprintTools.get_graph` to get the EventGraph
6. `BlueprintTools.create_node` to add nodes
7. `BlueprintTools.connect_pins` to wire pins together
8. `BlueprintTools.compile_blueprint` to compile and check for errors

### Material Instances

1. `MaterialInstanceTools.create` with an existing parent material
2. `MaterialInstanceTools.list_parameters` to see what parameters are exposed
3. `MaterialInstanceTools.set_scalar_parameter` / `set_vector_parameter` / `set_texture_parameter` to configure values
4. Assign the instance to meshes via `ObjectTools.set_properties`

### Widget/UI Creation

1. `UMGToolSet.CreateWidgetBlueprint` to create a Widget Blueprint
2. `UMGToolSet.AddWidget` to add UI elements (text, buttons, images)
3. `ObjectTools.set_properties` to configure widget and slot properties
4. `UMGToolSet.CompileWidgetBlueprint` to compile

### Niagara Particle Effects

1. `AssetTools.find_assets` to find template Niagara systems (e.g., under `/Niagara`)
2. `NiagaraTools.System_CreateNiagaraSystem` from a template
3. `NiagaraTools.System_GetSystemTopology` to understand the structure
4. `NiagaraTools.System_GetModuleSchema` to see available inputs
5. `NiagaraTools.System_SetStackInputData` to modify parameters

### Slate UI Automation

1. `SlateInspectorToolset.Snapshot` to get the widget tree
2. `SlateInspectorToolset.Observe` to start tracking a subtree
3. `SlateInspectorToolset.Click` / `Type` / `PressKey` to interact with widgets
4. `SlateInspectorToolset.Screenshot` for visual verification
5. `SlateInspectorToolset.Unobserve` to clean up when done

### Batch Operations

1. `ProgrammaticToolset.get_execution_environment` to learn the API (call once per session)
2. `ProgrammaticToolset.execute_tool_script` to run a Python script that calls multiple tools in one round-trip

## Toolset Reference

All toolsets listed below are registered via EDA's ToolsetRegistry and grouped by domain.

### Actors and Scene

**ActorTools** manages actors and components: get/set transforms, labels, bounds, add/remove components, query component hierarchy, and reparent scene components.

**SceneTools** handles level and scene management: spawn actors from assets or classes, find actors by type/name, delete actors, load levels, manage outliner folders, and get the current level path.

**PrimitiveTools** adds procedural DynamicMesh shapes to actors: cubes, spheres, and cylinders/cones with configurable dimensions and local transforms.

### Blueprints

**BlueprintTools** provides full Blueprint editing: create blueprints, add/remove function graphs and variables (primitive, struct, object types), create/delete/connect nodes, set pin values, manage function parameters, compile, reparent, query node types/categories, and bind component events.

### Assets and Content

**AssetTools** covers asset and file management: find, load, save, move, duplicate, delete assets and folders. Read/write text files. Check dirty state and discover plugin content paths.

**DataAssetTools** creates DataAsset instances from a specified class.

**DataTableTools** creates and edits DataTable assets: add/remove/rename rows, get/set row values as JSON, query column schemas, and search for compatible row structs.

**CurveTableTools** creates and edits CurveTable assets: add/remove/rename rows, add/set/get keys (time/value pairs).

**StringTableTools** creates and edits StringTable assets: add/remove/get entries by key, list keys, and query namespace/table ID.

### Materials

**MaterialTools** authors Materials: add expression nodes (constants, parameters, texture samples, math ops), connect expressions to each other and to material outputs, delete expressions, auto-layout, and recompile shaders.

**MaterialInstanceTools** creates and edits Material Instances: set/get scalar, vector, texture, and static switch parameters. List exposed parameters, change parent, and clear overrides.

### Meshes and Textures

**StaticMeshTools** inspects and edits Static Meshes: query bounds, LODs, triangle/vertex counts, material slots, Nanite state. Generate convex collisions and LODs, remove collisions/LODs, and toggle Nanite.

**SkeletalMeshTools** inspects and edits Skeletal Meshes: query bones (hierarchy, parents, children), material slots, sockets (add/remove/rename/transform), morph targets, LODs, bounds, vertex counts, and assign physics assets.

**TextureTools** queries Texture2D dimensions (width/height in pixels).

### Objects and Properties

**ObjectTools** provides generic UObject inspection: get class, list properties, get/set property values (as JSON), and search for subclasses of any class. Works on any UObject.

### Animation

**AnimationToolset (ControlRig)** builds and edits Control Rigs: create rigs, add bones/nulls/controls, manage graphs (forward/backward/interaction solve), create RigUnit nodes, connect pins, set transforms, and manage variables.

**StateTreeToolset** inspects StateTree assets: get root states, child states, tasks, transitions, enter conditions, evaluators, global tasks, and node descriptions.

**AIModuleToolset** inspects and navigates Behavior Trees: list nodes, get children, depths, decorators, blackboards, and subtree references.

### VFX

**NiagaraAIAssistantTools** provides full Niagara system authoring: create systems/emitters, add/remove modules and renderers, get/set stack inputs, manage user variables, query schemas, create Blueprint wrappers, and set component variables.

**DataflowAgent** creates and edits Dataflow graphs: add/remove/connect nodes, manage variables, inspect node type schemas, reposition nodes, and add comment boxes.

### UI

**UMGToolSet** builds UMG widget blueprints: create blueprints, add/remove/move/rename widgets, manage named slots, set widget-as-variable, compile, and reparent.

**SlateInspectorToolset** provides UI automation and inspection: snapshot widget trees, click/type/hover/drag widgets, fill forms, select combobox options, press keys, take screenshots, observe subtrees, and manage windows.

### Gameplay

**GameplayTagsToolset** manages gameplay tags: add, remove, rename tags and inspect tag details across tag sources.

**GASToolsets** covers three sub-toolsets for the Gameplay Ability System: **AbilitySystemInspector** (query active effects, tags, attributes, granted abilities on actors), **AttributeSet** (find attribute set classes and list their attributes), **GameplayCue** (manage cue tags, create notify assets, execute cues, find orphaned tags).

**GameFeaturesToolset** creates and inspects Game Feature Plugins: list plugins, find GameFeatureData assets, and query their actions.

**ConversationToolset** inspects Common Conversation assets: list entry points, speakers, navigate node graphs by GUID, get connections and sub-nodes.

**PhysicsToolsets** edits Physics Assets: add/remove bodies and constraints, set collision shapes (sphere/capsule/box), configure constraint limits, mass scale, and simulation modes. Create physics assets from skeletal meshes.

**WorldConditionsToolset** describes world conditions and world condition query definitions in human-readable form.

### Editor Utilities

**EditorAppToolset** handles editor interaction: capture asset images/viewport screenshots, get/set camera transform, select actors/assets, focus on actors, manage content browser path, screen-to-world coordinate conversion, search console variables, and get visible actors.

**LogsToolset** provides log inspection: list log categories, get/filter log entries by category or regex pattern, and get/set verbosity levels.

**AgentSkillToolset** manages Agent Skills: list, get details, create, and update skill assets.

**ClaudeCodeToolset** returns the docked editor context: which asset editor the Claude Code widget is docked in, focused graph, and selected nodes.

**AIAssistantToolset** returns docked editor context and project context for AI assistant widgets docked in asset editors.

### Scripting

**ProgrammaticToolset** executes Python scripts against all toolset APIs in a single call. Call `get_execution_environment` first to discover available modules and usage instructions.

## Configuration

### Editor Preferences > Plugins > Model Context Protocol

| Setting     | Default | Description                   |
|-------------|---------|-------------------------------|
| Server Port | 8000    | HTTP port for MCP server      |
| URL Path    | /mcp    | MCP endpoint path             |
| Auto Start  | false   | Start server on editor launch |

### Console Commands

| Command                                   | Description                                   |
|-------------------------------------------|-----------------------------------------------|
| `ModelContextProtocol.StartServer [port]` | Start the MCP server (optional port override) |
| `ModelContextProtocol.StopServer`         | Stop the MCP server                           |
| `ModelContextProtocol.RefreshTools`       | Re-register all tools                         |

## Troubleshooting

| Issue                | Solution                                                                                                                 |
|----------------------|--------------------------------------------------------------------------------------------------------------------------|
| MCP not connecting   | Ensure editor is running. Open Tools > Claude Code to trigger MCP server start. Check Output Log for errors.             |
| Failed to listen     | Run `ModelContextProtocol.StartServer <port>` with a different port number.                                              |
| Tools not appearing  | Run `ModelContextProtocol.RefreshTools` console command. Check that ToolsetRegistry plugins are enabled.                 |
| Tool calls failing   | Check Output Log for error details. Ensure editor is idle (not compiling, loading).                                      |
| Docked context empty | Claude Code tab must be docked inside an asset editor (Blueprint, Material, etc.) for `GetDockedContext` to return data. |
