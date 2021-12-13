ARG ALPINE_VERSION="3.15"

FROM alpine:$ALPINE_VERSION AS download-base
WORKDIR /downloads
RUN echo "$(apk --print-arch)" > /arch.txt 

FROM download-base AS installer
ARG LOGSTASH_VERSION="7.10.2"

# find the downloads for OSS versions here - https://www.elastic.co/downloads/past-releases#logstash-oss
# logstash is still OSS, see - https://www.elastic.co/pricing/faq/licensing

RUN wget -O logstash.tar.gz https://artifacts.elastic.co/downloads/logstash/logstash-oss-${LOGSTASH_VERSION}-linux-$(cat /arch.txt).tar.gz

RUN mkdir /logstash && \
    tar -xzf logstash.tar.gz --strip-components=1 -C /logstash && \
    rm -rf /logstash/jdk

# Logstash requires a JVM - this image provides a minimal JRE installation
# see the product-JVM versions in https://www.elastic.co/guide/en/logstash/current/getting-started-with-logstash.html
FROM openjdk:11.0.11-jre-slim

WORKDIR /usr/share/logstash
COPY --from=installer /logstash .

EXPOSE 9600 5044
ENV LS_JAVA_OPTS="-Dls.cgroup.cpuacct.path.override=/ -Dls.cgroup.cpu.path.override=/" 

COPY ./config/ ./config/
COPY ./pipelines/ /pipelines/

CMD ["/usr/share/logstash/bin/logstash"]