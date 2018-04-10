# Seasoned Software: Rust tutorial

## Prerequisites

Before beginning on this tutorial you will need a Rust project with at least one fuzz-test that can be used with Cargo-fuzz.

If you need to add a fuzz-test to an existing project, please refer to the [Cargo-fuzz tutorial](https://rust-fuzz.github.io/book/cargo-fuzz/tutorial.html).

## Building your fuzz-test harnesses with Seasoned Software

For your fuzz-test to run inside the Seasoned Software service, the service needs to know how to build the harnesses (fuzz targets in cargo-fuzz terminology) for your project.

Thus you must add a build script named `seasoned-software.sh` to the root of your repository.

This script should:
1. Download and build needed dependencies not manged by `cargo` (if any).
1. Use the provided rust and Clang compilers to build the code and the relevant fuzz-tests.
1. Invoke the `register-binary` command-line utility available on our build machines to register the fuzz-test harnesses.

For example, a `seasoned-software.sh` script may look like:

```.sh
#!/bin/bash

# Cargo-fuzz needs nightly rust, so switch this project to nightly
rustup override set nightly

# Build the fuzz target target_1 in release mode by asking it to print its help menu
cargo fuzz run target_1 --release -- -help=1

# Find the executable for the fuzz target
EXE="$(find fuzz/target -iname target_1 -executable)"

# Register the executable for the fuzz target under the name target_1
register-binary "target_1" "$EXE"
```

In this case, the Cargo-fuzz target name is `target_1` and we register the executable for this testing harness using the same name.

For now, you can think of a testing harness as the logical group for the test, whereas the code we just build is the latest binary version of it (something we can run).

## Setting up a testing harness

Once your fuzz-test is ready to be build and be registered by the service, you need to create a new project on the service website, so it knows where to get your code.

When logged in on the website on <https://app.seasoned.software>, click the "Create project" option to the upper right.

1. Pick the Github repository from the dropdown list
1. Enter the git branch name that contains the fuzz-test (e.g., `master` or `fuzzing`)
1. Enter the Harness name used in the build script (in our case `target_1`)
1. Click "Create"

It should now be a matter of time before your project is build and you should start to see something on the project status page.

## Adding more test harnesses later

If you need to add additional test harnesses later:
1. On the status page for the project, enter a new harness name and click "Add"
2. Update your build script to also build and register the new fuzz-test with the chosen harness name.
3. The next time the service builds your code, it will add fuzz-tests for the new harnesses and they will start running.

## Discovering crashes
If one of your test harnesses experiences a reproducible crash, it will show up on its status page under the relevant harness.

Until then, the harness will include the text "No crashes yet".
