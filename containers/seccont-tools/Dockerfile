FROM golang:1.12
RUN apt update
RUN apt install -y libgpgme-dev libassuan-dev libbtrfs-dev libdevmapper-dev libostree-dev ent curl wget
RUN mkdir -p $GOPATH/src/github.com/containers/ \
RUN cd $GOPATH/src/github.com/containers/ && \
    git clone https://github.com/harche/skopeo.git && \
    cd skopeo && git checkout pr_branch && \
    make binary-local && \
    cp skopeo /skopeo

FROM ubuntu:19.04
RUN apt update
RUN apt install -y libgpgme11-dev libassuan-dev libbtrfs-dev libdevmapper-dev libostree-dev vim ent
COPY policy.json /etc/containers/policy.json
COPY --from=0 /skopeo .
