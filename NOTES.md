# skyrim-rs-reversed-z (local fork work)

- Upstream: https://github.com/TwistedModding/skyrim-rs-reversed-z (MPL-2.0). Local branch `crossover-d3d11`; `origin` not set yet (fork denied by the classifier 2026-11-13; user runs `gh repo fork`). `extern/skyrim-rs` is a linked worktree of `tools/skyrim-rs` at v1.0.23.
- Problem on CrossOver (D3DMetal): the plugin turned the depth buffer to reversed Z but rewrote no shaders, so the world broke.
  1. `D3D11CreateDevice(WARP)` on the system `d3d11.dll` fails (0x80070057), so the early shader-creation hook never installed and capture ran after the game had built its shaders (420,682 bytecode misses).
  2. After 1 was fixed, `GetPrivateData` on D3DMetal shader objects fails even right after a successful `SetPrivateData` (0x80070057), so stored bytecode never came back (563,410 misses, 0 twins).
- Changes (`reversed-z/src/shader.rs`): log why the system device fails; fall back to hooking the loaded `d3d11.dll`'s `D3D11CreateDevice*`; keep bytecode in a pointer-keyed `STORE` (holds a reference per shader) when private data is not returned; log a capture readback check.
- Result 2026-11-13, build 1.0.25 (`dist/reversed-z-1.0.25-crossover.zip`, cross-built with `scripts/build-macos.sh`): `ReversedZ` on, bytecode misses 0, 77 vertex twins and 4 pixel twins after a few minutes, no log errors, world renders normally (user check). Not checked: far-LOD z-fighting before/after, water, Community Shaders features beyond what is in the list.
- Known cost: `STORE` keeps all ~19.6k shaders and their bytecode alive (tens of MB); the private data twin/constants steps (`CONSTANTS`, `TWIN`) also use `SetPrivateData` and were not shown to fail, but were not checked either.
