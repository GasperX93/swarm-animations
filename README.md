# Swarm Educational Animations

A collection of self-contained canvas animations that visually explain how the [Swarm](https://www.ethswarm.org/) decentralised storage network works, based on the [Book of Swarm](https://papers.ethswarm.org/p/book-of-swarm/). Each animation is a single HTML file with vanilla JS — no frameworks, no build tools, no external dependencies.

## Credits

Design and concept by [@Cafe137](https://github.com/Cafe137).

## Scenes

| # | File | Concept |
|---|------|---------|
| 1 | [topology-left-right.html](animations/topology-left-right.html) | Traditional internet vs P2P Swarm topology (side-by-side) |
| 2 | [topology-top-bottom.html](animations/topology-top-bottom.html) | Traditional internet vs P2P Swarm topology (stacked) |
| 3 | [push-syncing.html](animations/push-syncing.html) | Chunks pushed outward through the network to the responsible neighbourhood |
| 4 | [pull-syncing.html](animations/pull-syncing.html) | Nodes request missing chunks from peers to fill gaps in local store |
| 5 | [chunk-retrieval.html](animations/chunk-retrieval.html) | Requests forwarded via Kademlia routing, chunks returned along the same path |
| 6 | [postage-stamps.html](animations/postage-stamps.html) | Attaching postage stamps to chunks to pay for storage |
| 7 | [erasure-coding.html](animations/erasure-coding.html) | Data + parity chunks enable reconstruction even after loss |
| 8 | [pss-messaging.html](animations/pss-messaging.html) | Trojan chunks carry encrypted messages to target neighbourhoods |
| 9 | [act-access-control.html](animations/act-access-control.html) | Access Control Trie manages encryption keys for granting/revoking access |
| 10 | [feeds.html](animations/feeds.html) | Single-owner chunks form updateable pointers to latest content |

## How to view

Open any HTML file directly in a browser:

```bash
open animations/topology-left-right.html
```

Each animation runs at a fixed canvas resolution of 1280x680 and scales responsively.

## Visual identity

All animations follow a consistent design language:

- **Background:** `#272727`
- **Orange (accent):** `#d4633a`
- **White (lines/labels):** `#ffffff`
- **Dark (inner fills):** `#191919`

Nodes use orange outer rings with dark inner circles and white strokes. Motion and colour carry meaning — packet size, speed, and timing communicate protocol behaviour.

## Reference material

Animations are grounded in the protocol mechanics described in [The Book of Swarm](https://github.com/ethersphere/the-book-of-swarm).

## License

MIT
