# How To Use The Eye Model (Simple Guide)

This guide walks you through making your eye model, saving it as a file, and
bringing it into COMSOL together with your microstent. No experience needed —
just follow the steps in order.

---

## 1. What you're working with

Think of the eye model like a hollow ball made of two layers glued together:

- **Cornea** — the clear, curved "window" at the very front (colored light blue in the tool).
- **Limbus** — the thin ring where the clear cornea meets the white sclera (colored orange).
- **Sclera** — the white, tougher shell that makes up the rest of the eyeball (colored off-white).
- **Schlemm's canal** — a thin ring-shaped tube buried right at the limbus, where fluid drains out and where your stent sits (colored red).
- **Insertion guide** — a small red ball-and-stick marker showing exactly where and in what direction the stent should go. This part is just a guide — delete it before you run your simulation.

All three parts are built in **millimeters**, the same units your microstent
tool uses, so everything lines up without any extra conversion.

---

## 2. Make your eye model files

1. Open the **Eye Globe Model Studio** tool.
2. Move the sliders on the left until the numbers in the "derived" box match
   the eye you want to study (the defaults already match a normal adult eye,
   so you can skip this step if you just want to get started).
3. Click the three download buttons, one at a time:
   - **Download eye globe STL** → saves `eye_globe_cornea_limbus_sclera.stl`
   - **Download Schlemm's canal STL** → saves `schlemms_canal.stl`
   - **Download insertion guide STL** → saves `stent_insertion_guide.stl`
4. Also export your stent from the **Microstent Geometry Studio** tool the
   same way you already do (the "Download edited STL" button).

You should now have four `.stl` files. Put them all in one folder so they're
easy to find.

---

## 3. Bring the files into COMSOL

1. Open **COMSOL Multiphysics** and start a new model (or open your existing
   glaucoma model).
2. In the **Model Builder** tree, right-click **Geometry** → **Import**.
3. In the Settings window, click **Browse** and pick `eye_globe_cornea_limbus_sclera.stl`.
   Set the **Length unit** to `mm`.
4. Click **Import**. The eye shape should appear in the graphics window.
5. Repeat steps 2–4 for `schlemms_canal.stl` — this adds the canal as a second
   part in the same geometry sequence.
6. Right after each Import node, add a **Repair** step (right-click the
   Import node → this is usually automatic, but if the shape looks broken or
   COMSOL warns about the geometry, add **Geometry → Repair** or use
   **Geometry → Tessellated Surface → Deflection Faces/Edges** to clean it up).
7. Click **Build All** at the bottom of the Geometry settings. If it finishes
   without a red error icon, your eye geometry is in.

**Tip:** If COMSOL says the imported STL is "not a closed surface" or similar,
just re-run the Repair step with a slightly looser tolerance — the models
from this tool are built to be watertight, but STL files are made of flat
triangles, so COMSOL sometimes wants to smooth tiny seams automatically.

---

## 4. Add your stent in the right spot

This is the part that used to be tricky — the eye model tool now does the
math for you. Open the eye model tool and look at the derived box for these
three lines:

- **Entry point (mm)** — where the stent should start.
- **Insertion axis (unit vec)** — which direction the stent should point.
- **Tip point (mm)** — where the tip of the stent should end up.

To place the stent:

1. Import your stent STL the same way you imported the eye (**Geometry →
   Import**, unit = `mm`).
2. Right-click the stent's Import node and add a **Move** transform (found
   under **Geometry → Transforms**).
3. In the Move settings, type the **Entry point** coordinates into the
   Displacement fields (this slides the stent to the right spot).
4. Add a **Rotate** transform above the Move so the stent's long axis lines
   up with the **Insertion axis** vector. If your stent was drawn pointing
   along the x-axis in the microstent tool, rotate it so that axis now points
   along the insertion axis vector.
5. Click **Build All** and rotate the 3D view to check that the stent tip
   sits inside the red canal ring. Nudge the Move/Rotate numbers slightly if
   it's not quite touching.
6. Once it looks right, you can delete the insertion-guide part — it was
   only there to help you aim.

---

## 5. Tell COMSOL what material each part is

The research plan uses two material models:

| Part | Material model |
|---|---|
| Cornea + Limbus | Yeoh hyperelastic |
| Sclera | Neo-Hookean hyperelastic |

Because the eye is exported as **one connected shell**, you'll select the
cornea/limbus region by hand:

1. Under **Component → Materials**, add two blank materials and rename them
   `Cornea-Limbus` and `Sclera`.
2. Under **Definitions**, add a **Sphere selection** (or **Box selection**)
   centered on the cornea (roughly a sphere of radius equal to your corneal
   diameter, centered at the corneal apex). This grabs the cornea + limbus
   domain/boundary automatically.
3. Assign the `Cornea-Limbus` material to that selection, and the `Sclera`
   material to everything else (use "All domains" then subtract the
   selection, or just assign Sclera first and override with Cornea-Limbus).
4. Set the material models in **Solid Mechanics**: Yeoh for Cornea-Limbus,
   Neo-Hookean for Sclera, matching the values from your paper.

---

## 6. Common problems and quick fixes

- **"Geometry is not closed" error** → Re-run Repair with a larger tolerance,
  or increase the mesh resolution in the eye tool (more segments = smoother
  seams) and re-export.
- **Stent floats outside the eye** → Double-check you used the *Entry point*
  and not the *Tip point* for the Move displacement, and that units are mm
  in both imports.
- **Parts overlap oddly** → Turn on transparency for the eye globe
  (right-click the geometry → **Appearance**) so you can see the stent inside
  it while you adjust the Rotate/Move numbers.
- **Mesh fails near the canal** → Lower the canal tube diameter slightly, or
  increase mesh refinement just in that region using a **Mesh → Size**
  feature restricted to the canal boundary/domain.

---

## 7. Quick checklist

- [ ] Exported `eye_globe_cornea_limbus_sclera.stl`
- [ ] Exported `schlemms_canal.stl`
- [ ] Exported `stent_insertion_guide.stl` (for aiming only)
- [ ] Exported your stent STL from the microstent tool
- [ ] Imported eye + canal into COMSOL, units set to mm, geometry builds clean
- [ ] Imported stent, moved to Entry point, rotated to Insertion axis
- [ ] Stent tip visually sits inside the canal ring
- [ ] Deleted the insertion-guide part
- [ ] Assigned Yeoh to cornea/limbus and Neo-Hookean to sclera
- [ ] Ready to mesh and run your FSI/FEA study

That's it — you now have a matched eye + stent geometry, in the right units,
positioned correctly, ready for meshing in COMSOL.
