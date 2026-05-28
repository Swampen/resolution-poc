# Dependabot Yarn Resolutions Security Update POC

This repo is a minimal reproduction for Dependabot failing to create a security
update when the vulnerable transitive dependency is controlled by Yarn
`resolutions`.

## Shape of the reproduction

- `external-editor@3.1.0` depends on `tmp@^0.0.33`.
- `package.json` overrides that transitive dependency with:

  ```json
  "resolutions": {
    "tmp": "^0.2.4"
  }
  ```

- `yarn.lock` is intentionally stale and currently resolves that override to
  vulnerable `tmp@0.2.5`.
- `tmp@0.2.6` and later are non-vulnerable for the relevant advisory.

## Expected Dependabot behavior

Dependabot should update the Yarn resolution/lockfile so the installed `tmp`
version is no longer vulnerable, for example by changing `yarn.lock` from
`tmp@0.2.5` to `tmp@0.2.7`.

## Actual behavior seen in the real repo

Dependabot detects the vulnerable package and runs commands similar to:

```text
corepack yarn up -R tmp --mode=skip-build
corepack yarn add tmp@0.2.7 --mode=skip-build
corepack yarn dedupe tmp --mode=skip-build
corepack yarn remove tmp --mode=skip-build
```

The commands exit successfully, but the final repository state has no changed
files because Dependabot never updates `package.json#resolutions.tmp`.

The job then fails with:

```text
Dependabot::NpmAndYarn::FileUpdater::NoChangeError
No files were updated! Package manager: yarn
```

## Native package manager behavior

Running this locally updates the lockfile as expected:

```sh
corepack yarn up -R tmp --mode=skip-build
```
