FROM openfaas/of-watchdog:0.8.1 AS watchdog
FROM alpine:3.12 AS build

RUN apk update && apk add py-pygments ca-certificates bash git && rm -rf /var/cache/apk/*

ENV HUGO_VERSION 0.78.2
ENV HUGO_ENV=production

RUN ARCH=`uname -m` && \
    if [ "$ARCH" == "x86_64" ]; then \
       export HUGO_BINARY="hugo_${HUGO_VERSION}_Linux-64bit"; \
    elif [ "$ARCH" == "armv7l" ]; then \
       export HUGO_BINARY="hugo_${HUGO_VERSION}_Linux-ARM"; \
    elif [ "$ARCH" == "aarch64" ]; then \
       export HUGO_BINARY="hugo_${HUGO_VERSION}_Linux-ARM64"; \
    fi \
  && mkdir /usr/local/hugo \
  && cd /usr/local/hugo \
  && wget  https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/${HUGO_BINARY}.tar.gz \
  && tar xzf /usr/local/hugo/${HUGO_BINARY}.tar.gz -C /usr/local/hugo/ \
	&& ln -s /usr/local/hugo/hugo /usr/local/bin/hugo \
	&& rm /usr/local/hugo/${HUGO_BINARY}.tar.gz

WORKDIR /home/app
COPY ./function .

RUN ["hugo", "--gc", "--minify"]

FROM alpine:3.12

# Add non root user
RUN addgroup -S app && adduser app -S -G app
RUN chown app /home/app

WORKDIR /home/app

USER app

COPY --from=build /home/app/public public
COPY --from=watchdog /fwatchdog /usr/bin/fwatchdog

ENV mode="static"
ENV static_path="/home/app/public"

HEALTHCHECK --interval=3s CMD [ -e /tmp/.lock ] || exit 1

CMD ["fwatchdog"]
