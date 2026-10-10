# Muguet
A native Mach-O iOS/macOS port of GTA V, with multiplayer

## How does this work ?
this whole port relies on the WebAssembly compilation of the game. in a browser, that `game.wasm` runs inside the JS engine, behind WebGPU, Web Audio, web workers...
Muguet throws that layer away and runs the same `game.wasm` as a regular native app, with [wasmtime](https://wasmtime.dev) for the code and [wgpu](https://wgpu.rs) on Metal for the graphics

On macOS, the game code is compiled to native arm64 the first time you launch it, then cached. On iOS, we can't properly do that (i dont rlly want you to deal with the hassle of enabling JIT), so the game code is compiled when you build the app and baked into it

On top of that, Muguet adds a small UDP multiplayer (for other people who have the same port to play with you)

## Game files
Because I care about me not getting sued, you will need your own copy as a `.zip` : the folder holding `data/` and `b/<build>/` (with `game.wasm` and `shaders/`)

Muguet itself will then asks for that zip

## Requirements
- Apple Silicon Mac, macOS 14 or later
- Rust toolchain
- Xcode 26 or later

For iOS, also :
- `rustup` with `rustup toolchain install 1.98.1` and `rustup target add aarch64-apple-ios --toolchain 1.98.1`
- `xcodegen` (`brew install xcodegen`)
- an Apple developer account
- a device on iOS 15 or later, ideally 8 GB of RAM

## Building
macOS :
```
./make-app.sh
```
gives `build/Muguet.app`

iOS :
```
TEAM=<your team id> ./make-ios.sh game.zip --install
```
`--ipa` makes `build/Muguet.ipa` instead. With a free account, add `FREE=1` (+ a `BUNDLE_ID` of your own)

## Multiplayer
One player hosts from the launcher (or runs `muguet-server` on any machine), the others join with that address. UDP port 47520

## Limitations
- iOS needs the increased memory limit entitlement
- Minimap doesn't show
- Story missions are single player only


Contributions are welcome !

## Build an IPA from GitHub (no Mac/PC needed)

The `.github/workflows/build-ipa.yml` workflow uses a GitHub-hosted Apple Silicon macOS runner with Xcode, Rust, and `xcodegen`. You can start it from GitHub's website on an iPad and download `muguet-ios-ipa` from the run's **Artifacts** section.

1. Merge the workflow into the default branch so GitHub can offer it under **Actions**.
2. Open **Settings → Secrets and variables → Actions** and add:
   - `MUGUET_GAME_ZIP_URL`: an HTTPS download URL for your own authorized game ZIP. Keep the URL private if it contains a signed token.
   - `MUGUET_GAME_ZIP_TOKEN` (optional): a bearer token if your file host requires one.
3. Keep the game ZIP in private storage that you control. Do not commit it to this repository or upload it as a workflow artifact.
4. On the **Actions** tab, select **Build iOS IPA → Run workflow**. Set **include_game_code** to true to precompile the private ZIP into the app; otherwise the workflow builds a packaging-only IPA without game code.
5. When the run finishes, open it and download the `muguet-ios-ipa` artifact. It expires after seven days.

The ZIP must contain `b/<build>/game.wasm`. The workflow validates the ZIP layout and then calls the repository's existing `make-ios.sh`; it does not replace Muguet's engine build steps.

**Important:** GitHub Actions creates an unsigned/ad-hoc sideload IPA, not an App Store-signed app. Installing it on an iPad still requires a compatible sideloading method. The first build may fail if the upstream Rust toolchain, Wasmtime precompile flags, Xcode project, or iOS target no longer match; inspect the run logs before assuming the IPA is ready to play. The no-game-code mode is only a build/packaging smoke test.
