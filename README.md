[](This file is part of bindgen-wrapper. It is subject to the license terms in the COPYRIGHT file found in the top-level directory of this distribution and at https://raw.githubusercontent.com/lemonrock/bindgen-wrapper/master/COPYRIGHT. No part of bindgen-wrapper, including this file, may be copied, modified, propagated, or distributed except according to the terms contained in the COPYRIGHT file.)
[](Copyright © 2016 The developers of bindgen-wrapper. See the COPYRIGHT file in the top-level directory of this distribution and at https://raw.githubusercontent.com/lemonrock/bindgen-wrapper/master/COPYRIGHT.)

# bindgen-wrapper

This small git module makes it easier to generate FFI bindings for rust using [bindgen]. is intended to be used as a git submodule inside a `-sys` module, to make it easier to work with [bindgen] on Mac OS X and with more complex FFI wrappers. It installs all required dependencies using `cargo`, and, on Mac OS X, `brew` (Homebrew), except for Rust, `cargo` and `brew` itself. As a final step, after generating the bindings, it checks they compile.

It checks for any essential dependencies by looking in the `PATH`; a standard Mac OS X `PATH` with an additional to find binaries installed by cargo should be sufficient.

As an example, check out [mbedtls-sys] on GitHub.

## Installation

At the terminal, do the following:-

```bash
# my-crate-repo should already contain a `.git` folder or file
cd my-crate-repo

mkdir -m 0755 -p tools
git submodule add https://github.com/lemonrock/bindgen-wrapper.git tools/bindgen-wrapper
git submodule update --init --recursive
ln -s tools/bindgen-wrapper/bindgen-wrapper

cd -
```

## Configuration

To use `bindgen-wrapper` we need to create some files.

At the terminal, do the following:-
```bash
# my-crate-repo should already contain a `.git` folder or file
cd my-crate-repo

mkdir -m 0755 bindgen-wrapper.conf.d

# Place any header (*.h) files in here that add to or replace ones shipped by your library
mkdir -m 0755 bindgen-wrapper.conf.d/header-overrides

# Rust code snippet prepended to bindgen output. Add crate-level attributes, copyright statements, etc, here
touch bindgen-wrapper.conf.d/preamble.rs

# Rust code snippet interjected between `use` statements and remainder of generated code. Place additional `use` statements here
touch bindgen-wrapper.conf.d/post-includes.rs

# General configuation (does not need to executable)
touch bindgen-wrapper.conf.d/configuration.sh
```

See [mbedtls-sys] for an example of `configuration.sh`. As a minimum, you should define `bindingsName`, `rootIncludeFileName` and `link`. `link` is a space-separated list of lib names (on Unix systems, omit any `lib` prefix, eg `libmbedtls` is `mbedtls`). The functions `postprocess_after_generation` and `postprocess_after_rustfmt` default to empty. The statement `bindgen_wrapper_addTacFallbackIfNotPresent` is only necessary if either `postprocess_after_generation` or `postprocess_after_rustfmt` need to use the `tac` binary. The values `macosXHomebrewPackageName` and `alpineLinuxPackageName` (if known) can be set to a space-separated list of packages to install as prerequisites, perhaps containing header files. `headersFolderPath` can be used to specify a repository-local or unusual location for headers; it defaults to `/usr/include/$bindingsName` on Linux and `$(brew --prefix)/include/$bindingsName` on Mac OS X.

The following read-only variables are available to `configuration.sh`:-

* `homeFolder` - the root of the git repository, ie `tools/bindgen/../..`.
* `configurationFolderPath` - the parent folder containing `configuration.sh`

These values may not be absolute. Do not `cd` inside `configuration.sh`. The [mbedtls-sys] `configuration.sh` uses `homeFolder` to find a local copy of the mbedtls source code included as a git submodule.

## Extras

* On Mac OS X, a shell function compatible version of `tac` is available inside your `configuration.sh`. Do `bindgen_wrapper_addTacFallbackIfNotPresent` outside of any function. See [mbedtls-sys] for an example.
* On Mac OS X, `bindgen-macosx` can be used standalone instead of `bindgen`; it sets paths correctly for use with `brew`. However, you'll have to pass `-- -U__BLOCKS__` (an option to clang) to get it to work on El Capitan if any of the header files you use in generation have `#include <stdlib.h>` in them (directly or indirectly via their includes).


## Known Issues

* This wrapper is untested on anything but Mac OS X El Capitan, but with modification, should work on Alpine Linux, Debian-derivatives and Red Hat derivatives
* `sed` is somewhat broken on Mac OS X, and we try to work around it.
* On Mac OS X, if you already installed `llvm` with `brew` make sure you have installed it as `brew install --shared-`


[mbedtls-sys]: https://github.com/lemonrock/mbedtls-sys "mbedtls-sys GitHub page"
[bindgen]: https://github.com/Yamakaky/bindgen "bindgen GitHub page"
