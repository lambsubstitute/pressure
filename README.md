# PRESSURE

A pressure-washing game. Nine years of algae on a rear patio in the rain shadow of a
leylandii hedge, one petrol machine, four nozzles, and a client who wants it back to the
colour it was when it was laid.

It is one HTML file. No build step, no bundler, no asset pipeline — every texture, the
sky, and all of the audio are generated procedurally at load. The only dependency is
three.js from a CDN. Visits to the live site are counted with GoatCounter (no cookies,
no personal data), along with whether a visit reaches 1, 5, 15 or 30 minutes of play.

## Play

Open `index.html` in a browser. That is the whole installation procedure.

Needs WebGL 2 and a machine that can push a full-resolution buffer with 4× multisampling —
anything with a discrete GPU or recent integrated graphics is comfortable.

## Controls

| | |
|---|---|
| `W` `A` `S` `D` | walk · hold `Shift` to move quickly |
| Mouse | aim · hold left button to spray |
| `1` – `4` / wheel | change nozzle |
| `Q` / `R` | roll the fan |
| `C` | crouch — get the angle on low walls |
| `E` | refill at the outside tap |
| `Esc` | release the mouse |

Roll matters. A fan tip lays a flat blade, not a cone, so the blade wants to sit across the
direction you are sweeping. The crosshair *is* the blade, drawn at its true angular width —
what you see is the footprint you are about to lay down.

## The nozzles

Real trade colour codes, and they behave like the real things.

| | Tip | Blade at 0.5 m | Notes |
|---|---|---|---|
| 🔴 | 0° | 12 mm | Will strip mortar, and will destroy a border in about a second |
| 🟡 | 15° | 13 cm | For the moss in the shaded corner |
| 🟢 | 25° | 22 cm | The one you will live on |
| ⚪ | 40° | 36 cm | Fast, gentle, wants two passes |

Work at roughly 0.3 m/s and one pass cuts to bare stone. Hurry it at 1 m/s and you get a
ghost you will have to come back over. That relationship is the game.

## How it works

**One dirt atlas.** Every cleanable surface in the level claims a rectangle in a single
2048² RGBA8 texture, sized by its real world area, so texel density is constant everywhere
and the progress readout is area-weighted for free. `R` is dirt, `G` is dryness. Splats
subtract with `ReverseSubtract`, so one quad both removes filth and adds wetness in the same
draw, and a full-screen additive pass dries it back off over about eighteen seconds.

**Swept capsules, not dots.** The naive approach — stamp a circle where the raycast landed
this frame — produces a dotted trail at any speed above a crawl, because the gap between
frames is not filled by anything. Instead, each sample across the fan blade remembers where
it landed last frame and stamps the *capsule* between the two points. The gap is filled by
geometry rather than by hoping the frame rate is high enough. It only joins across frames
when both hits are on the same face of the same slab, so it cannot smear across an atlas
seam.

**Dose is conserved along the sweep.** Smearing a disc of radius `r` along `L` metres
spreads the same water over `πr² + 2rL` instead of `πr²`, so the splat amplitude is divided
by that ratio. Two things fall out of this: a slow pass scours and a fast one only ghosts,
which is the entire feel of the game; and the result stops depending on frame rate. Measured
across 30/60/144 fps for an identical sweep, the residual dirt agrees to within 2%.

**Progress is measured on the GPU.** The atlas is reduced 2048 → 256 → 64 through two
box-filter passes, then 16 KB is read back every fourteenth frame. A precomputed table maps
each of the 4096 cells to the objective groups that overlap it, so four independent
percentages come out of one readback.

**Everything is procedural.** Tileable fBm built by upscaling wrapped noise grids, packed
four greyscale octave stacks to an RGBA channel each. The overcast sky is a canvas
equirect that doubles as the IBL source via `PMREMGenerator`. The machine is a two-stroke
loop of detuned oscillators through a resonant lowpass that bogs down under load; the water
is filtered noise whose brightness tracks your stand-off distance.

**The look is a post pass.** The scene renders linear into a half-float buffer and
everything photographic happens at the end: two-stage bloom off the wet highlights, ACES,
contrast, a cool cast pushed into the shadows only, vignette, grain, radial chromatic
aberration, and water on the lens that builds while you work in close and clears over a
couple of seconds.

The one shader trick worth stealing: wet grime goes almost black while wet clean stone stays
bright, so the contrast between done and not-done peaks at exactly the moment you are
looking at it.

## Status

Prototype. One contract, one garden. The job sheet, the money and the star rating are all
wired up and the level completes properly, but there is only the one level.

Known gaps:

- No screenshots in this README yet.
- The atlas is not saved, so closing the tab loses the job.
- Slab subdivision, detergent (the `B` channel is reserved for it) and a second contract are
  all unbuilt.

## Licence

MIT.
