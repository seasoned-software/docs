# Seasoned Software: Rust tutorial

## Prerequisites

Before beginning on this tutorial you will need a Rust project with at least one fuzz-test that can be used with Cargo-fuzz.

If you need to add a fuzz-test to an existing project, please refer to the [Cargo-fuzz tutorial](https://rust-fuzz.github.io/book/cargo-fuzz).

## Building your fuzz-tests with Seasoned Software

For your fuzz-test to run inside the Seasoned Software service, the service needs to know how to build the fuzz targets for your project. You let it know by adding a build script named `seasoned-software.sh` to the root of your repository.

This script should:
1. Download and build needed build-dependencies not manged by `cargo` (if any).
1. Use the provided rust and Clang compilers to build the code and the relevant fuzz-tests.
1. Invoke the `upload-binary` command-line utility available on our build machines to upload the fuzz-test binaries into our service.

For example, a `seasoned-software.sh` script may look like:

```.sh
#!/bin/bash

# Cargo-fuzz needs nightly rust, so switch this project to nightly
rustup override set nightly

# Name of the fuzz target
FUZZ_TARGET=target_1

# Build the fuzz target in release mode by asking it to print its help menu
cargo fuzz run $FUZZ_TARGET --release -- -runs=0

# Find the executable for the fuzz target
EXE="$(find fuzz/target -iname $FUZZ_TARGET -executable)"

# Upload the executable for the fuzz target
upload-binary "$EXE"
```
## Build and runtime dependencies

If needed, you can get us to install system libraries for use during building or when the fuzzer is running later on. You can ask for packages available in the `apt-get` package manager.

To use this feature, you add an extra config file named `seasoned-software.yaml` to your repository, next to your `seasoned-software.sh` build script.

Here is an example config file that asks for installation the sqlite3 and libsqlite3-dev packages:
```sh
install:
  apt-get:
  - sqlite3
  - libsqlite3-dev
```

These packages are installed before running the `seasoned-software.sh` build script and again before running any of the resulting fuzz-test binaries.

## Testing your configs locally

You can test your `seasoned-software.sh` build script and `seasoned-software.yaml` config locally using our provided [docker build image](https://hub.docker.com/r/seasonedsoftware/builder/). You will need a modern docker installation (we tested it with 18.03.1-ce).

Inside your code repository (where the `seasoned-software.sh` build script lives) run:
```sh
wget https://raw.githubusercontent.com/seasoned-software/docs/master/local-checks/SeasonedSoftwareDockerfile
docker build -f SeasonedSoftwareDockerfile .
```

If the image builds succesfully, your config is likely working correctly. You can check that you get a line similar to `/binaries/YOUR_BINARY: OK` for every one of your binaries, at the end of the build.

## Setting up the Seasoned Software project

Once your fuzz-tests are ready to be build and uploaded into the service, you need to create a new project on the service website, so it knows where to fetch your code from.

When logged in on the website on <https://app.seasoned.software>, click the "Create project" option to the upper right.

1. Pick the Github repository from the dropdown list
1. Enter the git branch name that contains the fuzz-test (e.g., `master` or `fuzzing`)
1. Click "Create"

It may take a few minutes before the project is configured and the first build started. Once the first build starts, you can see it on the project status page. Click the build to follow it and see whether it succeeds.

## Adding more fuzz-tests later

If you need to add additional fuzz-tests later, simply update the project's `seasoned-software.sh` build script to upload more fuzz-test binaries using the provided `upload-binary` tool.

## Discovering crashes with continuous integration
Once your project is setup correctly, the service will automatically fetch each new commit, build it and run your project's regression tests against it. The project status page shows recent builds and their testing result, and you can use this page to see whether a new commit introduced or resolved an issue.
