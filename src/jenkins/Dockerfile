ARG ALPINE_VERSION="3.15"
ARG TRIVY_VERSION="0.21.1"

FROM alpine:$ALPINE_VERSION AS packages

ARG KUBECTL_VERSION="1.22.4-r0"
ARG DOCKER_CLI_VERSION="20.10.11-r0"
RUN apk add --no-cache --repository http://dl-cdn.alpinelinux.org/alpine/edge/testing \
    kubectl=$KUBECTL_VERSION \
    docker-cli=$DOCKER_CLI_VERSION

FROM alpine:$ALPINE_VERSION AS downloads

ARG BUILDKIT_VERSION="v0.9.3"
WORKDIR /downloads/buildkit
RUN wget https://github.com/moby/buildkit/releases/download/$BUILDKIT_VERSION/buildkit-$BUILDKIT_VERSION.linux-amd64.tar.gz
RUN tar xvf buildkit-$BUILDKIT_VERSION.linux-amd64.tar.gz

ARG HELM_VERSION="v3.7.1"
WORKDIR /downloads/helm
RUN wget https://get.helm.sh/helm-$HELM_VERSION-linux-amd64.tar.gz
RUN tar xvf helm-$HELM_VERSION-linux-amd64.tar.gz

ARG COMPOSE_VERSION="v2.2.0"
WORKDIR /downloads/compose
RUN wget https://github.com/docker/compose/releases/download/$COMPOSE_VERSION/docker-compose-linux-x86_64

FROM aquasec/trivy:$TRIVY_VERSION AS trivy

FROM alpine:$ALPINE_VERSION AS jenkins

ARG JENKINS_VERSION="2.319.1"
WORKDIR /downloads/jenkins
RUN wget http://mirrors.jenkins.io/war-stable/$JENKINS_VERSION/jenkins.war

# Jenkins
FROM alpine:$ALPINE_VERSION

# jenkins deps
RUN apk add --no-cache \
    bash \
    coreutils \
    jq \
    git \
    openjdk11 \
    openssh-client \
    ttf-dejavu \
    unzip 
    
ENV JENKINS_HOME="/data"
VOLUME ${JENKINS_HOME}

EXPOSE 8080
ENTRYPOINT /start.sh

COPY --from=packages /usr/bin/kubectl /usr/bin/kubectl
COPY --from=packages /usr/bin/docker /usr/bin/docker

COPY --from=downloads /downloads/buildkit/bin/buildctl /usr/bin/buildctl
COPY --from=downloads /downloads/helm/linux-amd64/helm /usr/bin/helm
COPY --from=downloads /downloads/compose/docker-compose-linux-x86_64 /root/.docker/cli-plugins/docker-compose

COPY --from=trivy /usr/local/bin/trivy /usr/local/bin/trivy
COPY --from=jenkins /downloads/jenkins/jenkins.war /jenkins/jenkins.war

COPY ./jenkins.install.UpgradeWizard.state ${JENKINS_HOME}/
COPY ./scripts/ ${JENKINS_HOME}/init.groovy.d/
COPY start.sh /

RUN chmod +x /start.sh && \
    chmod +x /root/.docker/cli-plugins/docker-compose