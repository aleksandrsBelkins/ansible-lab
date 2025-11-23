FROM ubuntu:22.04
RUN apt-get update && \
    apt-get install -y software-properties-common && \
    add-apt-repository --yes --update ppa:ansible/ansible && \
    apt-get update && \
    apt-get install -y ansible

WORKDIR /root/ansible


