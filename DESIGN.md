# FlowPlan 3D — App Design Document

## 1. Product Vision
**FlowPlan 3D** is a field-ready mobile and web application built for plumbers, sewer contractors, and building inspectors. It converts a live video walkthrough of a multi-unit, multi-story building into an interactive **2D blueprint and 3D X-Ray schematic**—complete with fixture outlines (toilets, sinks, tubs, cleanouts), room dimensions, pinned site photos, and a **3D sewer/plumbing drawing studio** viewable from any angle.

---

## 2. Core Field Workflow

### Phase 1: Exterior Building Walk-Around
- User launches live rear-camera video and walks the building perimeter.
- User marks the **building footprint** (width × depth), building name/address, and **main sewer exit side** (street, alley, left, right).
- Photos of exterior cleanouts, grade slopes, or tie-in points are pinned directly to the exterior perimeter.

### Phase 2: Unit Entry & Multi-Story Grouping
- As the user enters each unit (e.g., `Unit 101`, `Unit 102`, `Unit 201`, `Basement / Mechanical`), they mark the unit and assign its **floor elevation** (`Basement`, `1st Floor`, `2nd Floor`, `3rd Floor`) and **building side/wing**.
- Each unit receives a distinct architectural border color so stacked wet walls (upstairs bath directly over downstairs bath) are immediately recognizable.

### Phase 3: Live Room Tagging & Fixture Outlines
- Walking through each doorway, the user taps one-hand room tags:
  - **Bathroom** → auto-places schematic `toilet`, `sink`, and `tub/shower` outlines.
  - **Kitchen** → auto-places schematic `kitchen sink` outline.
  - **Closet** → places compact closet outline (useful for spotting chase walls/vertical stack cavities).
  - **Laundry / Utility** → auto-places `water heater`, `floor drain`, and `cleanout`.
  - **Living / Bedroom / Hallway** → open room outlines with dimensions.
- User can drop additional fixtures (`+Toilet`, `+Sink`, `+Tub`, `+Drain`, `+Cleanout`, `+Heater`) and snap **spatially pinned job-site photos** (via live video capture or native phone camera fallback).
- A **live 2D schematic mini-map** updates in real time on the camera HUD.

### Phase 4: 3D/2D Blueprint & Sewer System Studio ("Any Angle")
- **View Modes:**
  - **2D Blueprint:** Top-down architectural plan showing unit labels, room types, dimensions, and fixture outlines.
  - **3D X-Ray Mode:** Adjustable wall opacity (`0%–100%`) so users can orbit 360° around or underneath the slab to inspect multi-story fixture alignment.
  - **Isometric Riser & Side Cutaway:** Instant plumbing riser and elevation angles.
- **Interactive 3D Pipe Drawing:**
  - Magnetic snapping directly onto detected fixture drains, existing pipe junctions, and the exterior main sewer exit.
  - Axis-locked horizontal drawing planes (`Under-Slab Trench`, `Basement`, `1st/2nd/3rd Floor Subfloor`).
  - One-tap **Vertical Stack Drops** (`Drop 1 Floor`, `Drop to Trench`) and **Vertical Vent Risers** (`Rise Vent +10'`).
  - System color-coding: **Sanitary Sewer** (Green), **Vent Stack** (Orange), **Storm Drain** (Purple), **Water Supply** (Blue) with selectable diameters (`3/4"` to `4"`).
- **Deliverables & Takeoffs:**
  - Real-time linear-foot material takeoff grouped by pipe size and system.
  - Interactive pinned photo markers with editable field notes.
  - Exportable **PNG Blueprint snapshots** from any angle and **JSON project files** for backup/sharing.

---

## 3. System Architecture

### Current Release (v1.0 — GitHub Pages Mobile Web App / PWA)
- **Frontend:** Single-file HTML5 + CSS3 + ES Module JavaScript (zero build step, instant HTTPS deployment on GitHub Pages).
- **Video & Photo Capture:** WebRTC `MediaDevices.getUserMedia` (rear camera default + front flip) paired with `<input type="file" capture="environment">` native camera fallback.
- **3D & 2D Engine:** **Three.js** (`OrbitControls`, procedural 2D canvas floor textures, 3D parametric fixtures, raycasted magnetic snapping, cylindrical 3D pipe extrusion).
- **Persistence:** Automatic local browser persistence (`localStorage`) + portable `.json` export/import.

### Future Roadmap (v2.0 — Automated Spatial Scanning & Cloud Sync)
1. **Tier 1 Automated Live-Video Scanning (iOS LiDAR):**
   - Wrap the 3D Studio in a native SwiftUI / Capacitor shell using **Apple ARKit + RoomPlan API** to measure room widths/depths and auto-detect toilets, sinks, tubs, and stairs directly from live video without manual entry.
2. **Tier 2 Pure-RGB Video Reconstruction (Android & Non-Pro Phones):**
   - Capture live video + 6DoF ARCore/ARKit pose keyframes and run cloud multi-view depth reconstruction + semantic fixture segmentation (CubiCasa-style) to output the exact same `FlowPlan 3D` JSON schema.
3. **Pro Plumbing Features:**
   - Automatic sewer grade/slope calculator (`1/8"` and `1/4"` per foot drop verification from farthest fixture to street invert).
   - Drag-and-drop room/fixture repositioning directly in 2D Blueprint view.
   - Multi-page PDF submittal package (Floor Plans + Isometric Riser + Numbered Photo Report + Material Takeoff) and DXF/DWG CAD export.

---

## 4. Core Data Model (`JSON`)

```json
{
  "building": {
    "name": "Oak Ridge 2-Story Duplex",
    "width": 32,
    "depth": 26,
    "sewerExitSide": "south",
    "marked": true
  },
  "units": [
    { "id": "u_101", "name": "Unit 101 (Downstairs)", "floor": 0, "zone": "left", "color": "#38bdf8" }
  ],
  "rooms": [
    {
      "id": "r_101_bath",
      "unitId": "u_101",
      "floor": 0,
      "type": "Bathroom",
      "x": -9, "z": -6, "w": 9, "d": 8,
      "fixtures": [
        { "id": "f1", "type": "toilet", "rx": -2.2, "rz": -2.2, "rot": 0 }
      ]
    }
  ],
  "photos": [
    { "id": "ph_1", "floor": 0, "x": -9, "y": 4.5, "z": -6, "dataUrl": "data:image/jpeg;base64,...", "note": "Existing cast iron flange" }
  ],
  "pipeNodes": [
    { "id": "n1", "x": -11.2, "y": 10.05, "z": -8.2 }
  ],
  "pipes": [
    { "id": "p1", "startId": "n1", "endId": "n2", "system": "sewer", "diameter": 3 }
  ]
}