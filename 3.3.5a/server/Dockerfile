FROM ubuntu:latest AS builder
RUN DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y mysql-client libmysqlclient21 git gosu inetutils-ping libncurses-dev git libboost-all-dev git
WORKDIR /repos
RUN git clone --depth 1 -b 3.3.5 https://github.com/TrinityCore/TrinityCore.git
WORKDIR /server/bin
CMD echo "Use docker-compose to run any command"
