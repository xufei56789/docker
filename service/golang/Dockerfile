FROM golang:latest
WORKDIR /go/src/www

ENV GO111MODULE on
ENV GOPROXY https://mirrors.aliyun.com/goproxy/

RUN dpkg-reconfigure -f noninteractive tzdata \
    && apt-get update && apt-get install -y \
          git \

&& cd /srv && git clone https://github.com/brendangregg/FlameGraph.git \
&& cd FlameGraph/  && cp flamegraph.pl /usr/local/bin \
&& go get -v github.com/uber/go-torch











