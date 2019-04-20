# This image includes (nearly) all the tools supported by Flycheck.  It is used
# to run Flycheck's integration tests on CI.

#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# Builder image.  We build ruby and GO tools, then we don't need build-essential
# for the later image
FROM ubuntu:18.04 AS builder

RUN apt-get update >/dev/null && \
    apt-get install --no-install-recommends -y \
            build-essential \
            curl \
            ruby-dev \
            # Needed by `go get`
            git \
            zlib1g-dev \
            >/dev/null \
    && rm -rf /var/lib/apt/lists/*

# Install Ruby tools
RUN gem install --no-document --quiet \
        foodcritic \
        haml \
        mdl \
        puppet-lint \
        reek \
        rubocop \
        ruby-lint \
        sass \
        scss_lint \
        scss_lint_reporter_checkstyle \
        slim \
        slim_lint \
        sqlint

# Install GO and GO tools
ENV GOLANG_VERSION 1.12
ENV GOPATH /go
ENV PATH   $GOPATH/bin:/usr/local/go/bin:$PATH
RUN curl -sSLo go.tgz "https://golang.org/dl/go${GOLANG_VERSION}.linux-amd64.tar.gz" \
    && tar -xf go.tgz -C /usr/local \
    && rm go.tgz \
    && go get \
          golang.org/x/lint/golint \
          github.com/kisielk/errcheck \
          github.com/mdempsky/unconvert \
          honnef.co/go/tools/cmd/staticcheck \
          honnef.co/go/tools/cmd/megacheck \
          >/dev/null

#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# Runtime image
FROM ubuntu:18.04

# Install tools that are in APT
ENV DEBIAN_FRONTEND noninteractive
RUN apt-get update >/dev/null && \
    apt-get install --no-install-recommends -y \
            asciidoc \
            asciidoctor \
            cfengine3 \
            chicken-bin \
            chktex \
            clang \
            coq \
            cppcheck \
            # For fetching tarballs
            curl \
            erlang \
            g++ \
            gcc \
            gfortran \
            ghc \
            gnat \
            groovy \
            hlint \
            jruby \
            lacheck \
            libperl-critic-perl \
            luarocks \
            perl \
            php-cli \
            php-codesniffer \
            protobuf-compiler \
            puppet \
            python3-distutils \
            python3-pip \
            racket \
            rpmlint \
            ruby \
            scala \
            shellcheck \
            texinfo \
            tidy \
            unzip \
            verilator \
            xmlstarlet \
            zsh \
            # For uncompressing tarballs
            xz-utils \
            >/dev/null \
    && rm -rf /var/lib/apt/lists/*

# Get Ruby tools (and GO install) from builder
COPY --from=builder /usr/local/bin /usr/local/bin
COPY --from=builder /var/lib/gems /var/lib/gems

# Get GO tools from builder
ENV GOPATH /go
ENV PATH   $GOPATH/bin:/usr/local/go/bin:$PATH
COPY --from=builder /usr/local/go /usr/local/go
COPY --from=builder /go /go

# Install Rust
ENV RUSTUP_HOME /usr/local/rustup
ENV CARGO_HOME  /usr/local/cargo
ENV PATH        $CARGO_HOME/bin:$PATH
RUN curl -sSLf "https://sh.rustup.rs" | sh -s -- -y \
# Cargo needs to update its index otherwise tests will timeout
# There's a `-Z offline` flag, but nightly only.
# See https://github.com/rust-lang/cargo/issues/4686
    && cargo search --quiet

# Install nodejs and NPM tools
ENV NODE_VERSION 10.15.3
RUN curl -sSLo node.txz "https://nodejs.org/dist/v${NODE_VERSION}/node-v${NODE_VERSION}-linux-x64.tar.xz" \
    && tar -C /usr/local -xf node.txz --strip-components=1 --no-same-owner \
    && rm node.txz \
    && ln -s /usr/local/bin/node /usr/local/bin/nodejs \
    && npm install --quiet --global \
           coffeelint \
           coffeescript \
           csslint \
           eslint \
           handlebars \
           js-yaml \
           jscs \
           jshint \
           jsonlint \
           less \
           markdownlint-cli \
           pug-cli \
           sass-lint \
           semistandard \
           standard \
           stylelint \
           textlint \
           tslint \
           typescript

# Install Python tools
# We need setuptools to install the tools, and we can't do it in the same
# command
RUN pip3 install --quiet --upgrade pip setuptools
RUN pip3 install --quiet \
         docutils \
         flake8 \
         mypy \
         proselint \
         pylint \
         schema-salad \
         sphinx \
    && rm -rf /root/.cache/pip

# Setup C locale, needed by a proselint dependency at least when using python3
# See http://click.pocoo.org/5/python3/
ENV LC_ALL C.UTF-8
ENV LANG   C.UTF-8

# Install Lua tools
RUN luarocks install luacheck

# Install Processing
ENV PROCESSING_VERSION 3.3.7
RUN curl -sSLo processing.tgz "http://download.processing.org/processing-${PROCESSING_VERSION}-linux64.tgz" \
    && mkdir /opt/processing \
    && tar -xf processing.tgz -C /opt/processing --strip-components=1 \
    && ln -s /opt/processing/processing-java /usr/local/bin/processing-java \
    && rm processing.tgz

# Get rebar3
RUN curl -sSLo /usr/local/bin/rebar3 https://s3.amazonaws.com/rebar3/rebar3 \
    && chmod +x /usr/local/bin/rebar3

# Get hadolint
COPY --from=hadolint/hadolint:latest /bin/hadolint /usr/local/bin/

# Output version of all installed tools
COPY check-tools /usr/local/bin/
RUN check-tools
