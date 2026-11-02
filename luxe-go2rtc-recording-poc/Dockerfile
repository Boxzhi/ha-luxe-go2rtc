# syntax=docker/dockerfile:1.7

ARG GO_VERSION=1.27
ARG GO2RTC_REF=f9ed2c41c767cbd0adbb46da20f8fd339985c2fe

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

# The POC is native Go. No FFmpeg or GPAC runtime is installed or invoked.
FROM alpine:3.22 AS runtime

RUN apk add --no-cache tini ca-certificates
COPY --from=build /out/go2rtc /usr/local/bin/go2rtc

EXPOSE 1985
ENTRYPOINT ["/sbin/tini", "--"]
VOLUME /config
WORKDIR /config

# Keep this POC isolated from the stable add-on:
# - API uses 1985 instead of 1984.
# - RTSP server and WebRTC listener are disabled.
# - The three test streams are RTSP aliases to the stable add-on on loopback.
CMD ["go2rtc", "-config", "api.listen=\":1985\"", "-config", "rtsp.listen=\"\"", "-config", "webrtc.listen=\"\"", "-config", "streams.jinghuafu_keting_ptz_3k=rtsp://127.0.0.1:8554/jinghuafu_keting_ptz_3k", "-config", "streams.jinghuafu_keting_fixed_3k=rtsp://127.0.0.1:8554/jinghuafu_keting_fixed_3k", "-config", "streams.jinghuafu_canting_2k=rtsp://127.0.0.1:8554/jinghuafu_canting_2k"]
