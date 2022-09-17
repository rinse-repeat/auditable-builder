# rust-auditable-builder
Build Rust Projects with Cargo-Auditable

e.g. w/ matrix.json
```json
[
    {"rust-target": "x86_64-pc-windows-gnu"     , "os": "windows-latest" , "rust": "stable"},
    {"rust-target": "x86_64-pc-windows-msvc"    , "os": "windows-latest" , "rust": "stable"},
    {"rust-target": "i686-pc-windows-gnu"       , "os": "windows-latest" , "rust": "stable"},
    {"rust-target": "i686-pc-windows-msvc"      , "os": "windows-latest" , "rust": "stable"},
    {"rust-target": "aarch64-pc-windows-gnu"    , "os": "windows-latest" , "rust": "stable"},
    {"rust-target": "aarch64-pc-windows-msvc"   , "os": "windows-latest" , "rust": "stable"},
    {"rust-target": "armv7-unknown-linux-gnueabihf" , "os": "ubuntu-latest"  , "rust": "stable"},
    {"rust-target": "armv7-unknown-linux-musleabihf", "os": "ubuntu-latest"  , "rust": "stable"},
    {"rust-target": "arm-unknown-linux-gnueabihf"   , "os": "ubuntu-latest"  , "rust": "stable"},
    {"rust-target": "armv7-unknown-linux-musleabihf", "os": "ubuntu-latest"  , "rust": "stable"},
    {"rust-target": "x86_64-unknown-linux-gnu"  , "os": "ubuntu-latest"  , "rust": "stable"},
    {"rust-target": "aarch64-unknown-linux-gnu" , "os": "ubuntu-latest"  , "rust": "stable"},
    {"rust-target": "x86_64-apple-darwin"       , "os": "macos-latest"   , "rust": "stable"},
    {"rust-target": "aarch64-apple-darwin"      , "os": "macos-latest"   , "rust": "stable"}
]
```

## Use it

```yaml
name: "Some Release Builder"

on:
  workflow_dispatch:
    inputs:
      repository:
        description: 'Input Repository'
        required: true
        type: string
        default: someorg/somerepository
      tag:
        description: 'Release tag'
        required: true
        type: string

jobs:

  matrix:
    name: Generate Matrix
    runs-on: ubuntu-latest
    outputs:
      matrix-json: ${{ steps.set-matrix.outputs.matrix }}
    steps:
      - uses: actions/checkout@v2
        with:
          path: release-repo
      - id: set-matrix
        run: |
          content=`cat ./release-repo/matrix.json`
          # the following lines are only required for multi line json
          content="${content//'%'/'%25'}"
          content="${content//$'\n'/'%0A'}"
          content="${content//$'\r'/'%0D'}"
          # end of optional handling for multi line json
          echo "::set-output name=matrix::$content"
  build:
    needs: [matrix]
    uses: rinse-repeat/rust-auditable-builder/.github/workflows/rust-builder.yml
    with:
      repository: ${{ github.event.inputs.repository }}
      rust-target: ${{ matrix.rust-target }}
      os: ${{ matrix.os }}
      rust: ${{ matrix.rust }}
      use-cache: true
      use-locked: false
      serial: AAB
      zig-version: 0.9.1
    strategy:
      fail-fast: false
      matrix:
        include: ${{ fromJson(needs.matrix.outputs.matrix-json) }}

  release:
    needs: [build]
    uses: rinse-repeat/rust-auditable-builder/.github/workflows/release-builder.yml
    with:
      repository: ${{ github.event.inputs.repository }}
      tag: ${{ github.event.inputs.tag }}
```