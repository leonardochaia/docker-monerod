ARG MONERO_BRANCH=v0.15.0.1

FROM ubuntu:18.04 as build

# https://github.com/monero-project/monero#compiling-monero-from-source
RUN apt-get update && apt-get -y install build-essential cmake \
    pkg-config libboost-all-dev libssl-dev libzmq3-dev libunbound-dev \
    libsodium-dev libunwind8-dev liblzma-dev libreadline6-dev libldns-dev \
    libexpat1-dev doxygen graphviz libpgm-dev qttools5-dev-tools libhidapi-dev \
    libusb-dev libprotobuf-dev protobuf-compiler libgtest-dev git

WORKDIR /usr/src/gtest

RUN cmake . && make && mv libg* /usr/lib/

WORKDIR /monero

ARG MONERO_BRANCH
RUN git clone --recursive -b ${MONERO_BRANCH} \
    https://github.com/monero-project/monero . \
    && git submodule init && git submodule update

RUN apt-get install -y libnorm-dev libpgm-dev
RUN apt-get install -y libusb-1.0-0-dev
RUN apt-get install -y libudev-dev

RUN make -j8 release-static

FROM ubuntu:18.04

RUN apt-get update && apt-get install -y libnorm-dev libpgm-dev

RUN useradd -ms /bin/bash monero && mkdir -p /home/monero/.bitmonero \
    && chown -R monero:monero /home/monero/.bitmonero
USER monero

WORKDIR /home/monero
COPY --chown=monero:monero --from=build /monero/build/Linux/*/release/bin/* /usr/local/bin/

RUN monerod --help

ENTRYPOINT ["monerod"]
CMD ["--non-interactive", "--restricted-rpc", "--rpc-bind-ip=0.0.0.0", "--confirm-external-bind"]
