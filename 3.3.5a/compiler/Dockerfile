FROM ubuntu:latest AS builder
# RUN dpkg-reconfigure -f noninteractive tzdata
RUN DEBIAN_FRONTEND=noninteractive
#RUN apt-get update && apt-get install -y mysql-client libmysqlclient21 git gosu iputils-ping
RUN apt-get update && apt-get install -y apt-utils build-essential clang gcc g++ automake git-core autoconf make patch libmysql++-dev libtool libssl-dev grep binutils zlib1g libc6 libbz2-dev cmake libncurses-dev libboost-all-dev git gosu mysql-client p7zip libmysqlclient21 libreadline-dev
RUN update-alternatives --install /usr/bin/cc cc /usr/bin/clang 100
RUN update-alternatives --install /usr/bin/c++ c++ /usr/bin/clang 100

COPY wow wowclient
WORKDIR /repos

RUN git clone -b 3.3.5 https://github.com/TrinityCore/TrinityCore.git
# RUN git clone -b master https://github.com/TrinityCore/TrinityCore.git

WORKDIR /repos/TrinityCore/build
RUN cmake ../ -DCMAKE_INSTALL_PREFIX=/server
RUN make && make install

RUN mv /server/etc/authserver.conf.dist /server/etc/authserver.conf && mv /server/etc/worldserver.conf.dist /server/etc/worldserver.conf

# Now we need to extract + create maps
# https://trinitycore.info/en/install/Server-Setup/Linux-Server-Setup

# dbc	3.3.5 & master	Mandatory
# maps	3.3.5 & master	Mandatory
# vmaps	3.3.5 & master	Mandatory
# mmaps	3.3.5 & master	HIGHLY Recommended
# cameras	3.3.5 & master	Recommended
# gt	master	Mandatory

# https://wowdl.net/client/World-of-Warcraft-3.3.5a.12340
WORKDIR /wowclient

RUN mkdir -p /server/data && \
    mkdir -p /wowclient/vmaps && \
    mkdir -p /wowclient/mmaps && \
    chmod +x /server/bin/mapextractor && \
    chmod +x /server/bin/vmap4extractor && \
    chmod +x /server/bin/vmap4assembler && \
    chmod +x /server/bin/mmaps_generator

# Cameras, DBC and Maps files
RUN /server/bin/mapextractor && \
    mv Cameras cameras
# Next line is for 3.3.5 only
RUN cp -r cameras dbc maps /server/data
# Next line is master only
# RUN mv cameras dbc maps gt /server/data
# change datadir from "." to "/server/data"
RUN sed -i 's/DataDir = "."/DataDir = "\/server\/data"/g' /server/etc/worldserver.conf

RUN sed -i 's/127.0.0.1;3306;/trinity335db;3306;/g' /server/etc/worldserver.conf
# Disable console (so it can run as Daemon and not an interactive console, which wont work as Docker Container)
RUN sed -i 's/Console.Enable = 1/Console.Enable = 0/g' /server/etc/worldserver.conf
# And we make sure Remote Access is enabled
RUN sed -i 's/Ra.Enable = 0/Ra.Enable = 1/g' /server/etc/worldserver.conf
RUN sed -i 's/SOAP.IP = "127.0.0.1"/SOAP.IP = "0.0.0.0"/g' /server/etc/worldserver.conf
RUN sed -i 's/127.0.0.1;3306;/trinity335db;3306;/g' /server/etc/authserver.conf

# # Visual Maps (aka vmaps)
RUN /server/bin/vmap4extractor
RUN /server/bin/vmap4assembler Buildings vmaps

# # Movement Maps (aka mmaps - optional RECOMMENDED)
RUN /server/bin/mmaps_generator

RUN cp -r mmaps /server/data
RUN cp -r vmaps /server/data

# All done. move compiled to /server (mounted volume)
CMD cp -r /server/* /server-compiled/ && exit 0