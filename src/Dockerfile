ARG MONERO_BRANCH=v0.17.3.2
ARG UBUNTU_VERSION=20.04

# Use Ubuntu for the build image base
FROM ubuntu:${UBUNTU_VERSION} as build

# Dependency list from https://github.com/monero-project/monero#compiling-monero-from-source
# Added DEBIAN_FRONTEND=noninteractive to workaround tzdata prompt on installation
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get -y install --no-install-recommends build-essential cmake \
    pkg-config libboost-all-dev libssl-dev libzmq3-dev libunbound-dev ca-certificates \
    libsodium-dev libunwind8-dev liblzma-dev libreadline6-dev libldns-dev \
    libexpat1-dev doxygen graphviz libpgm-dev qttools5-dev-tools libhidapi-dev \
    libusb-dev libprotobuf-dev protobuf-compiler libgtest-dev git \
    libnorm-dev libpgm-dev libusb-1.0-0-dev libudev-dev libgssapi-krb5-2 \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Switch to directory for gtest and make/install libs
WORKDIR /usr/src/gtest
RUN cmake . && make && cp ./lib/libgtest*.a /usr/lib

# Switch to Monero source directory
WORKDIR /monero

# Git pull Monero source at specified tag/branch
ARG MONERO_BRANCH
RUN git clone --recursive --branch ${MONERO_BRANCH} \
    https://github.com/monero-project/monero . \
    && git submodule init && git submodule update

# Make static Monero binaries
RUN make -j8 release-static

# Clean Ubuntu layer for the runtime image
FROM ubuntu:${UBUNTU_VERSION}

# Install remaining dependencies
RUN apt-get update && apt-get install --no-install-recommends -y libnorm-dev libpgm-dev libgssapi-krb5-2 \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Add user and setup directories for monerod
RUN useradd -ms /bin/bash monero && mkdir -p /home/monero/.bitmonero \
    && chown -R monero:monero /home/monero/.bitmonero
USER monero

# Switch to home directory and install newly built monerod binary
WORKDIR /home/monero
COPY --chown=monero:monero --from=build /monero/build/Linux/*/release/bin/monerod /usr/local/bin/monerod

# Expose p2p and restricted RPC ports
EXPOSE 18080
EXPOSE 18089

# Start monerod with required --non-interactive flag and sane defaults that are overridden by user input (if applicable)
ENTRYPOINT ["monerod"]
CMD ["--non-interactive", "--restricted-rpc", "--rpc-bind-ip=0.0.0.0", "--confirm-external-bind"]
