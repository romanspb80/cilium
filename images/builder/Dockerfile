ARG COMPILERS_IMAGE=docker.io/cilium/image-compilers:818f48aa3eada1d23018f02f3c43a169c51eea1a@sha256:2f71fc0c9bdbb9a430ccb8cd86cb1fecc63acff464fc253fe3cd3748488e0e65
ARG GOLANG_IMAGE=docker.io/library/golang:1.14.4@sha256:d39a459086c75920390c9b9d42bbc7c52fef1fe7666e7633a02508c607c889d4

FROM ${GOLANG_IMAGE} as golang-dist

FROM ${COMPILERS_IMAGE}
LABEL maintainer="maintainer@cilium.io"

COPY --from=golang-dist /usr/local/go /usr/local/go
RUN mkdir -p /go
ENV GOROOT /usr/local/go
ENV GOPATH /go
ENV PATH "${GOROOT}/bin:${GOPATH}/bin:${PATH}"

COPY build-go-deps.sh /tmp/build-go-deps.sh
RUN /tmp/build-go-deps.sh

WORKDIR /go/src/github.com/cilium/cilium
