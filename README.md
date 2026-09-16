# Universal Studios Singapore — 3D Miniature Park Planner

An interactive 3D miniature of Universal Studios Singapore, built as a single HTML file
with Three.js. It exists to answer the questions you actually have before a park day:
*where is everything, how far apart is it, and what order should I ride things in?*

Seven themed zones, eighteen attractions, the real ring-shaped walking loop around the
lagoon, and a saved itinerary that totals your walking and queueing time.

---

## Setup

You need a local web server — the page loads Three.js as an ES module, so opening
`index.html` with `file://` will not work.

### Run it

```bash
python3 -m http.server 63214 --bind 127.0.0.1
```

Run it from the folder containing `index.html`.

Then open:

```
http://127.0.0.1:63214/index.html
```

A server is already running on port 63214. If that port is taken, any other works:

```bash
python3 -m http.server 8080 --bind 127.0.0.1
```

Node users can use `npx serve .` or `npx http-server -p 8080` instead.

### Requirements

- A WebGL 2 browser: Chrome, Edge, Safari 16+, or Firefox.
- An internet connection **on first load** — Three.js r160 comes from unpkg and the two
  display fonts from Google Fonts. Everything else (every texture, every building) is
  generated in the browser, so there are no image files to download or ship.
- No build step, no `npm install`, no dependencies directory. One file, 159 KB.

---

## What you can do

### Getting around

| Action | How |
| --- | --- |
| Orbit | Drag |
| Pan | Right-drag, or two-finger drag |
| Zoom | Scroll wheel, pinch, or the **Zoom** slider in the bottom dock |
| Fly to a ride | Click it in the 3D view, click its label, or click it in the left list |
| Fly to a zone | Click a zone card in **Park Zones**, or press `1`–`7` |
| Reset the view | `R`, or the ⟲ button in the dock |

### Flight Mode

Press `F` or hit **Flight Mode**. The camera tours all eighteen attractions in walking
order around the park loop — five seconds of travel, then a slow orbit while a caption
gives you the zone, the ride type, the current modelled wait, and a one-line description.
Skip forward and back, pause with `Space`, leave with `Esc`.

If you have stops in your plan, **▶ Tour it** inside the plan panel flies that itinerary
instead, in your chosen order.

### Planning a day

- **Search** (`/`) matches ride names, zone names, and ride types — try `coaster`,
  `indoor`, `egypt`, `show`.
- **Filter chips** narrow by thrill rides, family rides, little kids, shows, food, and
  photo spots.
- **Rider height** greys out everything that would turn your group away at the gate. Set
  it once to 92 cm or 102 cm and the map stops lying to you about what your day looks like.
- **Add to plan** (the `+` on any list row, or the button on the detail card) builds an
  itinerary. The panel shows an arrival time per stop, the walk between consecutive
  stops, and running totals for walking distance, queueing time, and total day length.
- **⚡ Optimise** reorders your stops to cut walking. The park is a loop, so order matters
  a lot — this is usually worth 15–30 minutes.
- **Preset plans** — Thrill rush, With small kids, Greatest hits, Rainy day, Evening loop —
  fill the itinerary in one click.
- **Walk rings** draw 100 m and 200 m circles around the selected stop, so "is that close?"
  gets a real answer.
- **Park time** slider moves between 09:00 and 22:00. Queue lengths follow a modelled
  daily curve, the lighting moves from morning through golden hour to night, and window
  and neon lights come on after dark. `N` jumps between midday and night.

### Detail cards

Clicking any attraction opens a card with ride type, ride length, minimum height,
a thrill rating, whether Express and single-rider lines exist, whether it is indoors or
gets you wet, a typical-wait-through-the-day bar chart, the best time of day to go,
three practical tips, and the walking distance to its three nearest neighbours.

### Sharing and saving

Your itinerary, rider height, and settings persist in `localStorage`. **Share** copies a
URL that reopens the exact view — selected ride, plan, zone, height filter, and time of
day — so you can send a plan to whoever you are travelling with.

Parameters: `?at=<id>&plan=<id,id,…>&zone=<zone>&h=<cm>&t=<hour>`

### Keyboard

`F` flight mode · `T` top view · `R` reset view · `L` labels · `M` miniature blur ·
`N` day/night · `/` search · `1`–`7` zones · `Space` pause the tour · `Esc` back out ·
`?` help

---

## Accuracy — please read this part

**The wait times are modelled, not live.** Each ride carries a hand-authored hourly curve
that reflects how the park generally behaves — Transformers and Battlestar spiking mid-
afternoon, the kiddie rides staying short all day. It is useful for deciding what to do
first. It is not a feed, and it will not know about a breakdown or a public holiday.

Likewise **modelled, not authoritative**: height limits, Express availability, single-rider
lines, ride durations, and show lengths. They are close to reality as of 2025, and USS
changes them. Minion Land opened in 2025 in place of the old Madagascar zone, and this
build reflects that. **Check the official Universal Studios Singapore app on the day** for
live queues, show times, and closures.

**The geometry is a stylised miniature, not a survey.** Zone positions, the lagoon, and the
ring path follow the real park layout, so relative distances and "which zone is next to
which" are trustworthy for planning. Individual buildings are recognisable impressions —
the right silhouette, the right colours, the right roofline — not measured reconstructions.
Walking distances come from straight-line distance multiplied by a 1.25 detour factor at a
4.4 km/h theme-park pace, which is a decent estimate and not a routed path.

No reference video was provided for the visual direction, despite the brief mentioning one.
The look here is the standard miniature-diorama treatment: tilt-shift depth of field,
chunky low-poly massing, soft shadows, warm atmospheric light. If you had a specific
reference in mind, the art direction is easy to redirect.

---

## How it is built

Everything lives in `index.html`. No assets, no bundler.

- **Three.js r160** via an import map from unpkg, plus `OrbitControls`, `EffectComposer`,
  and `mergeGeometries` from the same addons.
- **Procedural textures.** Brick, glass facades, sandstone with carved hieroglyph panels,
  paving, water, foliage — all painted to a `<canvas>` at build time and uploaded as
  `CanvasTexture`. That is why there are no image files.
- **Draw-call discipline.** Materials are cached by their parameters, then `bake()` merges
  every mesh that shares a material into one geometry; meshes that differ only in colour
  get merged too, with the colour moved into vertex colours. Animated subtrees (the
  spinning globe, drifting rafts, flickering braziers) are detected and left alone. Trees
  and the 420-guest crowd are `InstancedMesh`. Landmarks stay clickable through invisible
  pick-proxy boxes, so their visible geometry is free to merge away.
- **Post-processing.** A custom tilt-shift shader pass over the render pass, which is what
  sells the "miniature" read. Toggle it with `M`.
- **Accessibility.** ARIA roles and live regions on the panels, keyboard operation for
  every control, focus management on dialogs, and `prefers-reduced-motion` support that
  makes camera moves instant.
- **Responsive** down to 375 px, where the left panel becomes a sheet behind the ☰ button
  and the bottom dock and plan panel take turns at the bottom edge.

### Performance note

The scene is roughly 500 draw calls and 520 k triangles with two shadow maps, which is
comfortable on integrated graphics. If it feels heavy on an older machine, turning off the
miniature blur (`M`) removes the post-processing pass.

---

## License

MIT — see [LICENSE](LICENSE).

Not affiliated with, endorsed by, or connected to Universal Studios, NBCUniversal, or
Resorts World Sentosa. Attraction and zone names are used descriptively to identify real
places for trip planning; all trademarks belong to their respective owners.
