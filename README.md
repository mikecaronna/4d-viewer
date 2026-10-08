# 4D capture viewer

Volumetric video from Sony RX0 camera arrays, played back in a browser. Orbit
with the mouse, scrub the timeline.

Live at https://mikecaronna.github.io/4d-viewer/

## What is here

| | |
|---|---|
| `index.html` | the landing page |
| `app/` | the built viewer |
| `assets/*.sog4d` | the scenes, compressed. Not in this repository: see below |
| `THIRD-PARTY-NOTICES.md` | license text for everything the build contains |
| `LICENSE` | covers the files in this repository that are not the build |

The scenes are served from Cloudflare R2 rather than from here. Git keeps every
version of a binary forever, GitHub rejects any single file over 100 MB, and
four scenes come to about 175 MB. Upload them with:

```
rclone copy assets r2:4d-scenes --include "*.sog4d" --progress
```

That bucket is public-read, so anything placed in `assets/` becomes fetchable
by filename. Keep unpublished scenes out of it.

## The viewer

The viewer is a fork of [supersplat4d](https://github.com/forgrabbit/supersplat4d),
which derives from [PlayCanvas SuperSplat](https://github.com/playcanvas/supersplat).
The modified source is at
https://github.com/mikecaronna/supersplat4d/tree/starling-viewer and `app/` is
a build of it.

Seven changes separate it from upstream, and the commit messages on that branch
explain each one. In short:

1. A `view=` address parameter that restores a camera exactly. The older `cam=`
   form is a world position and does not survive a rebuild of the scene.
2. Viewer mode by default, with the editor behind `edit=1`.
3. The frame rate readout hidden for readers.
4. Upstream's mobile camera limits removed, which otherwise stop a phone from
   getting close to the subject.
5. Ping-pong playback under `bounce=1`, which turns the clip round at the ends
   rather than cutting back to the start.
6. The eruda debug console removed. It was loaded from a CDN on every page view.
7. Page views counted with Cloudflare Web Analytics, which the build adds only
   when `CF_BEACON_TOKEN` is set. A fork built without it reports nothing.

To rebuild:

```
git clone -b starling-viewer https://github.com/mikecaronna/supersplat4d.git
cd supersplat4d
npm install
npm run build:prod
```

`build:prod` matters. The default `npm run build` produces a profiler build and
ships the instrumented PlayCanvas engine to every visitor. The output lands in
`dist/`, which replaces `app/` here.

Release builds do not emit source maps. A map embeds the complete source of
every file in the bundle, which was 12 MB of a 22 MB payload and republished
774 upstream source files with their copyright notices absent.

## Viewer mode and editor mode

This application is an editor. Served as a public viewer it handed every visitor
the editing apparatus: a tool palette over a third of the scene, menus belonging
to the tool rather than the work, and a live Rotation field that tipped the
scene over if anyone typed in it.

Viewer mode is the default. It keeps the scene, the timeline and the
attribution, and hides the rest.

**Add `&edit=1` to any scene link for the full editor**, with the toolbars, the
scene panel and the frame rate readout. That is how scenes get cleaned, so it is
one step away rather than removed.

```
.../app/?lng=en&edit=1&load=https://pub-5d477fff8046453485dba51fd2ed2aa6.r2.dev/nick_guitar.sog4d
```

Clean a scene by opening it with `edit=1`, deleting what should go, and
exporting. `apply_edit.py` in the capture pipeline then carries that cleanup
onto later builds of the same scene, so the work survives a retrain.

## Address parameters

| | |
|---|---|
| `load=` | the scene to fetch |
| `view=fx,fy,fz,azim,elev,distance` | the opening camera. Press V in any scene to copy the current one |
| `cam=px,py,pz,tx,ty,tz` | an older camera form, kept for existing links |
| `zoom=` | multiplies the framing radius, so a scene captured from inside a room can open from outside it |
| `bounce=1` | play forward, then backward |
| `edit=1` | the full editor |

## The capture

Sony RX0 cameras on stands around the subject, all recording at once at
29.97 fps. The number varies by shoot: 54 for the skateboard scene, 41 for the
guitar, 39 for the drums, 28 for the tennis.

Reconstructed with FreeTimeGS++ ([arXiv 2605.03337](https://arxiv.org/abs/2605.03337)),
in which each primitive carries a position, a time center, a duration and a
velocity, so the scene genuinely moves rather than being a flipbook of separate
reconstructions.

Scenes are rotated upright before packaging, so no orientation correction is
needed in the viewer.

## Known limitations

- **No view-dependent color.** The `.sog4d` container keeps only the degree-0
  spherical harmonic term and drops the rest, so color does not shift with
  viewing angle.
- **No progressive loading.** The file downloads whole before the first frame
  appears, and the scenes run from 34 MB to 59 MB. On a phone that is a wait.
- **Stray ellipsoids around the edges** are the stands and the rig
  photographing itself, reconstructed correctly.

## License and ownership

The captures are © Mike Caronna, made in a personal capacity. They are not
software and the licenses below do not cover them.

The viewer in `app/` is a build of MIT-licensed software from PlayCanvas Ltd
together with several other libraries. `THIRD-PARTY-NOTICES.md` carries the
license text for all of it and says where to get the corresponding source.

`LICENSE` covers the remaining files here, meaning the landing page and this
documentation.
