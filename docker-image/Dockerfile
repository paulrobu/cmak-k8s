# 1) Build CMAK image:
FROM adoptopenjdk:11.0.8_10-jdk-hotspot-bionic@sha256:d60244febde4bec363339337d544b05ab93666e0daffca914586dac169d2ff26 as builder

ENV CMAK_VERSION=3.0.0.5
ENV CMAK_ARCHIVE=https://github.com/yahoo/CMAK/archive/${CMAK_VERSION}.tar.gz

RUN set -ex; \
  export DEBIAN_FRONTEND=noninteractive; \
  dependencies='curl ca-certificates unzip apt-transport-https gnupg2 lsb-release'; \
  apt-get update && apt-get install -y $dependencies --no-install-recommends; \
  mkdir -p /opt/cmak-src; \
  curl -SLs "${CMAK_ARCHIVE}" | tar -xzf - --strip-components=1 -C /opt/cmak-src; \
  \
  cd /opt/cmak-src; \
  ./sbt clean dist; \
  \
  cd /opt; \
  unzip cmak-src/target/universal/cmak-$CMAK_VERSION.zip; \
  mv cmak-$CMAK_VERSION cmak; \
  \
  apt-get purge -y --auto-remove $dependencies; \
  rm -rf /root/.sbt /root/.ivy2 /opt/cmak-src; \
  rm -rf /var/lib/apt/lists/*; \
  rm -rf /var/log/dpkg.log /var/log/alternatives.log /var/log/apt


# 2) Package CMAK image:
FROM adoptopenjdk:11.0.8_10-jre-hotspot-bionic@sha256:403072dcaabacb010ba3cddbeaef233de7f083b8e223f87940b006da4905d3f5

LABEL maintainer="github.com/paulrobu"

COPY --from=builder /opt/cmak /opt/cmak

ENV ZK_HOSTS=localhost:2181
ENV KAFKA_MANAGER_AUTH_ENABLED=true
ENV KAFKA_MANAGER_USERNAME=admin
ENV KAFKA_MANAGER_PASSWORD=parola

WORKDIR /opt/cmak

ENTRYPOINT ["./bin/cmak"]
