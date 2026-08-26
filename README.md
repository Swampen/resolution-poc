# Dependabot Yarn Resolutions Security Update POC

This repo is a minimal reproduction for Dependabot failing to create a security
update when the vulnerable transitive dependency is controlled by Yarn
`resolutions`.

## Shape of the reproduction

- `node-gyp@^9.0.0` depends transitively on `tar@^6.1.2`.
- `package.json` overrides that transitive dependency with:

  ```json
  "resolutions": {
    "tar": "^7.5.19"
  }
  ```

- `yarn.lock` resolves the override to `tar@7.5.19`.
- The PoC uses `tar` as the dependency Dependabot should update when a
  security advisory affects the resolved version.

## Expected Dependabot behavior

Dependabot should update `package.json#resolutions.tar` and `yarn.lock` so the
installed `tar` version is no longer vulnerable.

## Actual behavior seen in the real repo

Dependabot detects the vulnerable package and runs commands similar to:

```text
corepack yarn up -R tar --mode=skip-build
corepack yarn add tar@<patched-version> --mode=skip-build
corepack yarn dedupe tar --mode=skip-build
corepack yarn remove tar --mode=skip-build
```

The commands exit successfully, but the final repository state has no changed
files because Dependabot never updates `package.json#resolutions.tar`.

The job then fails with:

```text
Dependabot::NpmAndYarn::FileUpdater::NoChangeError
No files were updated! Package manager: yarn
```

## Native package manager behavior

Running this locally updates the lockfile as expected:

```sh
corepack yarn up -R tar --mode=skip-build
```
