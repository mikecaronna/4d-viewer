# 4D capture viewer

Volumetric video from a 41-camera array, played back in a browser. Orbit with
the mouse, scrub the timeline.

Live at https://mikecaronna.github.io/4d-viewer/

## What is here

| | |
|---|---|
| `index.html` | the landing page |
| `app/` | the viewer, built from [supersplat4d](https://github.com/forgrabbit/supersplat4d) (MIT) |
| `assets/*.sog4d` | the scenes, compressed |
| `viewer-changes.patch` | the two local changes made to the viewer source |

## The two viewer changes

1. `src/index.html` — removes the eruda debug console, which was loaded from a
   CDN on every page view. With it gone the page makes no outbound request.
2. `src/file-handler.ts` — adds a `zoom` address parameter that multiplies the
   camera framing radius, so a scene captured from inside a room can open from
   outside it.

To rebuild the viewer from source:

```
git clone https://github.com/forgrabbit/supersplat4d.git
cd supersplat4d
git checkout 7f1a39da84a5dded583f724da27213fcfe1a8912
git apply ../viewer-changes.patch
npm install
BUILD_TYPE=prod npx rollup -c
```

## Viewer mode and editor mode

This application is SuperSplat, which is an editor. Served as a public viewer
it handed every visitor the editing apparatus: a tool palette over a third of
the scene, menus belonging to the tool rather than the work, and a live
Rotation field that tipped the scene over if anyone typed in it.

Viewer mode is the default. It keeps the scene, the timeline and the
attribution, and hides the rest.

**Add `&edit=1` to any scene link for the full editor**, with the toolbars,
the scene panel and the frame rate readout. That is how scenes get cleaned,
so it is one step away rather than removed.

```
.../app/?lng=en&edit=1&load=https://pub-5d477fff8046453485dba51fd2ed2aa6.r2.dev/nick_guitar.sog4d
```

Clean a scene by opening it with `edit=1`, deleting what should go, and
exporting. `Tools/rx0-4d-pipeline/apply_edit.py` then carries that cleanup
onto later builds of the same scene, so the work survives a retrain.

## The capture

Forty-one Sony RX0 cameras on stands around the subject, all recording at once
at 29.97 fps. Reconstructed with FreeTimeGS++ ([arXiv
2605.03337](https://arxiv.org/abs/2605.03337)), in which each primitive carries
a position, a time centre, a duration and a velocity, so the scene genuinely
moves rather than being a flipbook of separate reconstructions.

The scene is rotated upright before packaging, so no orientation correction is
needed in the viewer.

## Known limitations

- **No view-dependent colour.** The `.sog4d` container keeps only the degree-0
  spherical harmonic term and silently drops the rest, so colour does not shift
  with viewing angle.
- **No progressive loading.** The file downloads whole before the first frame
  appears. At 10 MB that is about a second.
- **Stray ellipsoids around the edges** are the stands and the rig
  photographing itself, reconstructed correctly.

## Licence and ownership

The captures are © Mike Caronna, made in a personal capacity. The viewer is
MIT, from the upstream project linked above.
