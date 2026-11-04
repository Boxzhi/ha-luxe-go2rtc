# syntax=docker/dockerfile:1.7

ARG GO_VERSION=1.27
ARG PYTHON_VERSION=3.13
ARG GO2RTC_REF=443f0f93222e8ba6013e42b0d0c75def516ae8b3

# Build the exact, CI-verified Luxe go2rtc source revision.
FROM --platform=$BUILDPLATFORM golang:${GO_VERSION}-alpine AS build
ARG TARGETOS
ARG TARGETARCH
ARG GO2RTC_REF

ENV GOOS=${TARGETOS}
ENV GOARCH=${TARGETARCH}

RUN apk add --no-cache git
WORKDIR /src

RUN git init . \
    && git remote add origin https://github.com/Boxzhi/go2rtc.git \
    && git fetch --depth=1 origin "${GO2RTC_REF}" \
    && git checkout --detach FETCH_HEAD

RUN --mount=type=cache,target=/go/pkg/mod go mod download
RUN --mount=type=cache,target=/root/.cache/go-build \
    CGO_ENABLED=0 go build -ldflags "-s -w" -trimpath -o /out/go2rtc .

# Match the upstream go2rtc runtime image's FFmpeg and Intel VAAPI packages.
FROM python:${PYTHON_VERSION}-alpine AS runtime
ARG TARGETARCH

RUN apk add --no-cache \
      tini ffmpeg ffplay bash curl jq alsa-plugins-pulse font-droid \
    && if [ "${TARGETARCH}" = "amd64" ]; then \
         apk add --no-cache libva-intel-driver intel-media-driver; \
       fi

COPY --from=build /out/go2rtc /usr/local/bin/go2rtc

EXPOSE 1984 8554 8555 8555/udp
ENTRYPOINT ["/sbin/tini", "--"]
VOLUME /config
WORKDIR /config
CMD ["go2rtc", "-config", "/config/go2rtc.yaml"]
