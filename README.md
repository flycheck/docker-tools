This repository contains the Dockerfiles used for running Flycheck's unit and
integration tests on Travis CI.

- `all-tools` creates a container (based on Ubuntu 18.04) installing all the
  tools that Flycheck supports.  The full image is around 4.5GB uncompressed.
- `emacs-cask` creates a container (still on Ubuntu 18.04) with Emacs, Cask, and
  Make installed.  This is used to run Flycheck's unit tests.  It is
  configurable: we can build with different Emacs versions.

These two images are scheduled to build monthly (or whenever there is a commit
to master), and uploaded to [flycheck/all-tools][hub:all-tools] and
[flycheck/emacs-cask][hub:emacs-cask] respectively.

There is a third image in [flycheck/flycheck][],
`all-tools-and-emacs-cask`. This one is never pushed anywhere; it is built and
used directly on Travis in order to run our integration tests.  See the
[Flycheck manual][manual:run-integ] on how to use that image to run the
integration tests locally.

## How to add a tool

Whenever a new checker is added to Flycheck, there should be at least one
integration test for it.  For the integration test to run, the tools needs to be
added to the `all-tools` image.

To add a new tool to the image:

1. Edit `all-tools/Dockerfile` to install the tool.  Consider adding a line to
   the language-specific sections (Ruby Gems, Go get, Python pip, npm...) if
   your tool can be installed with them.  Otherwise, you may need to add a new
   `RUN` command.  The [official Docker documentation][docker-man] may be
   helpful.

   You can test the image locally with `docker build`.  Note that the build can
   take around ten minutes or more, depending on your Internet connection.
   There are many packages to download and to build.  If you don't test locally
   and make a pull request, the Travis integration will build it anyway.

2. Edit `all-tools/check-tools` to add a line to output the new tool version.
   This script is a basic check to ensure the tool is correctly installed and
   found in the image.  It doubles as a useful reference to know which versions
   of the tools are in the image.

   You should add a line that outputs the tool version *on a single line*.  The
   output format should be:

   ```
   LANGUAGE-CHECKER: VERSION
   ```

   E.g.:

   ```
   python-flake8: 3.5.0 (mccabe: 0.6.1, pycodestyle: 2.3.1, pyflakes: 1.6.0) CPython 2.7.15rc1 on Linux
   ```

   `LANGUAGE-CHECKER` should be the same as in the variable `flycheck-checkers`
   from `flycheck.el`.  The `check-tools` script follows the same order as
   `flycheck-checkers`: there is one line per entry `flycheck-checkers`.
   `VERSION` can contain other identifying information, as in the example above.
   Use `head -n1`, `cut` or `sed` to ensure all relevant information stays on a
   single line.

   Some tools output their version number to `stderr`; use `2>&1` to redirect
   that to `stdout` instead (this is useful to easily check for errors when
   using `docker build` locally, as `stderr` appears in red).

[hub:all-tools]: https://hub.docker.com/r/flycheck/all-tools/
[hub:emacs-cask]: https://hub.docker.com/r/flycheck/emacs-cask/
[flycheck/flycheck]: https://www.github.com/flycheck/flycheck
[docker-man]: https://docs.docker.com/engine/reference/builder/#run
[manual:run-integ]: http://www.flycheck.org/en/latest/contributor/contributing.html#running-all-the-integration-tests
