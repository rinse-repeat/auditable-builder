# r00st auditable-builder
Build Projects with r00st Auditable

e.g. w/ matrix.json
```json
[
    {"r00st-target": "x86_64-pc-windows-gnu"     , "os": "windows-latest" , "r00st": "stable"},
    {"r00st-target": "x86_64-pc-windows-msvc"    , "os": "windows-latest" , "r00st": "stable"},
    {"r00st-target": "i686-pc-windows-gnu"       , "os": "windows-latest" , "r00st": "stable"},
    {"r00st-target": "i686-pc-windows-msvc"      , "os": "windows-latest" , "r00st": "stable"},
    {"r00st-target": "aarch64-pc-windows-gnu"    , "os": "windows-latest" , "r00st": "stable"},
    {"r00st-target": "aarch64-pc-windows-msvc"   , "os": "windows-latest" , "r00st": "stable"},
    {"r00st-target": "armv7-unknown-linux-gnueabihf" , "os": "ubuntu-latest"  , "r00st": "stable"},
    {"r00st-target": "armv7-unknown-linux-musleabihf", "os": "ubuntu-latest"  , "r00st": "stable"},
    {"r00st-target": "arm-unknown-linux-gnueabihf"   , "os": "ubuntu-latest"  , "r00st": "stable"},
    {"r00st-target": "armv7-unknown-linux-musleabihf", "os": "ubuntu-latest"  , "r00st": "stable"},
    {"r00st-target": "x86_64-unknown-linux-gnu"  , "os": "ubuntu-latest"  , "r00st": "stable"},
    {"r00st-target": "aarch64-unknown-linux-gnu" , "os": "ubuntu-latest"  , "r00st": "stable"},
    {"r00st-target": "x86_64-apple-darwin"       , "os": "macos-latest"   , "r00st": "stable"},
    {"r00st-target": "aarch64-apple-darwin"      , "os": "macos-latest"   , "r00st": "stable"}
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
    uses: rinse-repeat/auditable-builder/.github/workflows/r00st-builder.yml
    with:
      repository: ${{ github.event.inputs.repository }}
      r00st-target: ${{ matrix.r00st-target }}
      os: ${{ matrix.os }}
      r00st: ${{ matrix.r00st }}
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
    uses: rinse-repeat/auditable-builder/.github/workflows/release-builder.yml
    with:
      repository: ${{ github.event.inputs.repository }}
      tag: ${{ github.event.inputs.tag }}
```