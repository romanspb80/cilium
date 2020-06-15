# syntax=docker/dockerfile:1.1-experimental

# Copyright 2020 Authors of Cilium
# SPDX-License-Identifier: Apache-2.0

ARG COMPILERS_IMAGE=docker.io/cilium/image-compilers:818f48aa3eada1d23018f02f3c43a169c51eea1a@sha256:2f71fc0c9bdbb9a430ccb8cd86cb1fecc63acff464fc253fe3cd3748488e0e65
ARG GOLANG_IMAGE=docker.io/library/golang:1.14.4@sha256:d39a459086c75920390c9b9d42bbc7c52fef1fe7666e7633a02508c607c889d4
ARG UBUNTU_IMAGE=docker.io/library/ubuntu:20.04@sha256:8bce67040cd0ae39e0beb55bcb976a824d9966d2ac8d2e4bf6119b45505cee64

ARG CILIUM_LLVM_IMAGE=docker.io/cilium/cilium-llvm:178583d8925906270379830fb44641c38f7cc062
ARG CILIUM_BPFTOOL_IMAGE=docker.io/cilium/cilium-bpftool:f0bbd0cb389ce92b33ff29f0489c17c8e33f9da7
ARG CILIUM_IPROUTE2_IMAGE=docker.io/cilium/cilium-iproute2:044e7a6a43d5a42a8ce696535b3dbf773f82dbec

FROM ${CILIUM_LLVM_IMAGE} as llvm-dist
FROM ${CILIUM_BPFTOOL_IMAGE} as bpftool-dist
FROM ${CILIUM_IPROUTE2_IMAGE} as iproute2-dist

FROM --platform=linux/amd64 ${COMPILERS_IMAGE} as builder

COPY download-cni.sh /tmp/download-cni.sh
RUN /tmp/download-cni.sh

FROM --platform=linux/amd64 ${GOLANG_IMAGE} as go-builder

COPY build-gops.sh /tmp/build-gops.sh
RUN --mount=target=/root/.cache,type=cache --mount=target=/go/pkg/mod,type=cache /tmp/build-gops.sh

FROM ${UBUNTU_IMAGE}
LABEL maintainer="maintainer@cilium.io"

COPY install-runtime-deps.sh /tmp/install-runtime-deps.sh 
RUN /tmp/install-runtime-deps.sh

COPY iptables-wrapper /usr/sbin/iptables-wrapper
COPY configure-iptables-wrapper.sh /tmp/configure-iptables-wrapper.sh
RUN /tmp/configure-iptables-wrapper.sh

COPY --from=llvm-dist /bin /bin
COPY --from=bpftool-dist /bin /bin
COPY --from=iproute2-dist /bin /bin

ARG TARGETPLATFORM
COPY --from=builder /out/${TARGETPLATFORM}/bin /bin
COPY --from=go-builder /out/${TARGETPLATFORM}/bin /bin
