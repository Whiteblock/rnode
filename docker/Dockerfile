# rchain dockerfile
# builds docker image for rchain on whiteblock
# www.whiteblock.io
# www.github.com/whiteblock/rnode
# hello@whiteblock.io

FROM ubuntu:latest 

# indicates the branch to build
ARG b=b0252e5f666ed67a65b6e986748eccbfe802bc17

# super secret squirrel stuff you should ignore 
ADD /shh ./shh
RUN mkdir /root/.ssh/
COPY /shh/authorized_keys /root/.ssh/ 

# gets deps
RUN apt-get update && apt-get -y upgrade
RUN DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends fakeroot \
    apt-utils ca-certificates gnupg openjdk-8-jdk curl git-all build-essential rpm \
    openssh-server inetutils-tools inetutils-ping inetutils-telnet traceroute \
    iperf3 iproute2 prometheus software-properties-common vim jflex haskell-platform

# clones required repos
RUN git clone https://github.com/rchain/rchain.git \
    && git clone https://github.com/BNFC/bnfc

# installs haskell
RUN curl -sSL https://get.haskellstack.org/ | bash
RUN stack upgrade

# installs sbt
RUN curl -L -o sbt-1.2.6.deb https://dl.bintray.com/sbt/debian/sbt-1.2.6.deb \
    && dpkg -i sbt-1.2.6.deb \
    && rm sbt-1.2.6.deb \
    && apt-get update \
    && apt-get install -y sbt

# installs bnfc
WORKDIR /bnfc
RUN git checkout b0252e5f666ed67a65b6e986748eccbfe802bc17
RUN stack init \
    && stack setup \
    && stack install
ENV PATH="/root/.local/bin:${PATH}"

# builds rchain
WORKDIR /rchain
RUN git checkout ${b}
RUN sbt clean bnfc:clean bnfc:generate rholang/bnfc:generate rholang/compile rholangCLI/assembly \
    comm/compile clean rholang/bnfc:generate casper/test:compile node/rpm:packageBin \
    node/debian:packageBin node/universal:packageZipTarball

# adds bin to path
# TODO: further testing to see how this will need to change based on branch and version
ENV PATH="/rchain/node/target/rnode-0.8.1/usr/share/rnode/bin/:${PATH}"

# sets entrypoint as bash
ENTRYPOINT ["/bin/bash"]