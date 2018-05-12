This repository contains the Dockerfiles used for running Flycheck's unit and
integration tests on Travis CI.

- `all-tools` creates a container (based on Ubuntu 18.04) installing all the
  tools that Flycheck supports.  The full image is around 4.5GB uncompressed.
- `emacs-cask` creates a container (still on Ubuntu 18.04) with Emacs, Cask, and
  Make installed.  This is used to run Flycheck's unit tests.  It is
  configurable: we can build with different Emacs versions.

These two images are scheduled to build monthly (or whenever there is a commit
to master), and uploaded to
[flycheck/all-tools](https://hub.docker.com/r/flycheck/all-tools/) and
[flycheck/emacs-cask](https://hub.docker.com/r/flycheck/emacs-cask/)
respectively.

There is a third image in
[flycheck/flycheck](https://www.github.com/flycheck/flycheck),
`all-tools-and-emacs-cask`. This one is never pushed anywhere; it is built and
used directly on Travis in order to run our integration tests.
