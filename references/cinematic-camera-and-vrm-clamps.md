# Cinematic 3D Camera & Anatomical VRM Guardrails

## Scope & Trigger Context
- **Triggers**: "cinematic camera", "dolly zoom", "wide establishing", "close up", "arc orbit", "VRM 0.0 leg/arm polarity", "joint clamp", "anatomical physics".

## 1. Film Camera Vocabulary (Three.js)
Never lock a narrative short to a static FOV or repetitive zoom-in. Combine distinct cinematic beats with `smoothstep(t)` easing and subtle organic handheld micro-sway (`Math.sin(t * 0.85) * 0.006`):
- **Wide Establishing + Pedestal Down**: Start `fov 48–52`, high `y ~ 2.15m` -> settle `y ~ 1.48m` while dollying forward so viewers absorb the full environment first.
- **Dolly Zoom (Vertigo Effect)**: Move camera forward (`z: 4.6 -> 2.0`) while narrowing `fov` (`50 -> 29`) simultaneously so subject size holds while background perspective compresses.
- **Low-Angle Medium Truck -> Close-Up**: Camera below eye-level (`y ~ 1.06–1.12m`) tracking laterally toward subject, tightening `fov` (`35 -> 27`).
- **Orbital Arc**: Orbit camera around subject center `[cx, cy, cz]` via `[cx + Math.cos(angle) * r, y, cz + Math.sin(angle) * r]` (`angle` sweeping `12°–20°`) for contemplative side profiles.
- **Wide Dolly-Out**: Pull camera back (`z: 3.8 -> 6.0`) while widening `fov` (`38 -> 48`) for emotional release / group endings.

## 2. VRM 0.0 vs VRM 1.0 Anatomical Polarity & Joint Clamps
When mixing VRM 1.0 (e.g. `custom_model.vrm`) and `VroidVrmExporter` VRM 0.0 models with `facingOffset = Math.PI`:
- **Seated Legs (`sit_chair`)**: VRM 1.0 uses `upperLeg.x = -1.44, lowerLeg.x = +1.46`. VRM 0.0 has **inverted X leg polarity** — explicitly set `upperLeg.x = +1.44, lowerLeg.x = -1.46` so thighs extend forward over benches and knees bend downward (never backward!).
- **Seated Arms & Elbows**:
  - VRM 1.0: `leftUpperArm.rotation.set(-0.28, 0, -1.16)`, `leftLowerArm.rotation.set(0, -0.52, 0)`, `rightUpperArm.rotation.set(-0.28, 0, 1.16)`, `rightLowerArm.rotation.set(0, 0.58, 0)`.
  - VRM 0.0 (inverted local X & Z after `rotateVRM0`): `leftUpperArm.rotation.set(0.28, 0, 1.16)`, `leftLowerArm.rotation.set(0, -0.52, 0)`, `rightUpperArm.rotation.set(0.28, 0, -1.16)`, `rightLowerArm.rotation.set(0, 0.58, 0)`. Never apply negative `upperArm.x` or positive `leftLowerArm.y` (`+= 0.65`) on VRM 0.0 or elbows will bend backwards/outwards! Always verify via `--precheck` keyframe PNGs before full render.
- **Head/Neck Bow ("Nunduk")**: Always **assign** (not accumulate `+=`) bounded angles and run a biological clamp pass:
  - `head.x` in `[-0.36, 0.34]`, `head.y` in `[-0.65, 0.65]`, `head.z` in `[-0.28, 0.28]`
  - `neck.x` in `[-0.22, 0.22]`, `neck.y` in `[-0.35, 0.35]`
  - `chest.x` in `[-0.16, 0.16]`

---

## 3. Rainy-Night Grading, SpotLight Pitfall & World-Space 2-Bone Arm IK Props

- **VRM 1.0 vs VRM 0.0 `MToonMaterial` Brightness**: `@pixiv/three-vrm@3.3.3` VRM 1.0 renders ~π× brighter than VRM 0.0 under PBR lights. Multiply `mat.color` (or `uniforms.litFactor.value` if `mat.color` is absent) by `0.45` for VRM 1.0 and `0.92` for VRM 0.0, tint `shadeColorFactor` with moody slate/indigo (`0x4E6288`), and keep `toneMappingExposure ~ 0.96`.
- **`SpotLight` `castShadow` Shader Error**: On Three.js `r170` + `@pixiv/three-vrm@3.3.3` `MToonMaterial`, `SpotLight` with `castShadow = true` throws `ERROR: 'vSpotShadowCoord' : undeclared identifier`. Use `PointLight` or `DirectionalLight` for shadow-casting keys.
- **Readable Smartphone Props + OTS Insert Shots**: Place smartphones in world space with a `CanvasTexture` screen + back triple-camera island + subtle cyan `PointLight`, cradle with `solveTwoBoneArmIK`, curl finger bones (`Proximal/Intermediate/Distal.z` + tucked `Thumb`), and lock prop position `4.6 cm` from `rightMiddleProximal` toward `headWorldPos` so fingers never poke through the screen. Frame an Over-the-Shoulder (OTS) insert (`fov 29–32`, ~0.45m away) to show hands + readable UI together.
- **Zero-Hair-Whip Stand-Up (`springBoneManager`)**: NEVER call `engine.setState('idle')` while seated in procedural `sit_sofa` — the 1-frame `hips.y` teleport into `idle.vrma` violently whips hair/skirt springbones. Instead, keep `sit_sofa` continuous, damp `springBoneManager.joints` (`dragForce >= 0.72`, `gravityPower <= 0.055`), and smoothly interpolate `surfaceHeight` (`0.53 -> 0.95`) and `sitBlend` (`1 -> 0`) over `1.20s` with a `Math.sin(standU * Math.PI)` forward torso lean.
