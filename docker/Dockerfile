FROM python:3.5-slim as base

#######
FROM base as builder

COPY . /arthur

WORKDIR /arthur

# install dependencies
RUN apt-get update && \
    apt-get -y install --no-install-recommends \
        apt-utils git gcc python3-dev &&\
    apt-get clean && \
    find /var/lib/apt/lists -type f -delete

RUN pip3 install wheel && pip3 wheel --wheel-dir=/arthur/wheels -r requirements.txt

#######
FROM base

EXPOSE 8080

ENV \
  ARTHUR_HOST="0.0.0.0"\
  ARTHUR_PORT="8080" \
  ARTHUR_DATABASE="redis://localhost/8" \
  ARTHUR_LOGS_DIR="/arthur/logs"

ARG ARTHUR_VERSION

LABEL \
  maintainer="Olblak <me@olblak.com>" \
  description="Arthur is a distributed job queue platform that schedules and executes Perceval" \
  project="https://github.com/chaoss/grimoirelab-kingarthur" \
  version="$ARTHUR_VERSION"

RUN \
  groupadd -g 1000 arthur && \
  useradd -m -d /arthur -g 1000 -u 1000 -s /bin/sh arthur &&\
  mkdir /archives && chown arthur:arthur /archives 

WORKDIR /arthur

USER arthur

COPY --from=builder --chown=arthur:arthur /arthur /arthur

RUN \
  mkdir /arthur/logs &&\
  pip3 install --user --no-index --find-links=/arthur/wheels .

# /tmp is used by arthur to store temp files like git repository
VOLUME ["/tmp", "/arthur/logs", "/archives"]

ENTRYPOINT [\
  "/bin/sh", "-c", \
  "python3 bin/arthurd --no-daemon --host ${ARTHUR_HOST} --port ${ARTHUR_PORT} --database ${ARTHUR_DATABASE} --log-path ${ARTHUR_LOGS_DIR} --sync"\
  ]
