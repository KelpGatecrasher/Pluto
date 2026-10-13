# Pluto — Arma 3 asset pack builder

Drag 3D models in, get a finished Arma 3 mod out.

## Using it

1. Run **Pluto.exe** (no install needed).
2. At the top, set the **Mod name**, a short **Tag** (class-name prefix such as `MYP`), and your **Author** name.
3. Drag model files, or whole folders, onto the list. You can also drag files onto `Pluto.exe` itself.
4. Click an object to see it in 3D. The coloured wireframes show the collision Pluto generated.
   - Double-click a name (or press F2) to rename it. Class names are generated automatically and stay unique.
   - Select several objects to change their settings all at once in the property panel.
5. Press **Build Mod** (F5).
6. In the Arma 3 Launcher, go to **Mods → … → Add local mod** and pick the `@YourMod` folder.
   Your objects appear in Eden and Zeus under your category.

Save your work as a `.pluto` project (File → Save) to rebuild later.

## What it does for you

| Step | Done automatically |
|---|---|
| Import | FBX, OBJ (+MTL), glTF/GLB, DAE, 3DS, BLEND, STL, PLY, X, LWO, 3MF, USD and ~30 more, via Assimp. Node transforms are baked in. .blend files (any Blender version, incl. compressed) are exported to GLB by the installed Blender in the background and cached. Source engine (HL2 / Garry's Mod) .mdl + .vvd + .vtx with .vmt/.vtf materials are read natively. |
| Units / axes | Converts to Arma's left-handed, Y-up, metre space. Detects centimetre and millimetre files. Z-up formats are rotated. |
| Origin | Pivot goes on the ground under the model's centre (`autocenter=0`). |
| Collision | Convex hulls (QuickHull) for each separate piece, merged down to a component limit and simplified. Written to the Geometry, View Geometry and Fire Geometry LODs (plus Geometry PhysX for physics props), with `ComponentXX` selections and mass. |
| Textures | PNG, JPG, TGA, BMP, PSD (external or embedded) → power-of-two DXT1/DXT5 `.paa` with mipmaps and LZO compression. Normal maps become `_nohq`, and textures with alpha become `_ca`. |
| Materials | A Super-shader `.rvmat` for each material. The fire geometry gets a penetration material (metal, wood, concrete or plastic). |
| Shadows | Shadows come from the visual mesh (`sbsource=visual`). |
| Config | `CfgPatches`, an Eden category and subcategories, `CfgVehicles` (static `NonStrategic`, physics `ThingX`, or building `House_F`), and a rendered `editorPreview` image for each object. |
| Packing | `@Mod/addons/<tag>_<mod>.pbo` with a prefix and SHA1, plus `mod.cpp`. An unpacked source folder (with `$PBOPREFIX$`) is optional, for Addon Builder. |

### Collision modes
- **Auto** – one hull for each disconnected piece of mesh (table legs, crate lid, and so on). This is the default.
- **SingleHull** – one hull around everything.
- **Box** – the bounding box.
- **PerObject** – one hull for each object or mesh in your source file. Use this to control the collision layout from your 3D app.
- **None** – no collision.

## Troubleshooting
- **Wrong size?** Check the *Size (m)* readout and change *Scale*.
- **Lying on its side?** Change *Up axis*. **Facing the wrong way?** Set *Rotation*.
- **Inside-out in game?** Turn on *Flip faces*.
- **Model won't import, with an error about Visual C++?** Install the [VC++ Redistributable x64](https://aka.ms/vs/17/release/vc_redist.x64.exe).
- `.dds` and `.tif` textures aren't read yet. Save them as PNG or TGA next to the model.

## Known limitations (v0.1)
- Models are packed as unbinarized MLOD. Arma loads them fine, but they load a little slower than binarized ODOL. For release builds, tick *Also write unpacked source* and run the result through Arma 3 Tools' Addon Builder.
- Buildings: concave shapes such as rooms need *PerObject* collision with separate wall pieces in your source file. There is no door, ladder or path LOD generation yet.
- There is only one resolution LOD (no auto-generated lower-detail LODs).

## Source layout
```
Pluto.Core/   import, collision, P3D/PAA/PBO/rvmat/config writers, renderer (cross-platform)
Pluto.App/    Windows UI (WinForms)
Pluto.Cli/    command line: build / inspect / preview / selftest
```
Build: `dotnet publish Pluto.App -c Release` (.NET 8 SDK).
CLI example: `pluto-cli build --name "My Pack" --out D:\Mods C:\models\props`
