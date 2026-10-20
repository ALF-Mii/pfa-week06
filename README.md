# pfa-week06 — QuadWrap

**QuadWrap** is a semi-automatic retopology tool for Maya. It fits a clean quad cage
onto a high-poly sculpt by projecting it with Maya's own Shrink Wrap deformer, so
you spend your time on the loops that matter instead of nudging vertices by hand.

It shows the fitted cage as a ghost mesh before you commit it, colours the result
with a deviation heat map so you can see exactly where the cage drifted off the
sculpt, and can skin the result to test joints so you know the loops hold up
under deformation.

---

## Which app does this live in?

**Maya.** This is a Maya Python tool, not a standalone app.

| | |
|---|---|
| **Host app** | Maya 2027 (Windows). Written against `maya.cmds` and `maya.api.OpenMaya` (API 2.0). Should run on Maya 2022+ unchanged. |
| **Installs as** | One file. `quadwrap.py` is completely self-contained — no `pip install`, no packages, no add-in to compile. |
| **Standalone?** | **No.** Every feature (fit, preview, commit, heat map, deform test) needs a live Maya session. The file *does* import cleanly under a plain Python interpreter — it has a guard that sets `cmds = None` — but the tool functions then just raise a clear error. Don't expect it to do anything outside Maya. |
| **Needs Maya UI?** | The window needs Maya. The underlying functions (`fit_to_surface()`, `heat_map()`, `deform_test()`, …) can be driven headlessly with `mayapy`, which is how the tool is tested. |

---

## Install and run

A classmate should be able to get this working from this README alone.

### 1. Get the files

```bash
git clone https://github.com/ALF-Mii/pfa-week06.git
```

Or just download `quadwrap.py` — it is the only file you strictly need. The repo
also ships two test meshes: `TARGET_MESH.fbx` (the sculpt) and `CAGE_MESH.fbx`
(a blockout cage).

### 2. Install

**Option A — put it in Maya's scripts folder (recommended).** After this,
`import quadwrap` works from anywhere and you never type a path again.

1. Open Maya. In the **Script Editor** (Python tab) run this to find the folder
   Maya auto-loads scripts from:

   ```python
   import maya.cmds as cmds
   print(cmds.internalVar(userScriptDir=True))
   ```

   It prints a path like `C:/Users/You/Documents/maya/2027/scripts/`.

2. Copy `quadwrap.py` from the repo into that folder — in File Explorer, paste
   the printed path into the address bar, then drag `quadwrap.py` in.
3. **Restart Maya** so it picks the file up.

**Option B — leave it in the repo.** You will need the repo's full path in step 3.
To find it: in File Explorer, open the `pfa-week06` folder, click once in the
address bar so the full path shows up, and copy it.

> On macOS the scripts folder is `~/Library/Preferences/Autodesk/maya/2027/scripts/`
> and `cmds.internalVar(userScriptDir=True)` prints it for you the same way.

### 3. Run it

1. Open **Maya 2027**.
2. Open the **Script Editor** (`Window > General Editors > Script Editor`) and
   click the **Python** tab.
3. Paste the matching command below and press **Ctrl+Enter**.

   **If you used Option A** (installed into the scripts folder):

   ```python
   import quadwrap
   quadwrap.show_ui()
   ```

   **If you used Option B** (still in the repo) — use forward slashes, and swap
   in your own path:

   ```python
   import sys
   sys.path.append(r"C:/Users/You/Documents/pfa-week06")
   import quadwrap
   quadwrap.show_ui()
   ```

   The **QuadWrap** window opens.

### 4. Try it on the included assets (60 seconds)

**Load the two meshes.** Either use the menu — `File > Import…`, pick
`TARGET_MESH.fbx`, then repeat for `CAGE_MESH.fbx` — or paste this into the
Script Editor (change `repo` to your own folder):

```python
import maya.cmds as cmds
repo = r"C:/Users/You/Documents/pfa-week06"
cmds.loadPlugin("fbxmaya", quiet=True)      # needed before type="FBX"
cmds.file(repo + "/TARGET_MESH.fbx", i=True, type="FBX")
cmds.file(repo + "/CAGE_MESH.fbx", i=True, type="FBX")
```

> The `loadPlugin` line matters. Maya does not always have the FBX plugin
> loaded, and without it the import fails with
> `Invalid file type specified: FBX`. The `File > Import…` menu route loads it
> for you.

**Then drive the tool.** The window is numbered — follow it top to bottom.

1. Select **TARGET_MESH** in the viewport → click **Load Target Mesh**.
   The label should now read `target: TARGET_MESH`.
2. In **2 Cage**, leave the dropdown on **Prop** and click **Create Base Cage**.
   The tool makes a starting cage sized to your sculpt — you don't need a
   cage of your own to get started. (Or select any mesh and click
   **Use Selected as Cage** to use your own.)
3. Click **Fit to Surface**. The cage snaps down onto the sculpt, live.
4. Drag **Projection Strength** left and right — the cage follows in real time.
5. **Preview** → a ghost/reference copy of the fitted cage.
6. **Commit** → bakes it to real geometry in one undo step.
7. **Heat Map (deviation)** → colours the cage green→red by how far it is from
   the sculpt.
8. **Deform Test (skin)** → binds the result to two test joints and bends them
   so you can judge whether the loops hold up.

Expected result: the cage's distance from the sculpt goes from **7.09** to
**0.00**, and the committed mesh still has **400 faces / 382 verts / 0
non-manifold edges**.

### 5. Optional — verify it without opening the UI

`mayapy` is Maya's bundled Python and can run the tool headlessly:

```bash
"C:\Program Files\Autodesk\Maya2027\bin\mayapy.exe" -c "
import sys; sys.path.append(r'C:/Users/You/Documents/pfa-week06')
import quadwrap as q
print('import ok; cmds available:', q.cmds is not None)
"
```

### If something goes wrong

| What you see | Why | Fix |
|---|---|---|
| `ModuleNotFoundError: No module named 'quadwrap'` | Maya's Python can't find the file | You skipped step 2, or you installed it but didn't restart Maya. Restart Maya, or paste the `sys.path.append` line. |
| `Invalid file type specified: FBX` | Maya's FBX plugin isn't loaded | Run `cmds.loadPlugin("fbxmaya")` first, or use `File > Import…` instead. |
| **Fit to Surface** does nothing | Nothing is selected, or the cage and the target are the same mesh | Select the **cage**, then press Fit. The status line at the bottom of the window tells you what it wants. |
| The cage moves but only a little | **Projection Strength** is low | Drag it to 1.0. At 0.5 the cage only travels halfway to the sculpt. |
| The cage snaps somewhere odd | The cage doesn't overlap the sculpt | A shrink wrap projects onto the nearest surface, so the cage has to sit across or around the sculpt — not floating off to one side. |
| No ghost appears after **Preview** | You haven't pressed **Fit to Surface** first | Preview shows the *fitted* cage, so fit first. |
| A button does nothing | Read the status line at the bottom of the window | Missing selections and bad inputs are reported as messages, not errors. |

---

## What works now

Everything below was run and verified against `TARGET_MESH.fbx` + `CAGE_MESH.fbx`
in Maya 2027.

| Feature | Status | Verified result |
|---|---|---|
| **Fit to Surface** | ✅ | Cage max deviation from the sculpt went **7.0922 → 0.0000**. The cage lands exactly on the target surface. |
| **Projection Strength** slider | ✅ | Drives the shrinkWrap `.envelope`. At 0.5 the cage sits halfway to the surface, at 1.0 it is fully on it. Live and non-destructive. |
| **Projection Mode** | ✅ | All 5 real Maya modes work — Toward Inner, Toward Center, Parallel, Vertex Normals, Closest. |
| **Closest If No Intersection** | ✅ | Maps to `.closestIfNoIntersection` — keeps projection working on dense, messy sculpts with holes. |
| **Snap Offset / Tolerance** | ✅ | Maps to the shrinkWrap `.offset`. |
| **Preview (ghost mesh)** | ✅ | Renders the fitted cage as a reference-shaded duplicate so you can tweak before committing. |
| **Commit** | ✅ | Bakes the projected shape to real geometry in **one undo step**. Topology preserved exactly: **400 faces / 382 verts / 0 non-manifold edges**. |
| **Heat map (deviation)** | ✅ | Per-vertex distance to the target surface → green→red vertex colours. Reports the worst drift. |
| **Deform test** | ✅ | Binds the committed cage to two test joints and bends them 45° so you can inspect the loops. |
| **Base cages** | ✅ | Generates a starting cage sized to the loaded target: **Prop** (24 quads), **Hard-Surface** (26, chamfered) and **Human** (136, a real T-pose silhouette). No extra asset files — everything is built in Maya. |
| **Guided workflow** | ✅ | The window is numbered 1–5 with a "Getting started" panel. Fit falls back to the last cage, so you never have to re-pick a mesh. "Use Selected as Cage" accepts your own cage. |
| **Guide strips** | ✅ | Select 2+ vertices **in order** → **Draw Strip** makes a curve through them, every control point exactly on the surface (deviation 0.0000). **Close Loop**, **Auto-Ring** (count from the Density slider) and **Mirror** (X by default, with a working Symmetry Blend) all work. |
| **Presets** | ✅ | Prop / Character / Hard-Surface, plus **Save Current Settings** for custom presets. |
| **Settings persistence** | ✅ | Saved to `~/.quadwrap/settings.json` and reloaded on launch. |
| **Bad-input handling** | ✅ | Bad selections, missing targets and empty selections produce a warning, never a crash. |
| **Undo Last / Clear All** | ✅ | Only ever deletes nodes the tool itself created — never wildcards. `Clear All` also removes the deform-test rig (joints + skin) so nothing is left behind. |
| **Ctrl+Z after a commit** | ✅ | Commit is wrapped in a single undo chunk, so one `Ctrl+Z` removes the whole commit. The tool also prunes its own registry when Maya's undo removes a node, so the stats never point at a mesh that is gone. |

## What doesn't work yet

Honest list of the gaps:

- **Base cages are blockouts, not finished cages.** They are deliberately
  simple so you can shrink wrap them and then cut them down. There is no
  "build a production cage from the base" yet.
- **No auto-meshing from strips.** Guide strips are *reference curves* that mark
  where loops should go; they are not lofted into a cage for you. You still
  model the cage yourself and use the tool to fit it.
- **No curvature-adaptive density.** The **Density** slider currently only
  scales the Auto-Ring count. It does *not* yet tighten loops where the sculpt
  bends. This was the one piece of instructor feedback still unimplemented.
- **No comparison against Maya's built-in `polyRetopo`.** Maya's automatic
  Retopologize has not been benchmarked against QuadWrap yet, so there is no
  "here's what our tool does better" data for the demo.
- **No freehand drawing.** Strips come from vertex selections only; drawing a
  curve by hand and converting it to a strip is not implemented.
- **No ribbon helper.** Strips are curves, not mesh ribbons.
- **Untested on a production sculpt.** Everything so far has been verified on
  the included blockout pair. It has not been run against a detailed organic
  character sculpt, where the shrinkWrap may behave differently.
- **Auto-Ring counts and axis** are not yet tuned per-asset.

---

## How it works

The risky part of the original pitch — projecting a cage onto a dense, uneven
sculpt — is deliberately **not** hand-written. QuadWrap creates and owns a real
Maya `shrinkWrap` deformer and drives it from the panel:

```
cageShape ──> shrinkWrap1 ──> cageShape
                  │
                  └── .targetGeom <── targetShape.worldMesh[0]
```

That buys live previews, native undo, and no custom projection maths to go
wrong. Because the deformer is Maya's, the cage's original topology is never
touched — Commit bakes it out.

Per-vertex deviation for the heat map is computed with
`maya.api.OpenMaya.MFnMesh.getClosestPoint()` against the target surface.

### Base cages

You do not need a cage to start. The tool can make you one, sized to whatever
target you loaded:

| Base cage | What it is | Quads |
|---|---|---|
| **Prop** | A subdivided cube that encloses the sculpt | 24 |
| **Hard-Surface** | A chamfered box — the right starting shape for panels and props | 26 |
| **Human** | A T-pose blockout: feet, legs, hips, waist, chest, shoulders, neck, head | 136 |

They are generated procedurally, so there are no extra asset files to download
and nothing to install. Each one is a *blockout*: shrink wrap it onto the
sculpt, then cut it down into a proper quad cage by hand. That is the point —
the tool handles the repetitive fitting, you keep the creative decisions.

The **Human** cage is a real silhouette, not a box. Measured width by height:
feet 5.6 → legs 6.2 → hips 7.4 → waist 6.5 → chest 6.9 → **shoulders 9.7** →
neck 3.8 → head 3.9 (in the units of the loaded sculpt).

### Guide strips

A guide strip is a **reference curve** that marks where a loop should go. It is
a degree-1 polyline, so it passes exactly through the points you picked — every
control point sits on the sculpt surface (measured deviation: **0.0000**).

| Button | What it does |
|---|---|
| **Draw Strip** | Select 2+ vertices **in order** on the mesh → makes a curve through their world positions. |
| **Close Loop** | Turns the open strip into a closed loop. |
| **Auto-Ring** | Copies the strip around the mirror axis. The count comes from **Density** (`Density × 4`, minimum 3). |
| **Mirror** | Reflects the strip across the mirror axis (X by default). |

**Symmetry Blend** sets how hard the mirror applies: `0` leaves the strip alone,
`1` is a full reflection, and `0.5` is flat on the mirror plane. That midpoint
is not a bug — a reflection blends *through* its own plane, so half-way
between a shape and its mirror is the flattened shape.

Strips are guides, not geometry: the tool does **not** loft them into a cage.
You use them to plan where the loops go, then model the cage along them and fit
that cage with **Fit to Surface**.

---

## Files

| File | What it is |
|---|---|
| `quadwrap.py` | The tool. Single self-contained file — this is the deliverable. |
| `QUADWRAP_DESIGN.md` | The design document: architecture, the shrink-wrap driver, the guide-strip model, the 3-week plan, and the decisions taken. |
| `maya_setup.py` | Opens a socket port inside Maya so scripts can be run against a live session. |
| `maya_bridge.py` | The client side of that socket. Optional developer tooling. |
| `TARGET_MESH.fbx` | Test sculpt (5,077 faces). |
| `CAGE_MESH.fbx` | Test cage (400 faces). |

---

## Peer run

**Classmate:** Fayth Elisa Navarro

1. "I cloned the repo, ran the Option B lines (sys.path.append + import
   quadwrap + quadwrap.show_ui()) in Maya 2026 on Windows, imported
   TARGET_MESH.fbx and CAGE_MESH.fbx, then went Load Target Mesh → Fit to
   Surface → Projection Strength down to 0.5 and back to 1.0 → Preview → Commit
   → Heat Map → Deform Test → Clear All."
2. "Everything matched your README: the cage went from 7.09 to 0.00 off the
   sculpt (3.55 at strength 0.5, exactly halfway), the committed mesh had 400
   faces / 382 verts / 0 non-manifold edges, and selecting nothing or using the
   target as the cage gave a warning instead of an error. One Ctrl+Z also
   removed the committed mesh, even though 'What doesn't work yet' says Ctrl+Z
   can't undo a commit."
3. "Two things tripped me up: the paste-in import lines in step 4 failed with
   Invalid file type specified: FBX because the FBX plugin wasn't loaded in my
   Maya (running cmds.loadPlugin("fbxmaya") first fixed it), and Clear All left
   qw_test_joint_a behind in the scene after a Deform Test. The Preset dropdown
   is also too narrow, so it only shows 'Pro'."

### What broke, and what I fixed

| What broke | Why | Fix |
|---|---|---|
| The step 4 import lines failed with `Invalid file type specified: FBX` | Maya's FBX plugin is not loaded by default, and `cmds.file(..., type="FBX")` needs it | The README now loads it first with `cmds.loadPlugin("fbxmaya")`. Re-tested on a clean Maya session. |
| `Clear All` left `qw_test_joint_a` behind after a Deform Test | Maya parents a newly created joint under whatever is selected, so the test rig was not really the tool's to delete | The rig is now unparented to the world, registered on the tool's undo stack, and removed by both `Clear All` and `Undo Last`. |
| The Preset dropdown was too narrow (showed only `Pro`) | The menu shared a row with the Save button | The preset menu now gets its own full-width row and the window is wider. `Hard-Surface` fits — checked on screen. |
| This README claimed `Ctrl+Z` could not undo a commit | That list was written from the plan, not from a test | It can — Commit is wrapped in one undo chunk. The claim is corrected above, and the tool now prunes its own registry when Maya's undo removes a node. |

