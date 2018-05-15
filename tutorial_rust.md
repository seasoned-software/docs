# Seasoned Software: Rust tutorial

## Prerequisites

Before beginning on this tutorial you will need a Rust project with at least one fuzz-test that can be used with Cargo-fuzz.

If you need to add a fuzz-test to an existing project, please refer to the [Cargo-fuzz tutorial](https://rust-fuzz.github.io/book/cargo-fuzz/tutorial.html).

## Building your fuzz-tests with Seasoned Software

For your fuzz-test to run inside the Seasoned Software service, the service needs to know how to build the fuzz targets for your project. You let it know by adding a build script named `seasoned-software.sh` to the root of your repository.

This script should:
1. Download and build needed dependencies not manged by `cargo` (if any).
1. Use the provided rust and Clang compilers to build the code and the relevant fuzz-tests.
1. Invoke the `upload-binary` command-line utility available on our build machines to upload the fuzz-test binaries into our service.

For example, a `seasoned-software.sh` script may look like:

```.sh
#!/bin/bash

# Cargo-fuzz needs nightly rust, so switch this project to nightly
rustup override set nightly

# Build the fuzz target target_1 in release mode by asking it to print its help menu
cargo fuzz run target_1 --release -- -help=1

# Find the executable for the fuzz target
EXE="$(find fuzz/target -iname target_1 -executable)"

# Upload the executable for the fuzz target
upload-binary "$EXE"
```

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
