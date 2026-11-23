# Signalrail

Signalrail is a proposed Linux desktop panel for one operator managing terminal-based coding agents on an Omarchy workstation and a headless VPS over SSH. It is in the design stage: there is no application to install yet.

- [Architecture and release scope](ARCHITECTURE.md)
- [Product blueprint](docs/product-blueprint.md) and [static screen board](docs/screen-board.html) (open the HTML file locally)
- [First implementation ticket and acceptance gates](docs/first-release.md)

The behavioral reference is [Reflex](https://github.com/monotykamary/reflex). The [existing Omarchy port](https://github.com/ryan-brosas/reflex/tree/6d9857fbb7fc326f98ba624f7ea51f88e15fa6bc/linux) is a comparison target, not a verified drop-in backend. The first implementation ticket tests whether a remote session can be identified safely after reconnecting.
