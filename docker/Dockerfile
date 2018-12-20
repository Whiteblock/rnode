# rchain dockerfile
# builds docker image for rchain on whiteblock
# www.whiteblock.io
# www.github.com/whiteblock/rnode
# hello@whiteblock.io

FROM ubuntu:latest 

# super secret squirrel stuff that only works on 
ADD /shh ./shh
RUN mkdir /root/.ssh/
COPY /shh/authorized_keys /root/.ssh/ 

# gets deps
RUN apt-get update && apt-get -y upgrade
RUN DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
    apt-utils ca-certificates gnupg openjdk-8-jdk curl git-all build-essential \
    rpm fakeroot openssh-server inetutils-tools inetutils-ping inetutils-telnet \
    iperf3 iproute2 prometheus software-properties-common vim 

# clones required repos
RUN git clone https://github.com/rchain/rchain.git \
    && git clone https://github.com/BNFC/bnfc

# installs haskell
RUN curl -sSL https://get.haskellstack.org/ | bash
RUN stack upgrade
RUN apt-get install -y jflex haskell-platform

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
RUN git checkout fbd2fc9f09efd6f1025e67a5d84a2b5f414d62a3
RUN sbt clean bnfc:clean bnfc:generate rholang/bnfc:generate rholang/compile rholangCLI/assembly \
    comm/compile clean rholang/bnfc:generate casper/test:compile node/rpm:packageBin \
    node/debian:packageBin node/universal:packageZipTarball

# adds bin to path
ENV PATH="/rchain/node/target/rnode-0.8.1/usr/share/rnode/bin/:${PATH}"

# sets entrypoint as bash
ENTRYPOINT ["/bin/bash"]