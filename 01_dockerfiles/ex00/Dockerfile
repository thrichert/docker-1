ARG VERSION=latest
FROM alpine:$VERSION

MAINTAINER trichert <trichert@student.42.fr>

RUN apk update \
	&& apk upgrade \
	&& apk add emacs \
	&& apk del build-base \
	&& rm -rf /var/cache/* tmp/* /var/log/* ~/.cache 

ENTRYPOINT ["emacs"]

