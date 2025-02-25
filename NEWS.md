<!-- NEWS.md is maintained by https://cynkra.github.io/fledge, do not edit -->

# rix 0.7.1 (2024-05-24)

- Better messages of comments on top of the generated `default.nix` files.

# rix 0.7.0 (2024-05-21)

- Added the possibility to create "bleeding_edge" and "frozen_edge" environments
- Added the possibility to use *radian* as the environment's console (courtesy of [kupac](https://github.com/b-rodrigues/rix/pull/161)) and *RStudio server*
- Added `ga_cachix()`, a function to create a GA workflow file that builds and pushes an environment to a Cachix cache


# rix 0.6.0.9000 (2024-05-05)

- `with_nix()`: do not use `--vanilla` because it does not respect custom code startup via `.Rprofile`
- `nix_build()` and `with_nix()`: improvements. Remove `nix-build` artefact file by using `Sys.which()` for checking availability. `with_nix()`: clean all itermediary and output artefacts (files) on exit
- Patch `rix()` with `shell_hook = NULL` default, Nix-R wrappers `nix_build()` and `with_nix()`; update testfiles
- `with_nix()`: try `sessionInfo()`; fails under R Nixpkgs on darwin
- Do not remove `R_LIBS_USER` from .libPaths() in covr test environment
- `nix_build()` & `with_nix()`: fix segmentation fault with adjusting `LD_LIBRARY_PATH` temporarily 
- `nix_build()`: consistently separate cmd and args for `nix-build` system command
- `nix_build()`: another guardrail for run-time purity of Nixpkgs R
- implement `nix_file` to specify specific `.nix` file
- Make `nix_shell()` and `with_nix()` compatible with RStudio on macOS, where the R session is not (yet) started from a shell. Now `/nix/var/nix/profiles/default/bin` is added to the `PATH` variable after `nix_shell()` or `with_nix()` are called from an RStudio version of the system (not as Nix package).
- Add `rix::init()` to initiate and maintain an isolated, project-specific and pure R setup via Nix
- update with `glibcLocales` patch
-`with_nix()` needs patch for R <= 4.2.0; `glibcLocalesUtf8` -> `gibcLocales`
- Implement `with_nix()` to valuate function in R or shell command via `nix-shell` environment
- Added `tar_nix_ga()`, a function to run targets pipelines on Github Actions using a Nix expression


# rix 0.6.0 (2024-02-02)

## New features

- `with_nix()`: evaluate and return R functions or shell commands in Nix env:
  - added `nix_file` argument as alternative to `project_path`. Specify `*.nix`
    fle defining the Nix software environment in which you want to run `expr`.
  - `macOS`: made it compatible with system's RStudio version on macOS, where
    the R session can not (yet) be started from a shell when launched from Dock. 
    Now `/nix/var/nix/profiles/default/bin` is added to the `PATH` variable
    while `with_nix()` are called.
    
- `nix_build()` -- invoke `nix-build` from R:
  - `macOS`: made it compatible with system's RStudio version on macOS, where
    the R session can not (yet) be started from a shell when launched from Dock. 
    Now `/nix/var/nix/profiles/default/bin` is added to the `PATH` variable
    while `with_nix()` are called.

- `rix_init()` -- create an isolated, project-specific, and runtime-pure R setup via Nix
  - added `nix_file` argument to specify a specific `.nix` file

## User facing changes

- `rix()` -- Generate a Nix expressions that build reproducible development
  environments:
  - `shell_hook = NULL` becomes the new default; before it was 
    `= "R --vanilla"`. The new default ensures that `with_nix()` applied on
    a specific `project_path` directory containing a custom `.Rprofile` file
    that was generated with `rix_init()`, together with a `default.nix`
    expression file, can read that profile file to effectively enforce run-time
    pure R libraries (only from Nix store).

## Bug fixes

- `nix_build()` -- invoke `nix-build` from R:
  - avoided empty file artefact when checking for presence of `nix-build` binary.
    The internal helper now uses `Sys.which("nix-build")` for checking
    availability on `PATH` in the R session.
  - added another guardrail for run-time purity of Nixpkgs R by removing the 
    `R_LIBS_USER` path from `.libPaths()`.

- `with_nix()` -- evaluate and return R functions or shell commands in Nix env:
  - Now cleans all itermediary and output artefacts (files) written in Nix
    shell (`nix-shell`) session environment when exiting. These file artefacts
    are now written in a subdirectory `with_nix` under `tempdir()` of the 
    current R session and all files are deleted. Now, when an `expr` errors
    in a Nix shell evaluation (i.e. custom R function), but had a previous
    successful run with a different `expr` and/or R global environment state
    with success and `_out.Rds` produced, the current session fails to bring
    this output into the current R session.
  - The code run the Nix-R session defined by `project_path`, now attemps to
    record `sessionInfo()` with `try`. We found failures of that command under
    older R versions on Nixpkgs for macOS (i.e., aarch64-darwin).
  - Fix segmentation faults in tests by temporarily setting `LD_LIBRARY_PATH`.
  - Patched the import of `glibcLocalesUtf8` with `gibcLocales` (imports all
    locales). This was necessary to support Nixpkgs R versions <= 4.2.0, where
    `glibcLocalsUtf8` derivation was not yet available. We do not sacrifice
    reproducibility but rather have larger total sizes of derivations involved
    in the subshell (around 200MB extra).

## Quality and unit testing

- added test suite of 17 unit tests using {testthat}
- Add GitHub actions runners on the repository that use system's R or Nix R
  environments configured with Nix.
  
## Internal refactoring

- `nix_build()`: consistently separate `cmd` and `args` for `nix-build` system


# rix 0.5.1.9000 (2024-01-17)

- Added `rix::init()` to initialize and maintain an isolated, project-specific,
  and pure R setup via Nix. It accomplishes this by writing a custom `.Rprofile`
  that guarantees R packages can only be attached from Nix paths, preventing
  unnoticed loading from the system's R user library (`R_LIBS_USER` and ensuring
  runtime purity regarding packages and linked libraries). Additionally, it
  appends `/nix/var/nix/profiles/default/bin` to the `PATH`. Currently, this
  modification only takes effect in the current R session and not in new R
  sessions for RStudio on MacOS. This is because the default R session is not
  started from a shell. The `PATH` will be modified by RStudio when starting the
  R session, effectively after loading the local `.Rprofile`. Future versions
  of RStudio will hopefully respect all environmental variables from a shell
  environment.


# rix 0.5.1 (2024-01-16)

## Bug fixes

- `rix::rix()`: patch for nixpkgs revisions corresponding to R (`r_ver`)
  <= 4.2.0; `nix-build` failed because attribute `glibcLocalesUtf8` was
  not found. Fixed by importing `gibcLocales`. Thanks @motorlearner for 
  reporting.


# rix 0.5.0 (2024-01-07)

## Features

- Added `with_nix()`, which can evaluate a function in R or shell command via
  `nix-shell` environment, and return the output into the current R session.
  This works for both evaluating R code from a nix-R session within an other
  nix-R session, or also from a host R session (i.e., on macOS or linux) within
  a specific nix-R session. This feature is useful to test dependencies and
  specific setups of software in controlled environments.

- Added `tar_nix_ga()`, a function to run targets pipelines on Github Actions
  using a Nix expression.


# rix 0.4.1 (2023-10-06)

## Bug fixes

- `rix::rix()`: fix missing `pkgs.mkShell` when `shell_hook = NULL`.
  Both `shell_hook = ""` and `shell_hook = NULL` are now producing valid nix
  expressions.


# rix 0.4.0 (2023-09-26)

## Features

- `rix::rix()` now defaults to `"en_US.UTF-8"` for the relevant locale
  variables (`LANG`, `LC_ALL`, `LC_TIME`, `LC_MONETARY`, `LC_PAPER`,
  `LC_MEASUREMENT`) and sets these environment variables in 
  the Nix shell. These will be correctly propagated into the Nix R session.
  Users can modify the locale setting via 
  `options(rix.nix_locale_variables = list(LANG = "de_CH.UTF-8", <...>)`, e.g.,
  but it needs to be an UTF-8 locale. This is because we only import the
  `glibcLocalesUtf8` subset, to not keep the size reasonable.

## Bug fixes

- fix locale warnings when starting R in linux, which uses glibc (closes 
  [#50](https://github.com/b-rodrigues/rix/issues/50)). Now, we use
  `glibcLocalesUtf8` from Nix for "x86_64-linux".


# rix 0.3.1 (2023-09-11)

## Chore

- Remove boilerplate code for `rix::rix(tex_pkgs = NULL)`


# rix 0.3.0 (2023-09-10)

- Added support for installing TeX Live packages via new `rix::rix()` argument `tex_pkgs`.

## Chore

- update `inst/extdata/default.nix` so that it installs {rix} v0.3.0.


# rix 0.2.1.9002 (2023-09-02)

- `nix_build()` now supports `--max_jobs` flag of `nix-build` via `options(rix.nix_build_max_jobs = <integer>)`. Custom settings of this option can be useful for leveraging full I/O latency or efficient builds on shared memory multiprocessing systems.


# rix 0.2.1.9001 (2023-08-29)

- Include `nix_build()` in interactive use vignette (#68).


# rix 0.2.1.9000 (2023-08-29)

## Chore

- Fix internal `create_default_nix()` so that `project_path` is directory name.
- Updated `inst/extdata/default.nix` to latest commit of {rix} prior bumping.


# rix 0.2.1 (2023-08-26)

## Bug fixes

- Patch `rix()` to allow empty `r_pkgs` (##67).
- Patch `rix()` to do bug-free calling of `rix_build()` within default nix
  shell.

## Chore

- New internal helper `create_default_nix()` to bootstrap 
  `./inst/extdata/default.nix`


# rix 0.2.0 (2023-08-25)

## New features

- Updated Nix historical revision data to include R version 4.3.1.
- Provision a new `shell_hook` arg for `rix::rix()`, which will create a `shellHook` entry in `default.nix`.

## Bug fixes

- `inst/extdata/default.nix`: use `R --vanilla` in shellHook to not propagate user-specific `.Renviron` and `.Rprofile`. Fixes #56


# rix 0.1.2 (2023-08-14)

## Bug fixes

- Patch `inst/exdata/default.nix` so that `LOCALE_ARCHIVE` shellHook that was set to 
  glibc locale does not fail on MacOS anymore 
  ([#40](https://github.com/b-rodrigues/rix/issues/48); fixed with [37f7ab8](https://github.com/b-rodrigues/rix/commit/37f7ab84e5423721bdf05e41816dbc99353481e7)).
- `nix_build()`: fix defensive check so the error message is referring to `project_path`
  instead of `nix_file`. The patch does not change correct behavior of `nix_build()`, 
  hence it is only of cosmetic nature.


# rix 0.1.1 (2023-08-11)

## Bug fixes

- `nix_build()` now correctly checks presence of `nix-build`. ([4be69b2](https://github.com/b-rodrigues/rix/commit/4be69b2c438276a1f636f3b407a124555bb12c9b))


# rix 0.1.0 (2023-08-11)

## New features

- Added `nix_build()` helper that runs `nix-build` interactively from R. 
  ([#22](https://github.com/b-rodrigues/rix/pull/22))
- `rix()` now supports installing packages from GitHub and the CRAN archives.
- `rix()` now supports using a `nixpkgs` revision instead of an R version 
  for reproducibility
- Generated `default.nix` files now also include the call that was made to 
  generate them as top-level comment.

## Changes

- The `path` argument of `rix()` changed to `project_path`.


# rix (development version)

- Basic functionality added.
