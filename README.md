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

**Option A — keep it in the repo (recommended).** Skip to step 3 and point Maya at
the repo path.

**Option B — make it importable from anywhere.** Copy `quadwrap.py` into your Maya
scripts folder so `import quadwrap` always works:

```bash
# Windows
copy quadwrap.py "%USERPROFILE%\Documents\maya\2027\scripts\"
```

(Maya auto-loads anything in that folder at startup. On macOS the path is
`~/Library/Preferences/Autodesk/maya/2027/scripts/`.)

### 3. Run it

1. Open **Maya 2027**.
2. Open the **Script Editor** (`Window > General Editors > Script Editor`) and
   click the **Python** tab.
3. Paste this and press Enter:

   ```python
   import sys
   sys.path.append(r"C:\path\to\pfa-week06")   # <- your clone path
   import quadwrap
   quadwrap.show_ui()
   ```

   The **QuadWrap** window opens.

> If you used Option B you can drop the first two lines and just run
> `import quadwrap; quadwrap.show_ui()`.

### 4. Try it on the included assets (60 seconds)

1. `File > Import…` → `TARGET_MESH.fbx`  (the sculpt — 5,077 faces)
2. `File > Import…` → `CAGE_MESH.fbx`    (a blockout cage — 400 faces)
3. Select **TARGET_MESH** in the viewport → click **Load Target Mesh**.
   The label should now read `target: TARGET_MESH`.
4. Select **CAGE_MESH** → click **Fit to Surface**.
   The cage snaps down onto the sculpt, live.
5. Drag **Projection Strength** left and right — the cage follows in real time.
6. **Preview** → a ghost/reference copy of the fitted cage.
7. **Commit** → bakes it to real geometry in one undo step.
8. **Heat Map (deviation)** → colours the cage green→red by how far it is from
   the sculpt.
9. **Deform Test (skin)** → binds the result to two test joints and bends them
   so you can judge whether the loops hold up.

### 5. Optional — verify it without opening the UI

`mayapy` is Maya's bundled Python and can run the tool headlessly:

```bash
"C:\Program Files\Autodesk\Maya2027\bin\mayapy.exe" -c "
import sys; sys.path.append(r'C:\path\to\pfa-week06')
import quadwrap as q
print('import ok; cmds available:', q.cmds is not None)
"
```

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
| **Guide strips** | ✅ | Select 2+ vertices **in order** → **Draw Strip** makes a curve along them. **Close Loop**, **Auto-Ring** (radial duplicates) and **Mirror** all work. |
| **Presets** | ✅ | Prop / Character / Hard-Surface, plus **Save Current Settings** for custom presets. |
| **Settings persistence** | ✅ | Saved to `~/.quadwrap/settings.json` and reloaded on launch. |
| **Bad-input handling** | ✅ | Bad selections, missing targets and empty selections produce a warning, never a crash. |
| **Undo Last / Clear All** | ✅ | Only ever deletes nodes the tool itself created — never wildcards. |

## What doesn't work yet

Honest list of the gaps:

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
- **No undo of a committed mesh via Maya's own undo stack** — "Undo Last" is the
  tool's own action stack, not `Ctrl+Z`.

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

> **TODO — fill this in before submitting.** Replace this block with the name of
> the classmate you ran the tool with and their three sentences. Keep it in
> their own words.

**Classmate:** _name here_

1. _sentence one_
2. _sentence two_
3. _sentence three_
