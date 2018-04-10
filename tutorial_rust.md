# Seasoned Software: Rust tutorial

## Prerequisites
Before beginning on this tutorial you will need a Rust project with at least one fuzz-test that can be used with Cargo-fuzz.

If you need to add a fuzz-test to an existing project, please refer to the [Cargo-fuzz tutorial](https://rust-fuzz.github.io/book/cargo-fuzz/tutorial.html).

## Building your fuzz-test with Seasoned Software
For your fuzz-test to run inside the Seasoned Software service, the service first needs to know how to build it.

You can tell it, by adding a build script named `seasoned-software.sh` to the root of your repository.

This script should:
1. Download and build needed dependencies.
1. Use the provided Clang compiler and Cargo-fuzz to build the code and the relevant fuzz-tests.
1. Invoke our `register-binary` command-line utility to register the resulting LibFuzzer binaries.

For example, a `seasoned-software.sh` script may look like:
```
#!/bin/bash

# Cargo-fuzz needs nightly rust, so switch this project to nightly
rustup override set nightly

# Build the quicksort fuzz target in release mode by asking it to print its help menu
cargo fuzz run quicksort --release -- -help=1

# Find the executable for the fuzz target
EXE="$(find fuzz/target -iname quicksort -executable)"

# Register the fuzz target
register-binary "quicksort" "$EXE"
```

In this case, the Cargo-fuzz test name is `quicksort` and we choose to register this binary as the new binary version of the testing harness by the same name.

For now, you can think of a testing harness as the logical group for the test, whereas the code we just build is the latest binary version of it (something we can run).

## Setting up a testing harness
Once your fuzz-test is ready to be build and registered by the service, we need to create a new project on the service website, so it knows wherefrom to pull our code.

When logged in on the website, click the "Create project" option to the upper right.

1. Pick the Github repository from the dropdown list
1. Enter the git branch name that contains the fuzz-test (e.g. `master` or `fuzzing`)
1. Enter the Harness name used in the build script (in our case `quicksort`)
1. Click "Create"

It should now be a matter of time before your project is build and you should start to see something on its status page.

## Adding more test harnesses later
If you need to add further additional test harnesses later by:
1. On the status page for the project, enter a new harness name and click "Add"
2. Update your build script to also build and register the new fuzz-test with the chosen harness name.
3. The next time the service builds your code, it will add fuzz-tests for the new harnesses and they will start running.

## Discovering crashes
If one of your test harnesses experiences a reproducible crash, it will show up on its status page under the relevant harness.

Until then, the harness will include the text "No crashes yet".
