# Easy VRT

## Usage

Example workflow

```yaml
name: vrt

run-name: visual regression test

on: pull_request

permissions:
  contents: read

jobs:
  lookup:
    runs-on: ubuntu-latest
    outputs:
      actual-sha: ${{ steps.lookup.outputs.actual-sha }}
      actual-cache-hit: ${{ steps.lookup.outputs.actual-cache-hit }}
      expected-sha: ${{ steps.lookup.outputs.expected-sha }}
      expected-cache-hit: ${{ steps.lookup.outputs.expected-cache-hit }}
    steps:
      - uses: yorifuji/easy-vrt@main
        id: lookup
        with:
          mode: lookup

  expected:
    if: ${{ !cancelled() && !failure() && needs.lookup.outputs.expected-cache-hit != 'true' }}
    needs: lookup
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ needs.lookup.outputs.expected-sha }}

      # >>> create expected image

      # <<< create expected image

      - uses: yorifuji/easy-vrt@main
        with:
          mode: expected
          expected-dir: your-expected-image-dir
          expected-cache-key: ${{ needs.lookup.outputs.expected-sha }}

  actual:
    if: ${{ !cancelled() && !failure() && needs.lookup.outputs.actual-cache-hit != 'true' }}
    needs: lookup
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ needs.lookup.outputs.actual-sha }}

      # >>> create actual image

      # <<< create actual image

      - uses: yorifuji/easy-vrt@main
        with:
          mode: actual
          actual-dir: your-actual-image-dir
          actual-cache-key: ${{ needs.lookup.outputs.actual-sha }}

  compare:
    if: ${{ !cancelled() && !failure() }}
    needs: [lookup, expected, actual]
    runs-on: ubuntu-latest
    steps:
      - uses: yorifuji/easy-vrt@main
        with:
          mode: compare
          expected-cache-key: ${{ needs.lookup.outputs.expected-sha }}
          actual-cache-key: ${{ needs.lookup.outputs.actual-sha }}
```

## Features

## How it works

## Motivation

## Known issues

## License

MIT
