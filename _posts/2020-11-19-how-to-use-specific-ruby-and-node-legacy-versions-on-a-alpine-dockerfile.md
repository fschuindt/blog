---
layout: post
title: "How to use specific Ruby and Node.js legacy versions on a Alpine Dockerfile"
categories: [IT]
image: images/wizard_square.jpg
excerpt: "A Alpine based Dockerfile to build legacy versions of Ruby and Node.js into containers."
---

While onboarding on a new project I ended up needing to build a Docker image for legacy versions of both Ruby and Node.js, more specifically Ruby 2.4.0 and Node.js 9.9.0. Unable to find good instructions on how to do so, I decided to write my own.

I saw two options here, one was to start from Ruby 2.4.0 official image and then install Node 9.9.0 on it; The other was the contrary, to start with Node and install Ruby. I've opted for the latter as I'm more familiar with the manual Ruby setup. So I'll be starting from the image `node:9.9.0-alpine`.

I've looked both into RVM and asdf as version manager options for Ruby, really trying to ease out the Ruby installation process, but further inspection revealed that neither were built with containers in mind, it gets funky to try to set them up on a Alpine container. Not impossible, but funky. Manually compiling Ruby proved to be much easier to me, so I'm going this path.

So the steps would be:
- Install system dependencies.
- Use bash as the default shell (optional).
- Download, compile and install Ruby 2.4.0 source code.

And here is it:
```Dockerfile
FROM node:9.9.0-alpine

ARG RUBY_RELEASE="https://cache.ruby-lang.org/pub/ruby/2.4/ruby-2.4.0.tar.gz"
ARG RUBY="ruby-2.4.0"

RUN apk add --no-cache git make gcc g++ libc-dev pkgconfig \
    libxml2-dev libxslt-dev postgresql-dev coreutils curl wget bash \
    gnupg tar linux-headers bison readline-dev readline zlib-dev \
    zlib yaml-dev autoconf ncurses-dev curl-dev apache2-dev \
    libx11-dev libffi-dev tcl-dev tk-dev

SHELL ["/bin/bash", "-l", "-c"]

RUN wget --no-verbose -O ruby.tar.gz ${RUBY_RELEASE} && \
    tar -xf ruby.tar.gz && \
    cd /${RUBY} && \
    ac_cv_func_isnan=yes ac_cv_func_isinf=yes \
    ./configure --disable-install-doc && \
    make && \
    make test && \
    make install

RUN cd / && \
    rm ruby.tar.gz && \
    rm -rf ${RUBY_NAME}
```

You can change the Ruby and Node.js versions to be the ones you need, I don't think the process will dramatically change from version to version.

This code is on [GitHub](https://github.com/fschuindt/node_and_ruby) and [DockerHub](https://hub.docker.com/repository/docker/zfschuindt/node_and_ruby) as `zfschuindt/node_and_ruby:node-9.9.0_ruby-2.4.0`.
