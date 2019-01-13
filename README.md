# The Python Launcher for UNIX

An implementation of the `py` command for UNIX-based platforms.

The goal is to have `py` become the cross-platform command that all Python users
use when executing a Python interpreter. By having a version-agnostic command
it side-steps the "what should the `python` command point to?" debate by
clearly specifying that upfront (i.e. the newest version of Python that is
installed). This also unifies the suggested command to document for launching
Python on both Windows as UNIX as `py` which has existed as the preferred
[command on Windows](https://docs.python.org/3/using/windows.html#launcher) for
some time.

See the top of `py --help` for instructions.

# Search order

Please note that while searching, the search for a Python version can become
more specific. This leads to a switch in the search algorithm to the one most
appropriate to the specificity of the version.

## `py -3.6` (specific version)
1. Search `PATH` for `python3.6`

## `py -3` (loose/major version)
1. Use the version found in the `PY_PYTHON3` environment variable if defined
   (e.g. `PY_PYTHON3=3.6`)
1. Search `PATH` for all instances of `python3.Y`
1. Find the executable with largest `Y`

## `py` (any/unknown version)
1. Use `${VIRTUAL_ENV}/bin/python` immediately if available
1. If the first argument is a file path ...
   1. Check for a shebang
   1. If executable starts with `/usr/bin/python`, `/usr/local/bin/python`,
      `/usr/bin/env python` or `python`, proceed based on the version found
      (bare `python` is considered `python2` for backwards-compatibility)
1. Use the version found in the `PY_PYTHON` environment variable if defined
   (e.g. `PY_PYTHON=3` or `PY_PYTHON=3.6`)
1. Search `PATH` for all instances of `pythonX.Y`
1. Find the executable with the largest `X.Y`

# TODO

**NOTE**: I am using this project to learn
[Rust](https://www.rust-lang.org/), so please don't be offended if I choose to
implement something myself instead of accepting a pull request that you submit.
(Pull requests to do something I have already implemented in a more idiomatic
fashion are very much appreciated, though.)

[PEP 397: Python launcher for Windows](https://www.python.org/dev/peps/pep-0397/) ([documentation](https://docs.python.org/3/using/windows.html#launcher))

## Functionality
1. `py --list`
  - Output column format like `pip list` (based on a [Twitter poll](https://twitter.com/brettsky/status/1066795161236062208))
  - Skipping `py -0`/`py -0p`/`py --list-paths` for simplicity
1. [Configuration files](https://www.python.org/dev/peps/pep-0397/#configuration-file)
  - [Customized commands](https://www.python.org/dev/peps/pep-0397/#customized-commands)
  - Want a better format like TOML?
  - Probably want a way to override/specify things, e.g. wanting a framework build on macOS somehow
    - Aliasing? E.g. `2.7-framework` for `/System/Library/Frameworks/Python.framework/Versions/2.7/Resources/Python.app/Contents/MacOS/Python`?
    - Just provide a way to specify a specific interpreter for a specific version? E.g. `2.7` for `/System/Library/Frameworks/Python.framework/Versions/2.7/Resources/Python.app/Contents/MacOS/Python`
    - What about implementations that don't install to e.g. `python3.7` like `pypy3`?
  - How should config file search work?
    - Pre-defined locations?
    - Walk up from current directory?
    - [XDG base directory specification](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html)?
1. Windows support
  - Registry
  - `PATH`
  - Read `../pyvenv.cfg` to resolve for `Any` version
    - Acts as a heavyweight "symlink" to the Python executable for the virtual environment
    - Speeds up environment creation by not having to copy over entire Python installation (e.g. `.pyd` files)
1. Provide a `pylauncher` package (it will make the pipenv developers happy 😃)

## Polish
1. Provide an error message when no Python executable is found (and be helpful based on requested version)
1. Start using [`human-panic`](https://github.com/rust-clique/human-panic)
1. Man page?
1. [`PYLAUNCH_DEBUG`](https://docs.python.org/3.8/using/windows.html#diagnostics)?

## Maintainability
1. Move code out of `main.rs` and into `lib.rs` to facilitate testing
   - Function to check for the proper environment variable based on the requested version
   - Function to find all compatible executables on PATH
   - Change split_shebang() to return a Result (Python file shouldn't have a non-Python shebang)
   - Change version_from_flag() to return a Result (shouldn't be parsing a flag w/o a `-`)
1. Consider dropping `nix` dependency for a straight [`libc`](https://crates.io/crates/libc) dependency

