# Easy VRT

Easy VRT is a GitHub Action that simplifies visual regression testing by providing a workflow template.

## Usage

Example workflow:

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
      - uses: yorifuji/easy-vrt@v1
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

      - uses: subosito/flutter-action@v2

      - run: |
          flutter pub get
          flutter test --update-goldens --tags=golden

      - uses: yorifuji/easy-vrt@v1
        with:
          mode: expected
          expected-dir: test/golden_test/goldens
          expected-cache-key: ${{ needs.lookup.outputs.expected-sha }}

  actual:
    if: ${{ !cancelled() && !failure() && needs.lookup.outputs.actual-cache-hit != 'true' }}
    needs: lookup
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ needs.lookup.outputs.actual-sha }}

      - uses: subosito/flutter-action@v2

      - run: |
          flutter pub get
          flutter test --update-goldens --tags=golden

      - uses: yorifuji/easy-vrt@v1
        with:
          mode: actual
          actual-dir: test/golden_test/goldens
          actual-cache-key: ${{ needs.lookup.outputs.actual-sha }}

  compare:
    if: ${{ !cancelled() && !failure() }}
    needs: [lookup, expected, actual]
    runs-on: ubuntu-latest
    steps:
      - uses: yorifuji/easy-vrt@v1
        with:
          mode: compare
          expected-cache-key: ${{ needs.lookup.outputs.expected-sha }}
          actual-cache-key: ${{ needs.lookup.outputs.actual-sha }}
```

### Example for Flutter Applications

This section demonstrates how to adapt Easy VRT for Flutter applications, highlighting necessary changes in the workflow.

expected

```diff
-      # >>> add step to create expected image
+      - uses: subosito/flutter-action@v2

-      # <<< add step to create expected image
+      - run: |
+          flutter pub get
+          flutter test --update-goldens --tags=golden

       - uses: yorifuji/easy-vrt@v1
         with:
           mode: expected
-          expected-dir: your-expected-image-dir # set the directory where the expected image is stored
+          expected-dir: test/golden_test/goldens
           expected-cache-key: ${{ needs.lookup.outputs.expected-sha }}
```

actual

```diff
-      # >>> add step to create actual image
+      - uses: subosito/flutter-action@v2

-      # <<< add step to create actual image
+      - run: |
+          flutter pub get
+          flutter test --update-goldens --tags=golden

       - uses: yorifuji/easy-vrt@v1
         with:
           mode: actual
-          actual-dir: your-actual-image-dir # set the directory where the actual image is stored
+          actual-dir: test/golden_test/goldens
           actual-cache-key: ${{ needs.lookup.outputs.actual-sha }}
```

## Actions

### Inputs

```yaml
inputs:
  mode:
    description: "Mode, [lookup, expected, actual, compare]"
    required: true
  expected-dir:
    description: "Expected directory"
    required: false
  expected-cache-key:
    description: "Cache key for expected"
    required: false
  actual-dir:
    description: "Actual directory"
    required: false
  actual-cache-key:
    description: "Cache key for actual"
    required: false
  summary-comment:
    description: "If true, add a summary comment on workflow summary"
    required: false
    default: false
  review-comment:
    description: "If true, add a review comment on PR review (pull-requests permissions required)"
    required: false
    default: false
```

## Outputs

### Artifact

<img width="767" src="https://github.com/yorifuji/easy-vrt/assets/583917/0735cde4-3296-480b-83cf-ebcbc3a5989d">

### Report(In Artifact)

<img width="750" src="https://github.com/yorifuji/easy-vrt/assets/583917/8c5fe36b-6ef8-4977-b98a-96c6f134a88d">

### Summary(Optional)

<img width="766" src="https://github.com/yorifuji/easy-vrt/assets/583917/767a129e-bf9d-44e1-80f8-ae68bd4af07d">

### Pull Request Review Comment(Optional)

<img width="753"  src="https://github.com/yorifuji/easy-vrt/assets/583917/8dba42ed-5a77-4363-9993-0e9877dd9ccc">

## License

### [reg-cli](https://github.com/reg-viz/reg-cli)

The MIT License (MIT)

Copyright (c) 2017 bokuweb

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

### Easy VRT

MIT License
