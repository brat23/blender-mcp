# Strategic Codebase Audit (VP of Google Perspective)

## 🎯 Executive Summary
While the Blender-MCP project has strong core functionality, it currently operates as a "script-heavy" prototype. To transition to a "World-Class" integration, we must shift from a reactive bug-fixing model to a proactive, architecturally sound platform.

---

## 🛑 Critical "Granular" Bugs (Immediate Fix Required)

### 1. Blocking I/O in Main Thread 🐞
- **Issue**: Tools like `import_generated_asset_main_site` use `requests.get(stream=True)` inside the addon logic. 
- **Impact**: While the data is received in chunks, the `while` loop blocks the Blender UI thread during the entire download of a 50MB+ GLB.
- **Fix**: Move asset downloads to the `BlenderMCPServer` background thread and only use the `Timer` (Main Thread) for the final `bpy.ops` import call.

### 2. Fragile GLB Post-Processing 🐞
- **Issue**: `_clean_imported_glb` assumes a specific hierarchy (Empty node with one mesh child).
- **Impact**: If a model has multiple meshes or a different internal structure, the cleanup fails or deletes the wrong object.
- **Fix**: Implement a more robust "collection-based" tracking where all objects in the imported collection are analyzed and normalized.

### 3. Parameter Injection & Validation 🐞
- **Issue**: The command router in `_handle_client` takes `params` from JSON and passes them directly to Blender functions (e.g., `execute_code(params['code'])`).
- **Impact**: Malformed or malicious code/params can cause unhandled exceptions that crash the socket thread.
- **Fix**: Implement a Schema Validator (e.g., a simple dict-based checker) before any command hits the execution engine.

---

## 🚀 Strategic Feature Roadmap (Required for Scale)

### 1. Unified Context Management 💎
- **Requirement**: The AI currently doesn't know if Blender is in `EDIT` mode vs `OBJECT` mode. Many `bpy.ops` fail silently if the mode is wrong.
- **Feature**: `get_scene_info` should include `context.mode` and `context.workspace`. The server should automatically attempt to switch modes if a tool requires it.

### 2. Asynchronous Progress Reporting 💎
- **Requirement**: For long tasks (Rodin generation), the user sits in front of a frozen or "waiting" Claude.
- **Feature**: Implement a `progress_update` event in the socket protocol. Blender should report `%` completion back to the MCP server, which can then update the user.

### 3. Integrated Asset Registry 💎
- **Requirement**: Users have to manually remember if they used Polyhaven or Sketchfab.
- **Feature**: Maintain a local SQLite or JSON database of all assets imported via MCP. This allows the AI to "Refactor" or "Replace" existing assets by referring to their IDs.

### 4. Multimodal Feedback Loop (Auto-Screenshot) 💎
- **Requirement**: AI currently waits for the user to ask for a screenshot.
- **Feature**: Implement a "Verify" step where the server automatically triggers a viewport capture after a major modification and feeds it back to the AI for self-correction.

---

## 📊 Effectiveness Assessment
- **Reliability**: Fixing Blocking I/O will improve perceived performance by 10x for large assets.
- **Safety**: Parameter validation is non-negotiable for any tool being distributed to users.
- **Growth**: The Asset Registry and Context Management turn this from a "command executor" into a true "AI Co-pilot" that understands the state of the project.
