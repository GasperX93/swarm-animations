# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# Swarm Educational Animations — Claude Code Instructions

## What this project is

A series of self-contained canvas animations that visually explain how the Swarm decentralised storage network works. Each animation is a single HTML file with vanilla JS — no frameworks, no build tools, no external dependencies. The target audience is developers and non-technical people who want to understand Swarm's inner workings.

## Development

There is no build step, package manager, or test suite. Each scene is a standalone HTML file.

- **Preview a scene**: Open the HTML file directly in a browser (`open topology.html` on macOS)
- **Validate**: Visual inspection at 1280×680 — check no clipping, smooth looping, correct render order
- **No linting/formatting tools** are configured; just follow the code patterns in this file


## Reference material

### Swarm concepts

Before building a new scene, read the relevant chapter from **The Book of Swarm**:
- Repository: `https://github.com/ethersphere/the-book-of-swarm`
- The PDF is in the repo root. Key chapters for animation topics:
  - **Ch 1–2**: High-level architecture, design goals
  - **Ch 3**: Overlay network, Kademlia topology, proximity order (XOR distance)
  - **Ch 4**: Chunking, content addressing, Swarm hash, erasure coding
  - **Ch 5**: Push syncing, pull syncing, chunk retrieval
  - **Ch 6**: PSS (Postal Service over Swarm) — message routing
  - **Ch 7**: Incentives — postage stamps, redistribution
  - **Ch 8**: ACT (Access Control Trie) — encryption, access control

Always ground your animation in the actual protocol mechanics from this book. Simplify for visuals, but don't misrepresent.

### Design language

For colour palette inspiration, component patterns, and brand voice, reference the Ethswarm website:
- Repository: `https://github.com/ethersphere/ethswarm-nextjs`
- Key files: `tailwind.config.js` for colours, `src/components/` for UI patterns
- The orange (`#d4633a`) and dark background (`#272727`) come from here

## Scenes built so far

| # | File | Concept | Animation type |
|---|------|---------|----------------|
| 1 | `topology.html` | Traditional internet vs P2P Swarm topology | Continuous looping packets |
| 2 | `chunks.html` | Content-addressed storage: file → manifest + chunks | Staged reveal, loops |
| 3 | `storage.html` | Neighbourhood storage: chunks stored by proximity order | Staged per-chunk animation |

## Possible next scenes

These are suggestions based on the Book of Swarm. Each should explain **one concept**:

- **Push syncing** — a chunk gets pushed from the uploader outward through the network until it reaches the responsible neighbourhood
- **Pull syncing** — a node requests chunks from its neighbours to fill gaps in its local store
- **Chunk retrieval** — a request travels through forwarding Kademlia to find and return a chunk
- **Postage stamps** — attaching a stamp to a chunk to pay for storage; stamp value and batch depth
- **Erasure coding** — a file is split into data + parity chunks, showing how any subset can reconstruct the original
- **PSS messaging** — Trojan chunks carrying encrypted messages routed to a target neighbourhood
- **ACT access control** — encryption keys managed through a trie; granting/revoking access to content
- **Feeds** — single-owner chunks forming an updateable pointer; resolving latest version

---

## Visual identity — NEVER deviate from these

```
Background:  #272727
Orange:      #d4633a  (nodes, packets, header bar)
White:       #ffffff  (lines, labels, icon strokes)
Dark:        #191919  (inner circle fill)
```

### Nodes
- **Full-size**: outer ring R=44, inner circle R=32, white 3px stroke
- **Small (grid/swarm)**: outer ring R=34, inner circle R=24, same stroke
- **Chunk-size**: outer ring R=22, inner circle R=15
- Always: orange fill on ring, `#191919` inner fill, white stroke

### Lines
- White `#ffffff`, 1.5px default
- Orange `#d4633a` for semantic connections (e.g., manifest→chunk pointers)
- Dashed lines for in-progress/active connections

### Typography
- Labels: `bold 17px Arial, sans-serif`, white
- Header bar: `italic bold 22px Georgia, serif`, white on orange `#d4633a` bar, full-width, 42px tall, centered
- Section labels: `bold 18px Arial, sans-serif`, white, left-aligned
- Small labels: `bold 13px Arial, sans-serif` or `bold 13px monospace` for hex addresses
- Explanatory text: `15px Arial, sans-serif`, `rgba(255,255,255,0.6)`, bottom of canvas

### Canvas
- Fixed: `1280×680`
- CSS: `max-width: 1280px; width: 100%; height: auto;`
- Scales responsively via CSS, canvas coordinates stay 1280×680

---

## Code architecture

### File structure

Every scene is a single HTML file. No external files. Structure:

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Scene Name – Ethswarm</title>
<style>
* { margin: 0; padding: 0; box-sizing: border-box; }
body { background: #272727; display: flex; justify-content: center; align-items: flex-start; min-height: 100vh; }
canvas { max-width: 1280px; width: 100%; height: auto; display: block; }
</style>
</head>
<body>
<canvas id="c" width="1280" height="680"></canvas>
<script>
// ── Constants ──
// ── Seeded RNG ──
// ── Geometry helpers ──
// ── Drawing primitives ──
// ── Icon functions ──
// ── Packet / animation classes ──
// ── Node positions ──
// ── Scene-specific logic ──
// ── Header bar ──
// ── Main loop ──
</script>
</body>
</html>
```

Sections delimited by `// ── Name ──` comments.

### Primitives — copy these exactly into every new scene

```js
// ── Constants ──
const W = 1280, H = 680;
const BG = '#272727', ORANGE = '#d4633a', WHITE = '#ffffff', DARK = '#191919';
const NODE_R = 44, INNER_R = 32, STROKE_W = 3, LINE_W = 1.5;
const FONT = 'bold 17px Arial, sans-serif';

const canvas = document.getElementById('c');
const ctx = canvas.getContext('2d');

// ── Seeded RNG ──
let seed = 42;
function rng() { seed = (seed * 16807 + 0) % 2147483647; return (seed - 1) / 2147483646; }
function rngHex(n) { let s = ''; for (let i = 0; i < n; i++) s += '0123456789abcdef'[Math.floor(rng()*16)]; return s; }

// ── Geometry helpers ──
function ptLen(a, b) { const dx = b[0]-a[0], dy = b[1]-a[1]; return Math.sqrt(dx*dx+dy*dy); }
function pathLen(pts) { let s=0; for(let i=1;i<pts.length;i++) s+=ptLen(pts[i-1],pts[i]); return s; }
function posAt(pts, d) {
    let rem = d;
    for (let i=1; i<pts.length; i++) {
        const seg = ptLen(pts[i-1], pts[i]);
        if (rem <= seg) {
            const t = rem / seg;
            return [pts[i-1][0]+(pts[i][0]-pts[i-1][0])*t, pts[i-1][1]+(pts[i][1]-pts[i-1][1])*t];
        }
        rem -= seg;
    }
    return pts[pts.length-1];
}

// ── Drawing primitives ──
function ln(x1,y1,x2,y2,color,width) {
    ctx.beginPath(); ctx.moveTo(x1,y1); ctx.lineTo(x2,y2);
    ctx.strokeStyle = color || WHITE; ctx.lineWidth = width || LINE_W; ctx.stroke();
}

function shell(x,y,r,ir) {
    r = r || NODE_R; ir = ir || INNER_R;
    ctx.beginPath(); ctx.arc(x,y,r,0,Math.PI*2);
    ctx.fillStyle = ORANGE; ctx.fill();
    ctx.strokeStyle = WHITE; ctx.lineWidth = STROKE_W; ctx.stroke();
    ctx.beginPath(); ctx.arc(x,y,ir,0,Math.PI*2);
    ctx.fillStyle = DARK; ctx.fill();
}

function renderText(text,x,y,color,font,align,baseline) {
    ctx.fillStyle = color || WHITE;
    ctx.font = font || FONT;
    ctx.textAlign = align || 'center';
    ctx.textBaseline = baseline || 'top';
    ctx.fillText(text,x,y);
}
```

### Header bar — same in every scene

```js
function drawHeader() {
    ctx.fillStyle = ORANGE;
    ctx.fillRect(0, 0, W, 42);
    ctx.fillStyle = WHITE;
    ctx.font = 'italic bold 22px Georgia, serif';
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';
    ctx.fillText('Scene Title', W/2, 21);
}
```

### Easing helpers (for staged animations)

```js
function easeOut(t) { return 1 - (1 - t) * (1 - t); }
function easeInOut(t) { return t < 0.5 ? 2*t*t : 1 - Math.pow(-2*t+2, 2)/2; }
function clamp01(t) { return Math.max(0, Math.min(1, t)); }
```

### Packet class (for continuous-flow scenes)

```js
class Packet {
    constructor(pts, radius, speed, startDist) {
        this.pts = pts;
        this.radius = radius;
        this.speed = speed;
        this.total = pathLen(pts);
        this.d = startDist % this.total;
        this.life = 1;
        this.destination = null;
    }
    update(dt) { this.d = (this.d + this.speed * dt) % this.total; }
    draw() {
        if (this.life <= 0) return;
        const p = posAt(this.pts, this.d);
        ctx.beginPath(); ctx.arc(p[0],p[1],this.radius,0,Math.PI*2);
        ctx.fillStyle = ORANGE; ctx.fill();
    }
}
```

### Icon library

Reuse these. Do not reinvent them.

| Icon | Function | Used for |
|------|----------|----------|
| `iconLaptop(x,y)` | Laptop outline | User/You node |
| `iconGlobe(x,y)` | Globe with lat/lon lines | ISP node |
| `iconServer(x,y)` | 3-stack server rack | Server/destination node |
| `iconHexDouble(x,y)` | Double hexagon | Swarm node |
| `iconChunk(x,y)` | Document with folded corner | Data chunk |
| `iconImage(x,y,scale)` | Image frame with mountain+sun | File/manifest |

Add new icons only when genuinely needed. Draw them in the same style: white strokes on dark, 2px lineWidth, ~24–30px bounding box.

---

## Render order — ALWAYS respected

1. **prerender** — `ctx.fillRect` background clear, static lines between nodes
2. **render** — packets / animated elements (drawn BEFORE nodes so nodes sit on top)
3. **postrender** — node shells + icons + labels (always on top of everything)

Nodes always render on top of packets. Labels always render on top of nodes.

### Main loop pattern

```js
let lastTime = 0;
function frame(ts) {
    const dt = Math.min((ts - lastTime) / 1000, 0.05); // cap at 50ms
    lastTime = ts;

    // 1. prerender
    ctx.fillStyle = BG;
    ctx.fillRect(0, 0, W, H);
    drawHeader();
    drawLines();

    // 2. update + render packets/animations
    manager.update(dt);
    manager.draw();

    // 3. postrender — nodes on top
    for (const n of allNodes) {
        shell(n.x, n.y, n.r, n.ir);
        n.icon(n.x, n.y);
        renderText(n.label, n.x, n.y + (n.r || NODE_R) + 6, WHITE, FONT, 'center', 'top');
    }

    requestAnimationFrame(frame);
}
requestAnimationFrame(frame);
```

---

## Packet semantics — the conceptual core

Packet **size** and **timing** carry meaning:

| Situation | Size | Timing | Meaning |
|---|---|---|---|
| Traditional internet | Varying 3–14px | Strongly irregular | ISP can read metadata from both |
| Swarm P2P entry | Uniform 5px | Slight jitter ±14% | Chunk size is fixed (4 kB); minor timing leak |
| Swarm internal routing | Uniform 5px | Slight jitter | Same chunk, hopping through nodes |

Packets loop continuously in flow scenes. Pre-populate each stream so it looks busy from frame 0 (place packets at staggered `startDist` offsets across the full path length).

---

## Node types

| Type | Icon | Behaviour on packet arrival |
|------|------|-----------------------------|
| `UserNode` | Laptop | Source only, emits packets |
| `ISPNode` | Globe | Fans out to ALL destinations simultaneously |
| `ServerNode` | Server rack | Terminal — kills packet on arrival |
| `SwarmNode` | Double hexagon | Picks ONE random destination, forwards |

Nodes own their destination lists. A node's `prerender` draws lines to its destinations. `onPacketReceived` either assigns `packet.destination = pick(destinations)` to forward, or sets `packet.life = 0` to consume.

---

## Animation types

### 1. Continuous flow (topology.html)
- Packets loop forever on fixed paths
- Pre-populate streams so frame 0 looks busy
- Good for: network topology, traffic patterns, routing

### 2. Staged reveal (chunks.html)
- Timed phases: appear → build → connect → hold → fade → loop
- Each cycle can regenerate random data (hashes, addresses)
- Good for: data structures, relationships, explanations with a narrative arc

### 3. Staged per-item (storage.html)
- A sequence of N items animate one by one
- Each item goes through: appear → highlight target → travel → absorb
- Good for: routing decisions, storage placement, any process that repeats for multiple items

---

## Design principles

1. **One concept per scene.** Don't combine chunking and routing in one animation. Let each scene be focused and simple.

2. **No text walls.** Use at most one short explanatory sentence at the bottom of the canvas. Let the visual do the work.

3. **Colour carries meaning.** Use the orange/white/dark palette for structure. Introduce colour (HSL hues) only when it encodes information (e.g., address proximity). Never use colour decoratively.

4. **Motion carries meaning.** Packet size, speed, timing, and path all communicate protocol behaviour. Don't add motion for aesthetics.

5. **Deterministic on load.** Use seeded RNG so the animation looks the same on every page load. Different seeds per scene are fine, but within a scene the initial state should be reproducible.

6. **Respect the render order.** Nodes on top of packets. Labels on top of nodes. No exceptions.

7. **Fit in 1280×680.** Every element including labels must be fully visible. Bottom-row labels at y + NODE_R + 25 must be < 680. Test this.

8. **No external dependencies.** Single HTML file. No CDN imports, no fetch calls, no images. Everything is canvas-drawn.

---

## Checklist for building a new scene

1. Read the relevant Book of Swarm chapter for the concept
2. Define node positions on paper first — ensure everything fits in 1280×680 with label clearance
3. Copy the full primitives block (constants, RNG, geometry, drawing, icons, header)
4. Construct nodes right-to-left (terminals first, so destinations exist before referencing)
5. Wire destinations and create packet/animation managers
6. Implement the main loop respecting render order
7. Add the header bar with the scene title
8. Add one short explanatory sentence at the bottom
9. Test at actual 1280×680 — check no clipping on any edge
10. Verify animation loops cleanly (no visual jump at reset)

---

## Common mistakes to avoid

- **Nodes too big for spacing.** If using a grid, check that `2 × NODE_R + label height + gap` fits the row spacing. Use smaller nodes (R=34) for dense grids.
- **Labels overflowing canvas.** Bottom-most node label must end above y=680. Calculate: `nodeY + nodeR + 6 (gap) + 20 (text height) < 680`.
- **Uneven grid spacing.** Use calculated positions from center: `centerX ± n × spacing`, not hand-picked pixel values.
- **Forgetting to pre-populate streams.** A scene that starts empty and slowly fills up looks broken. Stagger initial `startDist` values.
- **Modifying primitives.** Do not rewrite `shell`, `ln`, `Packet`, `posAt`. If you need different behaviour, extend with new parameters (like the optional `r, ir` params on `shell`) rather than forking.
- **Random runtime behaviour without seeded RNG.** Use `rng()` instead of `Math.random()` everywhere so the scene is deterministic on load.
