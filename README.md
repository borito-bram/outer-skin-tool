# Outer Skin Tool

A single-page browser tool that strips a STEP model down to the faces you can see from outside. Contact faces between parts, hidden parts and internal cavities are removed, so you can share a model's outer shape without giving away its internals. Part names, colours, positions and the assembly tree are kept.

Everything runs locally in the browser. The STEP file is never uploaded.

## Use

1. Open `index.html` in a recent Chrome, Edge or Firefox (WebGL2 required).
2. Choose or drop a `.step` / `.stp` file. The model loads into the viewer.
3. Optional: draw **covers** over openings. Click **Draw cover**, click points around the edge of an opening, then press Enter or double-click. Everything behind a cover counts as hidden. Covers can be added to the output as plates so the opening looks closed.
4. Pick a gap size and accuracy, then click **Create outer skin**.
5. Check the before / after / removed views and the hole-check result, then download the new STEP file.

The first run downloads the OpenCascade CAD engine (about 48 MB) from jsDelivr; the browser caches it after that.

If the page won't start when opened straight from disk, serve the folder locally:

```bash
py -m http.server 8000 --bind 127.0.0.1
```

and open `http://localhost:8000/`.

## How it works

1. **Read and mesh.** The STEP file is read with [opencascade.js](https://github.com/donalffons/opencascade.js) into an XCAF document, and every face of every part instance is meshed.
2. **Visible faces.** The model is rendered on the GPU from 384 directions (Normal accuracy) with a colour per face, and every face that shows up in a render is marked visible.
3. **Gap closing.** The model is voxelized. A face is only kept if a ball of the chosen gap diameter, coming from outside, can touch it, so faces seen only through narrow slits are removed.
4. **Ray check.** Faces the renders could not decide are checked by casting rays from points on the face, spread over all CPU cores.
5. **Hole check.** The result is rendered from 64 new directions. Any removed face that still shows is put back, then a second set of directions measures what is left.
6. **Write.** A new XCAF document is built with the same tree, names and colours and only the kept faces, and written as STEP. Each part instance gets its own definition, because visibility can differ between instances of the same part.

Covers are flat plates built from the clicked points. They block sight in steps 2–5, and are optionally written as extra parts named `Cover 1`, `Cover 2` and so on.

## Limits

- The output faces are open surfaces, not closed solids, so volume and mass are no longer valid in CAD.
- Part names are kept as they are and may reveal supplier part numbers.
- Very large assemblies can run slowly or run out of browser memory.

## Third-party code

Loaded at runtime from jsDelivr, not bundled:

- [opencascade.js](https://github.com/donalffons/opencascade.js) 2.0.0-beta (OpenCascade Technology, LGPL-2.1 with exception)
- [three.js](https://github.com/mrdoob/three.js) 0.160.0 (MIT)
