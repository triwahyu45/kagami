# VRoid Studio Binary Protobuf & Texture UV Schemas

> On-demand reference module for `vroid-studio-protobuf-customizer`. Loaded when inspecting or editing `v1model/data.bin` Protobuf trees, `BrushHair` 3D geometry, float parameters, or clothing/skin UV islands.

---

## 1. Archive Layout & Top-Level `v1model/data.bin` Field Order
- **`.vroid` archive (ZIP):** `meta.json`, `v1model/meta.json`, `v1model/data.bin`, `thumbnails/thumbnail.jpg`.
- **`.vroidcustomitem` archive (ZIP):** Identical layout (uses `v1model/data.bin`, NOT `v1customitem/data.bin`), differentiated by `"rootTransferableGroupTypeId"` and `"customItemName"` inside `meta.json` and `v1model/meta.json`.
  - Sync exported custom items after saving `.vroid` by copying updated `v1model/data.bin` + `thumbnails/thumbnail.jpg` while preserving each `.vroidcustomitem`'s `meta.json` and `v1model/meta.json`.
- **Mandatory Top-Level `v1model/data.bin` Field Order:**
  - `f1` (header) -> `f2` (recipe metadata) -> `f3` (`BaseCollection`: shared materials/PNGs in `f2`, blobs in `f6`) -> repeated `f4` (`Transferable`) -> repeated `f5` (`TransferableGroup`) -> `f6` (`AppState` — **must stay last**). Inserting `f4`/`f5` after `f6` causes Unity `Player.log` NullRef crashes.
- **`LocalizedString` / Curves:** Never write raw ASCII bytes into `LocalizedString` (`BrushHair.f7`, `BrushHairGroup.f10`) or omit `hairWidthCurveId` (`f4.f3.f9`); clone template field blocks verbatim.
- **Body / Face Deformation Sliders (`Transferable.f3.f5.f2`) & Blendshape UUID Discovery (`f3.f3`):**
  - Body/Face parametric sliders live in `f3.f5.f2` (not `f3.f2.f3` floats) as pairs `f1 = ASCII slider name` (e.g. `headScaleShrink`, `chinFullCheeked`, `eyePosDown`) and optional `f2` wire-5 `float32`; `f2` is omitted when value is `0.0` (serialize zeroed sliders as `[(1, 2, name_bytes)]` only).
  - Clothing/Accessory blendshape float UUIDs in `f3.f2.f3` can be mapped 1:1 to their human-readable names (`BlendShape.<Part>.<Name>`) directly from printable strings inside that same `Transferable`'s `f3.f3` field (`Specifics`).

---

## 2. 3D Head Coordinates, `BrushHair` Protobuf & Per-Strip Hair Shaping
- **`J_Bip_C_Head` Local Coordinate Handedness:** Facing front camera (`yaw=180`): **`-X` = viewer's right (character's left)**, **`+X` = viewer's left (character's right)**, `+Y` = up, `+Z` = forward toward nose/viewer.
  - Around bangs/eyebrow height on N00 head: `cy ≈ +0.118..0.125`, `cz ≈ +0.095..0.106`, outer eyes `|x| ≈ 0.018..0.052`, temples `|x| ≈ 0.076`.
- **`BrushHair` Key Protobuf Fields (`Transferable.f3.f4.f2` -> `f4` `HairGroup` -> `f1` `BrushHairGroup`):**
  - `f4.f5` (`HairMaterialReference`): Assign `BaseCollection` material ID (e.g. `HairFront`'s `SharedHairMaterialId`) so crown accessories (`Ahoge`, side flicks) match crown brightness 1:1 without `HairAhoge` darkening.
  - `f9` (`isInGuideSpace`): Set `0` to place points in `J_Bip_C_Head` local coordinates.
  - `f5` (Children `BrushHairGrouplet` -> `f3` -> `f1` `BrushHair`):
    - **`f3.f3.f3.f1` (`Width` float)** & **`f3.f3.f4.f1` (`Thickness` float)**: Controls actual rendered ribbon width/thickness per strip (scale directly when making delicate center locks or slender side locks).
    - **`f4` (`ControlPoint`):** `f1` = `Vec3(x,y,z)`, `f2` = normal `Vec3`, **`f3` UV Vector2: `f1` = horizontal `U` across strand, `f2` = lengthwise `V` root->tip (`0.02..0.26` for pure crown color, `0.12..0.99` full gradient)**.
- **Per-Strip `HairFront` & `HairSideburns` Classification:**
  - Never apply a single uniform `target_y` across `HairFront`: temple-depth groups (`mid_z < 0.085`) will end at eye height and pierce the eyeballs!
  - Classify each strip by `mid_z` and `|tip_x|`:
    1. **Center Bridge V-Lock (`mid_z >= 0.085` & `|tip_x| <= 0.015`):** Scale width `~0.62x`, pull smoothly forward (`z >= 0.109`) between eyes to glasses bridge (`y ≈ -0.165`).
    2. **Over-Eye Arch Bangs (`mid_z >= 0.085` & `|tip_x| > 0.015`):** Arch cleanly around eyebrows (`y ≈ -0.136..-0.144`) above frames.
    3. **Outer Temple & Sideburns Locks (`is_sideburns` or `mid_z < 0.085`):** Scale width `~0.72x`, keep long (`y ≈ -0.238..-0.252`), and smoothly ramp `|x| >= 0.080 - 0.006*frac` below `y = -0.020` (`/ 0.110` smoothstep) so ribbons frame the outer cheeks cleanly with zero eye/lens collision.
- **Rear Bob Dome vs Crown Cracks:** Set `volume_boost=1.01..1.02`, `z_volume_boost=1.04`, `back_z_tuck=1.01`, `x_scale=0.96` (`>1.15` puffs into a balloon; `<=1.0` exposes scalp facets).
- **Rigid Hairclip / Angular Stroke Width Curves (`HairAhoge.Common.f8`):** Stock Ahoge width curves taper toward root (`0.09..0.24`) and tip (`0.0`). When building rigid angular clips (`keep_taper=True`), assign a flat `hairWidthCurveId` (`c3333333333343338333333333333333`) and update the actual material's `Color` / `ShadeColor` / `OutlineColor` UUIDs (`TransferableType.N00.HairAhoge` secondary `Material`) rather than guessing default hair slots.

---

## 3. Skin, Face, Eyes & Eyebrow Shader/Texture Schemas
- **Unified Skin Shading (`SkinGradationMapModifier`):** Both `BodySkin` and `FaceSkin` grayscale textures map `[0..255]` through Shadow/Main/Highlight Skin Colors (`Common.f2`), so dark strokes on `FaceSkin` fade to shadow skin; paint full-RGBA lineart (`lips/smile`) onto `TransferableType.N00.Cheek` (`1024x1024`) after premultiplying blush by its original color and resetting Cheek Color `2bee0407` to `(1,1,1,1)`.
- **`BodySkin` (`N00_000_00_Body`, `2048x2048`):**
  - `PNG[0]` = EditableImage base (`y` normalized gently around light warm peach `[253,232,218]`, compressing luminance `[185..255] -> [0.35..1.0]` to erase faint arm rectangle blemishes).
  - **`PNG[1]` = Hidden Brown Shaved-Scalp Undercut:** Contains `~183k` dark-brown pixels (`#3E2426`) at **`x=766..1281, y=1594..2039`**. Always blank `PNG[1]` and `PNG[2]` completely to transparent `(0,0,0,0)` (never clear only `y < 340` and never touch `PNG[3]` normal map `[127,127,254]`).
- **`BaseHair` (`TransferableType.N00.BaseHair`, `1024x1024`):** Fade nape early (`nape_fade = np.clip((0.66 - y) / 0.10, 0.0, 1.0)`) so cap never pokes below mid-neck bob tips.
- **`FaceSkin` (`1024x1024`):** Lift grayscale outside central face box (`yy in 240..820, xx in 240..784`) up to midtone (`~162..190`) to remove behind-ear/nape dark scalp shading.
- **Dual-Layer Color Lookups:**
  - `MouthTexture`: Search explicitly by `b"Transferable.MouthTexture"` (not `.material`, which matches `BaseCollection` `top[2]`).
  - `EyebrowTexture`: Update both PNG alpha (vertical erosion `brow_h // 6` for slim arches) AND Common colors `bcc26d8c` (main) + `f201924a` (outline).

---

## 4. Outfit & Accessory UV Islands (`2048x2048` unless noted)

### `InnerTop` (`N00/003` on `N00_Body_Skin` UV) — Boat-Scoop Tee + Pixiv 3D-Projected Straps
- **Hand/Palm/Thumb Exclusion Zone:** Tapered Hand UV island spans `x=628..845 & 1203..1420` at `y<=360`, narrowing diagonally along `hand_r_l = 835 - 1.45*(yy - 360)` down to `y=416`. Always apply `~hand_uv_hole` and keep `short_sleeves` strictly on upper arm (`yy=12..165, xx=65..435 | 1613..1983`) so tee paint never specks thumb-index webs.
- **Short-Sleeve Tee Mask:** Keep `(orig_a > 20)`, torso `yy=408..1035, xx=484..1564` (masked by `~hand_uv_hole`), upper-arm sleeves (`yy=12..165`), front shoulders (`xx=820..1228`, widened `775..1273` for `yy>=360` with `~hand_uv_hole`), and back shoulders (`xx=484..632 | 1416..1564`, `yy>=245`).
- **Widened Boat-Scoop Cutouts:** Front `u_collar = ((xx-1024)/196)^2 + ((yy-198)/122)^2 < 1.0`; Back `u_back_l` centered at `x=484, y=235` (`188, 158`, strictly guarded `484 <= xx < 1024`) and mirrored `u_back_r` at `x=1564`.
- **Seamless 3D-Projected Bra Straps:** Extract alpha (`alpha > 40`, `yy <= 435, xx=550..1498`) from stock `Black Bralette Red Straps` (whose native shoulder seam bridges Front `x=849..882, y=205..220` <-> Back `x=620..653, y=260..280`), erode 3px (`ImageFilter.MinFilter(7)`) for border + fill, and composite **under** `white_mask` so straps glide unbroken across the shoulder ridge and tuck behind the boat-scoop rim.

### `Tops#39` (`Blue Zip-Up Track Jacket`, `N00/121`) — Off-Shoulder Jacket & Hood-Collar Drape
- **Stand-Up Collar vs `y=426` Shoulder Stitch:**
  - Vertical stand-up neck flap sits at `y=340..423, x=756..1292`. Zero **ONLY `yy < 424`** (`is_collar_flap`).
  - **CRITICAL:** Front shoulder (`x=752..935, y=426`) and Back shoulder (`x=505..672, y=426`) stitch together in 3D along **`y=423..426`**. Never crop `yy < 474` across those shoulder spans or a transparent white shoulder gap will open.
  - Scoop only center back spine (`y_cut_back = 423 + 38 * clip(1 - dist_spine/118, 0, 1)^1.5`) and chest front (`t_open = clip((715-yy)/(715-423), 0, 1)`, `open_hw = 88 + 54*t_open^1.10`, leaving `x=752..882` stitched at `y=423`). Start zipper teeth at `zy >= 434`.
- **Charcoal Collar Drape & Elliptical Sleeve Caps:** Paint `#383F45` across front shoulders (`yy <= 562`), deeper back collar/hood drape (`yy <= 595 - 35*clip((dist_spine-80)/200, 0, 1)`), and curved sleeve caps (`ell_l = ((xx-812)/88)^2 + ((yy-215)/78)^2 <= 1, yy=136..294` mirrored at `1236`) — keeps armpits (`yy < 125 | yy > 305`) completely clear! Add subtle fold crease (`#22282E`) along lower collar edges.
- **3D Pockets & Track-Jacket Floats + Open-Front `SkinAlphaMask`:** Built-in 3D welt pocket slits sit at `L (786,896)->(758,1044)`, `R (1262,896)->(1290,1044)`. Key `Tops#39` blendshape float UUIDs (`Common.f3`): `6b518dba` (`AllVolume`), `4abff89a` (`NeckFlat` / Shoulder Slope), `1aae0b61` (`ShoulderCrees`), `948e1547` (`ArmFat`), `2fa2f68a` (`WaistFlat`), `be856824` (`WestThin04` / Widen Sag), `0a58f6c7` (`UnderLong` — lowers bottom hem), `d3ef2fa7` (`UnderStretch` — stretches hem downward), `db2bed20` (`UnderWide` — hip clearance), `1936bcb3` (`UnderTight`).
  - **Open-Front `SkinAlphaMask` & Bilinear Fringe Rule:** Stock `Tops#39` `SkinAlphaMask` culls `my=420..675` across the entire chest (`0,0,0`). When converting to an open-front jacket, un-hide `(my <= 710)` inside the placket using the torso circumference ratio `Body_hw / Tops_hw = 536.5 / 640.0 = 0.838` (`open_hw_body = open_hw_tops * 0.96 + 34.0` for clean 3/4 overlap while keeping outer bust culled behind aqua panels), and pre-fill zero-alpha `collar_bleed_zone` (`y=380..620, x=380..1668`) with charcoal RGB to eliminate white/aqua GPU bilinear fringe dots.

### `Tops#66`, `Bottoms#26`, `Socks`, `Shoes`, `GlassesLow`
- **`Tops#66` (`White Pullover Hoodie`):** Closed pullover with baked drawstring/armhole MToon wireframe ghosts if converted open-front; if keeping pullover hood, cut hood `Island 0` (`y=12..624`) along `hood_cut_y = 425 - 53*cos(clip(dist_center/260,0,1)*pi/2)` + `Island 3` (`yy=580..930`) and replace stock shading with cylindrical cosine shading.
- **`Bottoms#26` (`1024x1024`):** Set `Ankle Hem (Raise)` (`dfbd1f402e54cae6b859407ab50c8e7fe9245901`) = `50.0` for tailored mid-thigh shorts without sawtooth crop fringes. Upper elastic waistband spans `y=0..84` (`x=62..962`): painting `y <= 74 + 5*cos(...)` off-white with a stitched lower rim and cast shadow underneath turns high-waisted shorts into a natural untucked T-shirt overhang over the shorts.
- **`Socks` & `Shoes`:** Pair `Body/Whole Body/Foot Size` (`0.18`) + `Outfit/Shoes/Sole (Higher)` (`16`) with `Outfit/Socks` preset **`#24 White Mid-Calf Socks`** for smooth crew volume (if using `#3 Pink Cuffed Socks`, note 3 nodes: cuff `d1c35036`, ankle mesh `1.Additional`, and calf stickers to blank).
- **`GlassesLow` (`512x512`):** Keep `FrameThinly <= 15.0`, preserve outer hinges `(x=0..62 & 450..512, y=158..196)` at `alpha=255`, slim temples to `y=21..37`, and trim upper corners `y=114..172, x=0..46 & 466..512`.
