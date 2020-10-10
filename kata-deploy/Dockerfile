FROM ubuntu:20.04

ARG KATA_TAG=1.11.3
ENV TAG $KATA_TAG

ARG UPSTREAM_KERNEL=5.4.60
ENV UPSTREAM $UPSTREAM_KERNEL

RUN apt-get update && \
    apt-get install -y gcc ca-certificates git wget curl bc vim less file xz-utils unzip make flex bison libssl-dev libelf-dev && \
    rm -f /bin/sh && ln -s /bin/bash /bin/sh

ENV GOPATH=/go PATH=/go/bin:/usr/local/go/bin:${PATH} SHELL=/bin/bash

RUN wget -O - https://storage.googleapis.com/golang/go1.14.7.linux-amd64.tar.gz | tar -xzf - -C /usr/local

RUN mkdir -p /go/src/github.com/kata-containers && cd /go/src/github.com/kata-containers \
	&& git clone https://github.com/kata-containers/packaging.git && cd /go/src/github.com/kata-containers/packaging \
	&& git checkout $TAG

ADD k3s.conf /go/src/github.com/kata-containers/packaging/kernel/configs/fragments/common/k3s.conf

RUN cd /go/src/github.com/kata-containers/packaging/kernel \
	&& ./build-kernel.sh -e -f setup \
	&& ./build-kernel.sh -e -f build \
	&& ./build-kernel.sh -e -f install \
	&& ./build-kernel.sh -v ${UPSTREAM} -f setup \
	&& ./build-kernel.sh -v ${UPSTREAM} -f build \
	&& ./build-kernel.sh -v ${UPSTREAM} -f install

FROM centos/systemd
ARG KUBE_ARCH=amd64
ARG KATA_ARTIFACTS=./kata-static.tar.xz
ARG DESTINATION=/opt/kata-artifacts

COPY ${KATA_ARTIFACTS} .

RUN \
yum install -y epel-release && \
yum install -y bzip2 jq && \
mkdir -p ${DESTINATION} && \
tar xvf ${KATA_ARTIFACTS} -C ${DESTINATION}/ && \
chown -R root:root ${DESTINATION}/

COPY --from=0 /usr/share/kata-containers/ ${DESTINATION}/opt/kata/share/kata-containers/

RUN \
curl -Lso /bin/kubectl https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/${KUBE_ARCH}/kubectl && \
chmod +x /bin/kubectl

COPY scripts ${DESTINATION}/scripts
RUN \
ln -s ${DESTINATION}/scripts/kata-deploy-docker.sh /usr/bin/kata-deploy-docker && \
ln -s ${DESTINATION}/scripts/kata-deploy.sh /usr/bin/kata-deploy
