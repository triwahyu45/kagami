---
name: KAGAMI
description: 3D Character & Cinema Rig suite combining VRoid Studio binary Protobuf/UV/BrushHair customization (.vroid/.vroidcustomitem) with Three.js film cinematography grammar (Dolly Zoom, Orbital Arc, Pedestal, Micro-Breath) and VRM 0.0 vs VRM 1.0 anatomical bone polarity & joint clamp guardrails.
---

# Kagami — 3D Character & Cinema Rig

Unites **VRoid binary/UV engineering** (`vroid-binary-forge`) with **film camera grammar and VRM anatomical bone guardrails** (`cinematic-3d-camera`) so 3D characters look custom-crafted and move naturally without inverted knees or elbows.

---

## Sub-Modules

- **VRoid Protobuf & UV Schemas**: [`references/vroid-protobuf-and-uv.md`](references/vroid-protobuf-and-uv.md)
- **Close-Up Visual Audit & Overlay**: [`references/vroid-closeup-audit.md`](references/vroid-closeup-audit.md)
- **Film Camera Grammar & VRM 0.0/1.0 Clamps**: [`references/cinematic-camera-and-vrm-clamps.md`](references/cinematic-camera-and-vrm-clamps.md)
---

## Mandatory Self-Audit & Revise-Before-Delivery Gate

Before delivering any `.vroid` edit or Three.js VRM scene:
1. Run multi-angle close-up screenshots (`vroid-studio` MCP `batch`/`screenshot` or `--precheck` keyframe PNGs) and inspect via `read_file`.
2. Confirm zero inverted knees/elbows (VRM 0.0 vs 1.0 polarity), zero UV clipping, and natural head/neck bounds; fix and re-verify first.
