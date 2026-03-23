FROM golang:1.18-bullseye

ENV GIN_MODE=release
ENV PORT=18181
ENV GOPATH=/go
ENV CGO_ENABLED=1
ENV GOOS=linux
ENV PGROOT=/usr/local/pgsql
ENV CGO_CFLAGS=-I/usr/local/pgsql/include
ENV CGO_LDFLAGS=-L/usr/local/pgsql/lib
ENV LD_LIBRARY_PATH=/usr/local/pgsql/lib

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        bison \
        build-essential \
        ca-certificates \
        flex \
        git \
        libreadline-dev \
        libssl-dev \
        zlib1g-dev && \
    rm -rf /var/lib/apt/lists/*

COPY registry/db /app/db
WORKDIR /app/db

RUN make maintainer-clean || true && \
    ./configure \
        --prefix=${PGROOT} \
        --enable-thread-safety && \
    make -j"$(nproc)" -s && \
    make install -s

COPY doctor /go/src/ructf.org/doctor
WORKDIR /go/src/ructf.org/doctor

ARG USER=appuser
ARG GROUP=appgroup
ARG UID=1337
ARG GID=1337
RUN groupadd --gid ${GID} ${GROUP} && \
    useradd \
        --uid ${UID} \
        --gid ${GID} \
        --home-dir /go/src/ructf.org/doctor \
        --no-create-home \
        --shell /usr/sbin/nologin \
        ${USER} && \
    chown -R ${UID}:${GID} /go

USER ${UID}:${GID}

RUN go get ./... && go build -v .

EXPOSE ${PORT}

ENTRYPOINT ["./doctor-service"]
