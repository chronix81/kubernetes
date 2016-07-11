FROM alpine:3.4
MAINTAINER Michal Bandura <mjbandura@gmail.com>
# Based on TAGOMORI Satoshi's Dockerfile - https://hub.docker.com/r/fluent/fluentd/
# This image contains additional fluentd plugins, one of them requires ruby-dev during installation
# so basing on fluent/fluentd would require to install already deleted package ruby-dev
# still need to figure out the best way to pass custom fluentd configuration (using volumes is not very convinient in multi-nodes kubenetes installations)
LABEL Description="Fluentd for Kubernetes" Vendor="none" Version="1.1"

# Do not split this into multiple RUN!
# Docker creates a layer for every RUN-Statement
# therefore an 'apk delete build*' has no effect
RUN apk --no-cache --update add \
                            build-base \
                            ca-certificates \
                            ruby \
                            ruby-irb \
                            ruby-dev && \
    echo 'gem: --no-document' >> /etc/gemrc && \
    gem install oj && \
    gem install fluentd -v 0.12.26 && \
    gem install fluent-plugin-elasticsearch -v 1.5.0 && \
    gem install fluent-plugin-kubernetes_metadata_filter -v 0.23.0 && \
    gem install fluent-plugin-rewrite-tag-filter -v 1.5.5 && \ 
    apk del build-base ruby-dev && \
    rm -rf /tmp/* /var/tmp/* /var/cache/apk/*

RUN adduser -D -g '' -u 1000 -h /home/fluent fluent
RUN chown -R fluent:fluent /home/fluent

# for log storage (maybe shared with host)
RUN mkdir -p /fluentd/log
# configuration/plugins path (default: copied from .)
RUN mkdir -p /fluentd/etc /fluentd/plugins

RUN chown -R fluent:fluent /fluentd

USER fluent
WORKDIR /home/fluent

# Tell ruby to install packages as user
RUN echo "gem: --user-install --no-document" >> ~/.gemrc
ENV PATH /home/fluent/.gem/ruby/2.3.0/bin:$PATH
ENV GEM_PATH /home/fluent/.gem/ruby/2.3.0:$GEM_PATH

COPY fluent.conf /fluentd/etc/

ENV FLUENTD_OPT=""
ENV FLUENTD_CONF="fluent.conf"

#EXPOSE 24224 5140

CMD exec fluentd -c /fluentd/etc/$FLUENTD_CONF -p /fluentd/plugins $FLUENTD_OPT
