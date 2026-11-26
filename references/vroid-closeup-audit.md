# VRoid Ultra-Close-Up Audit & Height-Locked Opacity Overlay Protocol

> On-demand reference module for `KAGAMI`. Loaded when running multi-POV inspection via `vroid-studio` / `NINGYO` MCP (`batch`, `camera`, `screenshot`) or comparing 3D VRoid renders against 2D canonical character sheets.

---

## 1. `vroid-studio` MCP Session & `camera` Calibration Rules
- **Blocking & File Hygiene:**
  - `open` and `create` are valid ONLY from `main_menu`; always call `main_menu` `{}` before rewriting `.vroid` on disk with Python, then `open` with uppercase Windows drive letter (`D:\...`).
  - After rebuilding `.vroid`, `save` `{}` once approved inside VRoid Studio so thumbnails update cleanly, then sync `.vroidcustomitem` archives from the post-save `.vroid`.
  - **NEVER call `save`** after temporarily clearing outfit/hair layers (`select_preset(..., index=0)`); discard via `main_menu` `{}` and re-`open` the clean saved `.vroid`.
- **Camera `frame`, `zoom` Math & `batch` Pitfalls:**
  - `camera(action="frame", region="<body|face|head|upper|legs|feet>", angle="<front|back|left|right|threequarter_left|threequarter_right>")`: Always pass `"region"` and `"angle"` (passing `"part"`/`"view"` silently falls back to full `"body"` at `distance=5.186m`).
  - `camera(action="zoom", value=V)` steps camera distance linearly inward by **`~0.111m` per unit `V`** (e.g., from `frame face` `1.447m`, `zoom(3.5)` -> `1.058m`; from `frame body` `5.186m`, `zoom(26)` -> `2.297m` for upper torso/waist).
  - Always use **`"action": "zoom"`** / **`"action": "angles"`** inside `batch` operations (never `"mode": "zoom"`).
- **8-Screenshot Cap per `batch` Call:** `batch` returns/offloads at most **8 inline screenshots** per call (screenshots 9+ get dropped from output). Split head (<=8) and full-body 360° (8) into separate `batch` calls.
- **`frame upper` Distance:** Notice `camera(action="frame", part="upper", view="front")` sits at `distance=5.186m` on saved full models; use `zoom(26)` -> `2.297m` for full upper torso + hands, or `zoom(32)` -> `1.630m` for clavicle/collar close-ups.
- **Editor Pose Limitation (T-Pose vs Photo Booth):**
  - Avatar Editor and `vroid-studio` MCP (`Face`, `Hairstyle`, `Body`, `Outfit`) keep the avatar in the fixed ~45° T-pose. Custom A-pose or lowered-arm posing requires VRoid Studio's **Photo Booth** mode (`Pose & Expression`) or `.vrm` export into Blender / VSeeFace / Warudo.

---

## 2. Single-Batch 6-Angle Bug-Hunting Audit (Run After Every Edit)
Standard 5-view front/side/back renders miss eye-piercing front strands, shoulder seam gaps, strap kinks, and nape scalp smudges. Run this exact close-up diagnostic set inside one `batch` call:

| # | Camera Sequence | What It Exposes Immediately |
|---|---|---|
| 1 | `frame(face, front)` -> `zoom(3.5)` | Forehead parting gaps, center lock alignment, eye-vs-bangs clearance, brow/lens framing |
| 2 | `frame(head, threequarter_left)` -> `angles(140, 30)` -> `zoom(4)` | Diagonal top-down bangs piercing eyeballs, hairclip float (`cz`), side locks thickness |
| 3 | `frame(head, threequarter_right)` -> `angles(-140, 25)` -> `zoom(4)` | Asymmetric opposite-side eye clearance, frame fit, side hair flick attachment |
| 4 | `frame(upper, threequarter_left)` -> `angles(130, 20)` -> `zoom(18)` | Transparent shoulder gaps (`Tops`), bra strap seam kink (`InnerTop`), placket zipper edge |
| 5 | `frame(upper, back)` -> `angles(30, 20)` -> `zoom(18)` | Back shoulder stitch continuity, collar drape fold, stray zipper teeth at neckline |
| 6 | `frame(upper, left)` -> `angles(90, -30)` -> `zoom(16)` | Dark brown `BodySkin PNG[1]` undercut beneath nape, underarm dark collar leaks, pocket welt angle |

Pair with `feet:front`, `feet:threequarter_left + zoom(3.5)`, `feet:left + zoom(3.5)`, `feet:back` when inspecting footwear proportions.

---

## 3. Progressive Layer-by-Layer Stripping Audit
To verify every garment layer independently without occlusion:
1. Capture **Full Outfit** (upper/close-up batch + 8-angle 360° `frame(body, front)` -> `angles(135, -5.7) .. (-135, -5.7)` -> `save`).
2. **Layer 2 (`InnerTop` + `Bottoms`):** `select_preset("Outfit", "Tops", 0)` (`(clear)`) -> capture `upper:front`, `view(0.8, 0.22)` (collarbone straps), `angles(90, 15)` (side shoulder seam), `angles(135, 30)`, `angles(35, 25)`, `upper:back + zoom(14)`. Verify continuous strap over shoulder ridge before saving final approval.
3. **Layer 0 (`Body Only`):** Clear `Bottoms`, `Socks`, `Shoes`, all 6 `Hairstyle` slots (`Front`, `Back`, `Ahoge`, `Side`, `Extra`, `Base Hair`), set `Inner Top = 1` (`Black Bandeau Top`) -> inspect `body` 360° + `head:back + zoom(4)` to verify clean anatomy and zero scalp undercut.
4. **Isolated Glasses (No Hair):** Capture `face:front`, `head:threequarter_left`, `head:left` at `zoom(3.5)` -> immediately `main_menu` `{}` (discard!) -> `open` saved clean `.vroid`.

---

## 4. Height-Locked 50% Opacity Overlay Comparison
- Never scale VRoid front/back renders by full bounding-box width against an A-pose 2D character sheet: extended 45° T-pose arms shrink torso height by ~35%.
- Extract foreground mask by chroma distance from top-corner background (`diff > 14`), crop to vertical `[y_min .. y_max]` centered horizontally on `median(xs)` (robust to asymmetric ahoge/arms), scale VRoid and Ref strictly by **`scale = avail_h / ch`**, blend `0.5 * ref + 0.5 * vroid`, and report silhouette `IoU` + `W/H` aspect ratio.
- For **Face Close-Up**, strip the narrow ahoge and lower torso below neck pinch (`row_w >= 0.18 * max_w` top down to `argmin(row_w)` in `[0.55H..0.92H]`) before resizing so head scale matches 1:1.

---

## 5. NINGYO v2.0 (38 Tools) Automation & Verified Pitfalls

- **Direct Headless Texture & Expression Automation**: Use `texture_list` -> `texture_export_layer` / `texture_import_layer` to swap raster PNG layers headlessly (zero OS dialogs, undoable), and `expressions_list` / `expression_set` to drive all 14 Expression Editor categories; pair with `photobooth`, `look_get`/`look_set`, `accessory_list`/`add`/`remove`/`params`, `undo`/`redo`, and `body_metrics`.
- **IL2CPP Collection Enumeration & Streamable HTTP `Mcp-Session-Id`**: `Il2CppSystem.Collections.Generic.IReadOnlyCollection<T>` does not implement managed `System.Collections.IEnumerable` or `get_Item(int)`, and `/mcp` rejects `tools/list` or `tools/call` without the `Mcp-Session-Id` header from `initialize`. Enumerate via `Il2CppObjectBase.TryCast<Il2CppSystem.Collections.IEnumerable>()` (plus `FaceEditor.InitExpression` lazy fallback) and persist `Mcp-Session-Id` across HTTP calls.
- **Duplicate Dark Silhouette After `photobooth(exit)` / `export_vrm`**: Calling only `DeselectExportMode()` + `ScreenNavigation.Pop()` leaves `_umaGameObject` / `MeshReduction._referenceCache` active behind the character. Always call `OnLeaveExportEditMode()`, `MeshReduction.ClearReference()`, `ClearReference()`, `ClearPreviewAvatar()`, and `OnEditorWindowModeLeave()` on exit.
- **Visible Window Launch (Avoid `MainWindowHandle = 0`)**: Launching `VRoidStudio.exe` directly from PowerShell/Python can run inside a hidden/sandboxed window station (`MainWindowHandle = 0`) and exit with the Job Object. Always launch/restart via `"C:\Program Files (x86)\Steam\steam.exe" -applaunch 1486350` (or `ningyo_launcher.py launch`), keep `[Logging.Disk] Enabled = true` in `BepInEx\config\BepInEx.cfg`, and delete `BepInEx\cache\chainloader_typeloader.dat` when adding/updating plugins.
- **Camera Frozen at `{0, 0.8, 6}` & Export UI Chrome**: `BackToModelSelect` (`main_menu`) unbinds `VRoidStudioAvatarCameraViewController`, and `export_vrm` leaves Export UI chrome over screenshots if called first. Call `exit` and relaunch visibly via `steam.exe -applaunch 1486350`, start screenshot `batch` with `enter(section="Face", category="Mouth")`, and ALWAYS capture Edit-Mode screenshots BEFORE calling `export_vrm`.
- **Stale `.vrm` Audit Trap in Blender**: Always verify `asset.generator` (`VRoid Studio-...`), `mtime`, and embedded `thumbnailImage` on freshly exported `.vrm` files before A-pose inspection, and pose only `J_Bip_L/R_UpperArm` downward (~30–35° around Y).
