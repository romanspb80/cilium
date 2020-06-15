# syntax=docker/dockerfile:1.1-experimental
ARG CILIUM_BUILDER_IMAGE=docker.io/cilium/cilium-builder-dev:cbcb372494cf06dc98d5d52fb48054b771ef2f36
ARG CILIUM_RUNTIME_IMAGE=docker.io/cilium/cilium-runtime-dev:abd63602d52af83b07265f9489ddbacc00241919

FROM --platform=linux/amd64 ${CILIUM_BUILDER_IMAGE} as builder

ARG NOSTRIP
ARG LOCKDEBUG

RUN --mount=type=bind,readwrite,target=/go/src/github.com/cilium/cilium --mount=target=/root/.cache,type=cache --mount=target=/go/pkg/mod,type=cache \
  make clean-container build-container install-container \
    NOSTRIP=$NOSTRIP LOCKDEBUG=$LOCKDEBUG PKG_BUILD=1 SKIP_DOCS=true DESTDIR=/out/linux/amd64

RUN --mount=type=bind,readwrite,target=/go/src/github.com/cilium/cilium --mount=target=/root/.cache,type=cache --mount=target=/go/pkg/mod,type=cache \
  env GOARCH=arm64 CC=aarch64-linux-gnu-gcc \
    make clean-container build-container install-container \
      NOSTRIP=$NOSTRIP LOCKDEBUG=$LOCKDEBUG PKG_BUILD=1 SKIP_DOCS=true DESTDIR=/out/linux/arm64 \
      HOST_CC=aarch64-linux-gnu-gcc HOST_STRIP=aarch64-linux-gnu-strip

RUN --mount=type=bind,readwrite,target=/go/src/github.com/cilium/cilium \
  cd /go/src/github.com/cilium/cilium/plugins/cilium-cni \
    && cp cni-install.sh cni-uninstall.sh /out/linux/amd64 \
    && cp cni-install.sh cni-uninstall.sh /out/linux/arm64

RUN --mount=type=bind,readwrite,target=/go/src/github.com/cilium/cilium \
  cd /go/src/github.com/cilium/cilium/contrib/packaging/docker \
    && cp init-container.sh /out/linux/amd64 \
    && cp init-container.sh /out/linux/arm64

FROM ${CILIUM_RUNTIME_IMAGE}
ARG TARGETPLATFORM
LABEL maintainer="maintainer@cilium.io"

COPY --from=builder /out/${TARGETPLATFORM} /

WORKDIR /home/cilium

RUN groupadd -f cilium \
    && echo ". /etc/profile.d/bash_completion.sh" >> /etc/bash.bashrc

CMD ["/usr/bin/cilium"]
